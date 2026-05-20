# n8n Workflow Plan — Facebook Messenger Architecture

## Overview

This document explains **which n8n nodes you need** and **how they connect together** to build the Messenger chatbot workflow.

This is a **Facebook Messenger chatbot** — NOT an n8n hosted chat widget. Messages arrive from Meta's webhook system and replies are sent via the Messenger Send API.

---

## Platform: Facebook Messenger

| Component | Role |
|-----------|------|
| **Facebook Page** | What customers message |
| **Meta Webhooks** | Delivers customer messages to your n8n webhook URL |
| **n8n Webhooks** | Receives messages (POST) and verifies ownership (GET) |
| **Google Sheets** | Your database (products, orders, FAQ, etc.) |
| **Messenger Send API** | How n8n sends replies back to the customer |

### Why Messenger Webhooks (Not n8n Chat Trigger)?

The n8n "Chat Trigger" node is for n8n's built-in chat widget. This project uses **real Facebook Messenger** where customers message your actual Facebook Page. This requires:

1. A **GET webhook** for Meta's one-time verification handshake
2. A **POST webhook** to receive every incoming message
3. An **HTTP Request node** to send replies via Facebook's Graph API

---

## Workflow Architecture (The Big Picture)

### Path A: GET Webhook — Meta Verification (one-time setup)

```
Meta sends GET request (during webhook setup)
        |
        v
[1. GET Webhook] — Receives verification request
        |
        v
[2. Verify Token Check] — Does hub.verify_token match our secret?
        |
        ├── YES → [3. Respond with Challenge] — Return hub.challenge (200)
        |
        └── NO  → [4. Respond 403 Forbidden] — Reject the request
```

### Path B: POST Webhook — Message Processing

```
Customer sends message in Messenger
        |
        v
Meta sends POST to your webhook
        |
        v
[5. POST Webhook] — Receives message payload
        |
        ├──→ [6. Respond 200 OK] — Immediately acknowledge (Meta requires < 5 seconds)
        |
        └──→ [7. Extract Message Data] — Parse sender PSID + message text
                |
                v
        [8. Is Valid Message?] — Skip echoes, delivery receipts, non-text
                |
                ├── YES → [9. Message Classifier] — What does the customer want?
                |            |
                |            ├── product_inquiry → [10. Read Products] → [11. Search & Format] → [12. Send Reply]
                |            ├── faq            → [FAQ nodes - future v2]
                |            ├── greeting       → [Greeting reply - future v3]
                |            ├── order_create   → [Order nodes - future v4]
                |            ├── order_track    → [Tracking nodes - future v5]
                |            ├── human_support  → [Handoff nodes - future v6]
                |            └── unknown        → [Fallback reply]
                |
                └── NO (skip) → (workflow ends — no reply needed)
```

### Current v1: Product Lookup Only

In version 1, there is no message classifier yet. Every text message goes directly to product search:

```
[5. POST Webhook] → [6. Respond 200] + [7. Extract Message] → [8. Valid?] → [10. Read Products] → [11. Search & Format] → [12. Send Reply]
```

---

## All Nodes Explained

### Node 1: GET Webhook (Verify)

| Setting | Value |
|---------|-------|
| **Node Type** | Webhook |
| **HTTP Method** | GET |
| **Path** | `messenger-webhook` |
| **Response Mode** | Response Node (custom response) |
| **Purpose** | Meta calls this once during webhook setup to verify you own the URL |

**What Meta sends:**
```
GET https://your-n8n.com/webhook/messenger-webhook
  ?hub.mode=subscribe
  &hub.verify_token=YOUR_SECRET_TOKEN
  &hub.challenge=1234567890
```

**What your workflow must do:**
- Check that `hub.mode` is "subscribe"
- Check that `hub.verify_token` matches your secret
- If both match: respond with `hub.challenge` value as plain text (200)
- If token doesn't match: respond with 403

---

### Node 2: Verify Token Check

| Setting | Value |
|---------|-------|
| **Node Type** | IF |
| **Purpose** | Validates the verify token from Meta |
| **Condition 1** | `query['hub.mode']` equals `subscribe` |
| **Condition 2** | `query['hub.verify_token']` equals `CHANGE_ME_VERIFY_TOKEN` |
| **Combinator** | AND (both must be true) |

**Important:** Replace `CHANGE_ME_VERIFY_TOKEN` with your own secret string. The same value must be entered in Meta Developer Console.

---

### Node 3: Respond with Challenge

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Webhook |
| **Response Code** | 200 |
| **Response Body** | The value of `hub.challenge` from the query params |
| **Content-Type** | text/plain |
| **Purpose** | Proves to Meta that you control this webhook URL |

---

### Node 4: Respond 403 Forbidden

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Webhook |
| **Response Code** | 403 |
| **Response Body** | "Forbidden" |
| **Purpose** | Rejects verification attempts with wrong token |

---

### Node 5: POST Webhook (Messages)

| Setting | Value |
|---------|-------|
| **Node Type** | Webhook |
| **HTTP Method** | POST |
| **Path** | `messenger-webhook` |
| **Response Mode** | Response Node (custom response) |
| **Purpose** | Receives every message/event from Messenger |

**What Meta sends (example):**
```json
{
  "object": "page",
  "entry": [{
    "id": "PAGE_ID",
    "time": 1234567890,
    "messaging": [{
      "sender": { "id": "SENDER_PSID" },
      "recipient": { "id": "PAGE_ID" },
      "timestamp": 1234567890,
      "message": {
        "mid": "MESSAGE_ID",
        "text": "Show me black t-shirts"
      }
    }]
  }]
}
```

**Important:** Meta expects a 200 response within 5 seconds or it will retry. That's why we respond immediately and process the message in parallel.

---

### Node 6: Respond 200 OK

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Webhook |
| **Response Code** | 200 |
| **Response Body** | "EVENT_RECEIVED" |
| **Purpose** | Tells Meta "I got the message" so it doesn't retry |

**This runs in parallel with message processing** — both the 200 response and the Extract Message node connect directly from the POST Webhook.

---

### Node 7: Extract Message Data

| Setting | Value |
|---------|-------|
| **Node Type** | Code |
| **Purpose** | Parses the Meta webhook payload to extract useful data |

**What it extracts:**
- `senderPsid` — the unique ID of the person who sent the message
- `messageText` — the actual text they typed

**What it skips (returns `skip: true`):**
- Delivery receipt events (`messaging.delivery`)
- Read receipt events (`messaging.read`)
- Echo messages (`message.is_echo` — messages sent BY the page)
- Non-text messages (images, stickers, audio — no `message.text`)

---

### Node 8: Is Valid Message?

| Setting | Value |
|---------|-------|
| **Node Type** | IF |
| **Condition** | `skip` equals `false` |
| **TRUE path** | Continue to product search (or future classifier) |
| **FALSE path** | Stop — no reply needed for delivery/read/echo events |

---

### Node 9: Message Classifier (Future — Not in v1)

| Setting | Value |
|---------|-------|
| **Node Type** | AI Agent OR Switch node |
| **Purpose** | Routes messages to the correct branch |
| **Categories** | greeting, product_inquiry, faq, order_create, order_track, human_support, unknown |

**In v1:** This node doesn't exist yet. All valid messages go directly to product search.

**In v2+:** This will be added between "Is Valid Message?" and the branch-specific nodes. See `chatbot-branching-logic.md` for the full classification system.

---

### Node 10: Read Products Sheet

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Read (get all rows) |
| **Document** | Clothing Brand Chatbot Database |
| **Sheet** | Products |
| **Purpose** | Loads all product variants from the Products tab |

**Columns read:**
Product_ID, Variant_SKU, Product_Name, Category, Description, Price, Currency, Size, Color, Stock_Qty, In_Stock, Image_URL, Product_URL, Search_Keywords, Active

**Why read ALL rows (not search)?** The search logic needs to match against multiple columns simultaneously (Product_Name, Category, Description, Color, Size, Search_Keywords). A Code node handles this more flexibly than Google Sheets' built-in filter.

---

### Node 11: Search & Format Products

| Setting | Value |
|---------|-------|
| **Node Type** | Code |
| **Purpose** | Matches customer message against products and builds a reply |

**Logic:**
1. Extract meaningful search terms from customer message (remove short/common words)
2. Filter only Active = "YES" products
3. Score each variant by how many search terms match its searchable fields
4. Group results by Product_ID (so one product shows once with all its sizes/colors)
5. Sort by relevance score, show top 5
6. Build a formatted text reply

**If products found:**
```
I found 2 products for you:

1. Classic Black T-Shirt — $29.99
   Sizes: S, M, L
   Colors: Black, White, Navy

2. Premium V-Neck Tee — $34.99
   Sizes: S, M, L, XL
   Colors: Black, Charcoal

Would you like to order any of these? Just tell me the product, size, and color!
```

**If NO products found:**
```
I couldn't find any products matching "purple sandals".

Here are our categories:
• T-Shirts
• Jeans
• Hoodies
• Dresses
• Accessories

Try asking something like "Show me hoodies" or "Black t-shirts in large".
```

---

### Node 12: Send Messenger Reply

| Setting | Value |
|---------|-------|
| **Node Type** | HTTP Request |
| **Method** | POST |
| **URL** | `https://graph.facebook.com/v18.0/me/messages` |
| **Authentication** | Header Auth (Bearer token with Page Access Token) |
| **Purpose** | Sends the reply back to the customer in Messenger |

**Request body:**
```json
{
  "recipient": {
    "id": "SENDER_PSID"
  },
  "messaging_type": "RESPONSE",
  "message": {
    "text": "The formatted product reply..."
  }
}
```

**Authentication:** Uses an HTTP Header Auth credential with:
- Header Name: `Authorization`
- Header Value: `Bearer EAAxxxxxxx...` (your Page Access Token)

---

## Node Connections Summary

### Path A: GET Verification

| From | To | Condition |
|------|----|-----------|
| GET Webhook (Verify) | Verify Token Check | Always |
| Verify Token Check | Respond with Challenge | TRUE (token matches) |
| Verify Token Check | Respond 403 Forbidden | FALSE (token wrong) |

### Path B: POST Message Processing (v1)

| From | To | Condition |
|------|----|-----------|
| POST Webhook (Messages) | Respond 200 OK | Always (parallel) |
| POST Webhook (Messages) | Extract Message Data | Always (parallel) |
| Extract Message Data | Is Valid Message? | Always |
| Is Valid Message? | Read Products Sheet | TRUE (valid text message) |
| Is Valid Message? | (nothing — ends) | FALSE (echo/delivery/read) |
| Read Products Sheet | Search & Format Products | Always |
| Search & Format Products | Send Messenger Reply | Always |

---

## Future Nodes (v2-v6)

These will be added in future versions:

| Version | Nodes to Add | Connects After |
|---------|-------------|----------------|
| v2 | FAQ Search + FAQ Reply | Message Classifier → faq branch |
| v3 | Greeting Reply (personalized) | Message Classifier → greeting branch |
| v4 | Collect Order Info + Validate Variant + Save Order + Confirm | Message Classifier → order_create branch |
| v5 | Extract Order # + Tracking Lookup + Orders Fallback + Reply | Message Classifier → order_track branch |
| v6 | Priority Logic + Create Ticket + Handoff Reply | Message Classifier → human_support branch |

All future reply nodes will use the same **Send Messenger Reply** pattern (HTTP Request to Graph API with sender PSID).

---

## Error Handling

### Error: Google Sheets Connection Failed

| Where | What Happens | User Sees |
|-------|-------------|-----------|
| Read Products Sheet | Node throws error | Send a fallback reply: "I'm having trouble looking that up. Please try again in a moment." |

**How to set up:** Add an Error output from the Google Sheets node, connect it to a Code node that builds a fallback message, then to the Send Messenger Reply node.

### Error: Messenger Send API Failed

| Where | What Happens | Possible Cause |
|-------|-------------|----------------|
| Send Messenger Reply | HTTP 400/401/403 | Page Access Token expired or invalid |

**How to handle:** Check n8n execution logs. Regenerate the Page Access Token if expired.

### Error: Meta Sends Retry (Duplicate Messages)

| Where | What Happens | Why |
|-------|-------------|-----|
| POST Webhook | Same message arrives twice | n8n didn't respond 200 fast enough |

**How to handle:** The "Respond 200 OK" node runs in parallel with processing, so this should not happen. If it does, check n8n performance/load.

---

## Credentials Required

| Credential | Type in n8n | What For |
|-----------|-------------|----------|
| **Google Sheets OAuth2** | Google Sheets OAuth2 | Reading product data from your spreadsheet |
| **Facebook Page Access Token** | Header Auth | Sending replies via Messenger Send API |

**How to set up each credential:** See `messenger-import-instructions.md` for step-by-step guides.

**Security:** No credentials are stored in this repository. The JSON file uses placeholder values that you replace inside n8n.

---

## Data Flow (v1)

```
Messenger → Meta → POST Webhook
                        |
                        v
              Extract sender PSID + message text
                        |
                        v
              Read ALL rows from Products tab
                        |
                        v
              Search/score/group/format in Code node
                        |
                        v
              Send reply via Graph API → Messenger → Customer
```

---

## Testing Checklist (v1)

| # | Test | What to Send in Messenger | Expected Result |
|---|------|--------------------------|-----------------|
| 1 | Basic product search | "Show me t-shirts" | Returns t-shirt products with sizes/colors |
| 2 | Specific variant query | "Black hoodie in large" | Returns hoodie with stock info |
| 3 | No results | "Purple sandals" | Friendly "not found" with categories list |
| 4 | Price query | "What do you have under $40?" | Products under $40 |
| 5 | Short/empty query | "Hi" | Helpful prompt (no search terms extracted) |
| 6 | Non-text message | Send a photo | No reply (skipped correctly) |
| 7 | Active filter | Search for a product with Active = "NO" | Should NOT appear in results |
| 8 | Out of stock | Search for product with all variants Stock_Qty = 0 | Shows product with "out of stock" note |

---

## Messenger-Specific Limitations

| Limitation | Details |
|-----------|---------|
| **Message length** | Messenger supports up to 2000 characters per text message. If product list is longer, truncate to top 3-4 products. |
| **Rate limits** | Standard rate limit: 200 API calls per hour per page. This is per OUTGOING message. |
| **24-hour window** | You can only message a user within 24 hours of their last message to you (standard messaging). |
| **No formatting** | Messenger text messages don't support markdown, bold, or italic. Use plain text and line breaks only. |
| **Image support** | Future versions can send product images using Messenger's attachment API (not text). |
