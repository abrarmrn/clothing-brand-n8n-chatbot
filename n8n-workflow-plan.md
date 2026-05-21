# n8n Workflow Plan — Hybrid AI v2 Architecture

## Overview

This document explains the **Messenger Hybrid AI v2** workflow architecture — which nodes exist, how they connect, and why the system checks rules and Google Sheets **before** calling AI.

**Core principle:** Save AI/API costs by answering from rules and data first. Only use the AI Agent when no confident answer can be given from static rules or Google Sheets.

> **v5 Available (Latest):** The v5 workflow (`n8n-messenger-hybrid-ai-v5.json`) is a stable upgrade from v3 with: style/occasion routing before order intent, message.mid deduplication, customer memory, typo normalization, size guide, objection handling, budget-aware search, improved product ranking, and canonical output from all branches. See `messenger-hybrid-v5-import-instructions.md` for setup and `messenger-hybrid-v5-test-plan.md` for testing.

---

## Platform: Facebook Messenger

| Component | Role |
|-----------|------|
| **Facebook Page** | What customers message |
| **Meta Webhooks** | Delivers customer messages to your n8n webhook URL |
| **n8n Webhooks** | Receives messages (POST) and verifies ownership (GET) |
| **Google Sheets** | Your database (Products, FAQ, Orders, Tracking, Customers, Support_Tickets) |
| **Messenger Send API** | How n8n sends replies back to the customer |
| **AI Agent + Chat Model** | Fallback ONLY — handles complex questions that sheets/rules cannot answer |

---

## Cost-Saving Design

| Route | AI Used? | Cost |
|-------|----------|------|
| Greetings (hi, hello, assalamualaikum, thanks, bye) | No | **Free** |
| FAQ answers (matched from FAQ tab) | No | **Free** |
| Product lookup (matched from Products tab) | No | **Free** |
| Order tracking detection | No | **Free** |
| Human support detection | No | **Free** |
| Order/buy intent detection | No | **Free** |
| Complex/unclear questions (AI fallback) | **Yes** | **Costs money** |

**Expected:** 70-90% of messages handled without AI.

---

## Full Architecture Diagram

### Path A: GET Webhook — Meta Verification (one-time)

```
Meta sends GET request (during webhook setup)
        |
        v
[1. GET Webhook (Verify)] — path: messenger-webhook
        |
        v
[2. Verify Token Check] — hub.mode=subscribe AND hub.verify_token matches?
        |
        ├── YES → [3. Respond with Challenge] — return hub.challenge (200)
        └── NO  → [4. Respond 403 Forbidden]
```

### Path B: POST Webhook — Hybrid Message Processing

```
Customer sends message in Messenger
        |
        v
[5. POST Webhook (Messages)] — path: messenger-webhook
        |
        ├──→ [6. Respond 200 OK] — immediate ACK (parallel)
        └──→ [7. Extract Message Data] — parse PSID + text
                |
                v
        [8. Is Valid Text Message?] — skip echoes/delivery/read/non-text
                |
                ├── NO → (ends — no reply)
                └── YES ↓
                        v
        [9. Hybrid Router (Rules First)] — checks static rules IN ORDER:
                |
                ├── greeting/thanks/bye? → direct reply text → [14. Prepare Reply]
                ├── support keywords? → [12. Support Ticket Handler] → [14]
                ├── tracking keywords? → [11. Tracking Lookup] → [14]
                ├── buy/order keywords? → [13. Order Intent Handler] → [14]
                └── none matched? → route = "check_sheets"
                        |
                        v
        [10. Needs Sheet Lookup?] — route == "check_sheets"?
                |
                ├── YES → Read FAQ Sheet + Read Products Sheet (parallel)
                |              |
                |              v
                |         [FAQ + Product Search (No AI)] — score FAQ then products
                |              |
                |              ├── FAQ strong match? → reply with Answer (aiUsed: false) → [14]
                |              ├── Product match? → formatted product list (aiUsed: false) → [14]
                |              └── No match? → aiUsed: true
                |                      |
                |                      v
                |              [AI Needed?] — aiUsed == true?
                |                      |
                |                      ├── YES → [AI Agent (Fallback Only)] + [Chat Model] → [14]
                |                      └── NO  → [14. Prepare Reply]
                |
                └── NO (direct reply from router) → [14. Prepare Reply]
                        |
                        v
        [14. Prepare Reply] — truncate to 2000 chars, format
                |
                v
        [15. Send Messenger Reply] — HTTP POST to Graph API
```

---

## All Nodes Explained (17 total)

### Node 1: GET Webhook (Verify)

| Setting | Value |
|---------|-------|
| **Type** | Webhook |
| **Method** | GET |
| **Path** | `messenger-webhook` |
| **Response Mode** | Response Node |
| **Purpose** | Meta calls this once to verify webhook ownership |

---

### Node 2: Verify Token Check

| Setting | Value |
|---------|-------|
| **Type** | IF |
| **Conditions** | hub.mode == "subscribe" AND hub.verify_token == "CHANGE_ME_VERIFY_TOKEN" |
| **TRUE** | → Respond with Challenge |
| **FALSE** | → Respond 403 |

---

### Node 3: Respond with Challenge

| Setting | Value |
|---------|-------|
| **Type** | Respond to Webhook |
| **Code** | 200 |
| **Body** | hub.challenge value (plain text) |

---

### Node 4: Respond 403 Forbidden

| Setting | Value |
|---------|-------|
| **Type** | Respond to Webhook |
| **Code** | 403 |
| **Body** | "Forbidden" |

---

### Node 5: POST Webhook (Messages)

| Setting | Value |
|---------|-------|
| **Type** | Webhook |
| **Method** | POST |
| **Path** | `messenger-webhook` |
| **Purpose** | Receives every message/event from Messenger |

Meta sends the message payload here. We respond 200 immediately (Node 6) and process in parallel.

---

### Node 6: Respond 200 OK

| Setting | Value |
|---------|-------|
| **Type** | Respond to Webhook |
| **Code** | 200 |
| **Body** | "EVENT_RECEIVED" |
| **Purpose** | Tell Meta we received it (prevents retries) |

Runs in **parallel** with Extract Message Data — both connect from POST Webhook.

---

### Node 7: Extract Message Data

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Parse Meta's nested JSON to get sender PSID and message text |

**Extracts:**
- `senderPsid` — unique ID for the person
- `messageText` — what they typed

**Skips (returns skip: true):**
- Delivery receipts
- Read receipts
- Echo messages (sent BY the page)
- Non-text (images, stickers, audio)

---

### Node 8: Is Valid Text Message?

| Setting | Value |
|---------|-------|
| **Type** | IF |
| **Condition** | skip == false |
| **TRUE** | → Continue to Hybrid Router |
| **FALSE** | → End (no reply needed) |

---

### Node 9: Hybrid Router (Rules First)

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | The brain of the hybrid system — checks rules in priority order |

**Logic order:**

1. **Greeting/Thanks/Bye** (≤3 words, contains greeting word) → instant static reply
2. **Support keywords** (human, agent, complaint, refund, fraud, angry patterns) → support handler
3. **Tracking keywords** (track, order status, ORD-XXXXX pattern) → tracking handler
4. **Buy/Order keywords** (buy, order, purchase, I want) → order intent handler
5. **None matched** → pass to sheet lookup (FAQ + Products)

**Why this order?**
- Support and tracking are checked before products because "I want to return my order" should go to support, not product search
- Greetings checked first because they're the cheapest (no sheet read needed)

---

### Node 10: Needs Sheet Lookup?

| Setting | Value |
|---------|-------|
| **Type** | IF |
| **Condition** | route == "check_sheets" |
| **TRUE** | → Read FAQ + Read Products (parallel) |
| **FALSE** | → Prepare Reply (direct static reply from router) |

**Why this split?** If the Hybrid Router already produced a reply (greeting, support, tracking, order), we skip reading Google Sheets entirely — saves API calls and time.

---

### Node 10a: Read FAQ Sheet

| Setting | Value |
|---------|-------|
| **Type** | Google Sheets — Read |
| **Document** | Clothing Brand Chatbot Database |
| **Sheet** | FAQ |
| **Purpose** | Loads all FAQ rows for keyword matching |

---

### Node 10b: Read Products Sheet

| Setting | Value |
|---------|-------|
| **Type** | Google Sheets — Read |
| **Document** | Clothing Brand Chatbot Database |
| **Sheet** | Products |
| **Purpose** | Loads all product variants for search matching |

Both run in **parallel** to minimize wait time.

---

### Node 10c: FAQ + Product Search (No AI)

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Searches FAQ first, then Products. Only falls to AI if neither matches. |

**FAQ matching logic:**
1. Extract meaningful words from customer message
2. Score each active FAQ by keyword match (Keywords column + Question words)
3. If best score ≥ 3 → strong match → return FAQ Answer directly (no AI)

**Product matching logic (if FAQ didn't match):**
1. Score each active product variant against search terms
2. Group by Product_ID, collect in-stock sizes and colors
3. If any products found → format reply (no AI)

**If neither matched:** Set `aiUsed: true` → falls through to AI Agent

---

### Node 10d: AI Needed?

| Setting | Value |
|---------|-------|
| **Type** | IF |
| **Condition** | aiUsed == true |
| **TRUE** | → AI Agent (Fallback Only) |
| **FALSE** | → Prepare Reply (FAQ or product reply) |

---

### Node 10e: AI Agent (Fallback Only)

| Setting | Value |
|---------|-------|
| **Type** | AI Agent (LangChain) |
| **Purpose** | Generates a helpful reply ONLY when rules and sheets couldn't answer |

**System prompt rules:**
- Keep answers short (under 200 words)
- Do NOT invent product names, prices, stock levels, or delivery dates
- Do NOT provide order status or tracking info
- Do NOT process refunds or complaints
- If unsure, recommend speaking with support team
- Stay within clothing brand support topics only

---

### Node 10f: Chat Model

| Setting | Value |
|---------|-------|
| **Type** | LangChain Chat OpenAI |
| **Model** | gpt-4o-mini (cheapest) |
| **Purpose** | The AI model that powers the AI Agent |
| **Credential** | Placeholder — replace with your OpenAI API key |

---

### Node 11: Tracking Lookup (No AI)

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Handles order tracking requests |
| **AI Used?** | No |

If order ID detected in message → acknowledge and confirm lookup.
If no order ID → ask customer to provide it.

---

### Node 12: Support Ticket Handler (No AI)

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Handles human support requests and angry customers |
| **AI Used?** | No |

Returns a message confirming the request has been escalated to the team.

---

### Node 13: Order Intent Handler

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Handles buy/order intent detection |
| **AI Used?** | No |

Returns a message asking for the product details needed to create an order.

---

### Node 14: Prepare Reply

| Setting | Value |
|---------|-------|
| **Type** | Code |
| **Purpose** | Normalizes the reply text before sending |

**What it does:**
- Extracts reply text from whichever upstream node provided it
- Truncates to 1950 characters (Messenger limit is 2000)
- Provides a fallback message if something went wrong

---

### Node 15: Send Messenger Reply

| Setting | Value |
|---------|-------|
| **Type** | HTTP Request |
| **Method** | POST |
| **URL** | `https://graph.facebook.com/v18.0/me/messages` |
| **Auth** | Header Auth (Bearer token — Page Access Token) |
| **Body** | `{ recipient: { id: PSID }, messaging_type: "RESPONSE", message: { text: reply } }` |

---

## Node Connections Summary

| From | To | Condition |
|------|----|-----------|
| GET Webhook | Verify Token Check | Always |
| Verify Token Check | Respond with Challenge | TRUE |
| Verify Token Check | Respond 403 | FALSE |
| POST Webhook | Respond 200 OK | Always (parallel) |
| POST Webhook | Extract Message Data | Always (parallel) |
| Extract Message Data | Is Valid Text Message? | Always |
| Is Valid Text Message? | Hybrid Router | TRUE |
| Is Valid Text Message? | (ends) | FALSE |
| Hybrid Router | Needs Sheet Lookup? | Always |
| Needs Sheet Lookup? | Read FAQ + Read Products | TRUE (route=check_sheets) |
| Needs Sheet Lookup? | Prepare Reply | FALSE (direct reply) |
| Read FAQ Sheet | FAQ + Product Search | Always |
| Read Products Sheet | FAQ + Product Search | Always |
| FAQ + Product Search | AI Needed? | Always |
| AI Needed? | AI Agent | TRUE (aiUsed=true) |
| AI Needed? | Prepare Reply | FALSE |
| AI Agent | Prepare Reply | Always |
| Tracking Lookup | Prepare Reply | Always |
| Support Ticket Handler | Prepare Reply | Always |
| Order Intent Handler | Prepare Reply | Always |
| Prepare Reply | Send Messenger Reply | Always |

---

## Credentials Required (3 total)

| Credential | Type in n8n | What For | When It's Used |
|-----------|-------------|----------|----------------|
| **Google Sheets OAuth2** | Google Sheets OAuth2 | Reading FAQ and Products tabs | Every message that reaches sheet lookup |
| **Facebook Page Access Token** | Header Auth | Sending replies via Messenger | Every reply |
| **OpenAI API** | OpenAI API | AI Agent fallback responses | Only when rules + sheets can't answer (10-30% of messages) |

---

## Data Flow

```
Messenger → Meta → POST Webhook → 200 OK (parallel)
                        |
                        v
              Extract PSID + text
                        |
                        v
              Hybrid Router (rules check)
                        |
           ┌────────────┼────────────────┐
           |            |                |
      greeting/      tracking/        check_sheets
      support/       order              |
      (static)       (static)     Read FAQ + Products
           |            |                |
           |            |         Score & Match
           |            |                |
           |            |         ┌──────┴──────┐
           |            |         |             |
           |            |     FAQ/Product    AI Agent
           |            |      (free)       (costs $)
           └────────────┼─────────┴─────────────┘
                        |
                        v
                  Prepare Reply
                        |
                        v
              Send via Graph API → Customer sees reply
```

---

## Testing Checklist

| # | Test | Send in Messenger | Expected Route | AI Used? |
|---|------|-------------------|---------------|----------|
| 1 | Greeting | "Hi" | greeting → static reply | No |
| 2 | Thanks | "Thank you" | thanks → static reply | No |
| 3 | Bye | "Goodbye" | bye → static reply | No |
| 4 | FAQ match | "What is your return policy?" | check_sheets → faq_match | No |
| 5 | Product match | "Show me t-shirts" | check_sheets → product_match | No |
| 6 | No product | "Purple sandals" | check_sheets → no match → AI | Yes |
| 7 | Tracking | "Where is my order?" | tracking handler | No |
| 8 | Tracking with ID | "Track ORD-20250120-001" | tracking handler | No |
| 9 | Support | "I want to talk to a human" | support handler | No |
| 10 | Angry | "THIS IS TERRIBLE SERVICE" | support handler | No |
| 11 | Order intent | "I want to buy the black t-shirt" | order handler | No |
| 12 | Style question | "What should I wear to a wedding?" | AI fallback | Yes |
| 13 | Complex question | "Is cotton breathable in summer?" | AI fallback | Yes |
| 14 | Non-text | (send a photo) | skipped — no reply | No |
| 15 | Delivery receipt | (auto from Meta) | skipped — no reply | No |

---

## Comparison: v1 vs v2

| Feature | v1 | v2 |
|---------|----|----|
| Greetings | Not handled (treated as product search) | Static reply (free) |
| FAQ | Not handled | Matched from FAQ sheet (free) |
| Products | Always runs product search | Runs only when needed (free) |
| Tracking | Not handled | Detected and replied (free) |
| Support | Not handled | Detected and replied (free) |
| Order intent | Not handled | Detected with guidance (free) |
| AI fallback | None | AI Agent for complex questions (costs $) |
| Nodes | 11 | 17 |
| Credentials | 2 | 3 |
| Message routing | None (all = product search) | Priority-ordered hybrid routing |
