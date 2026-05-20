# Chatbot Branching Logic — Facebook Messenger

## Overview

This document explains **how the chatbot decides what to do** when a customer sends a message on Facebook Messenger. Think of it like a receptionist who listens to what someone says, figures out what they need, and directs them to the right department.

**Platform:** Facebook Messenger (customers message your Facebook Page)

---

## How Messages Arrive (Messenger-Specific)

Unlike a simple chat widget, Messenger messages arrive through Meta's webhook system:

```
Customer types in Messenger
        |
        v
Meta sends POST to your n8n webhook
        |
        v
n8n extracts:
  - sender PSID (unique ID for this person on your Page)
  - message text (what they typed)
        |
        v
Skip if: echo, delivery receipt, read receipt, or non-text (image/sticker)
        |
        v
Process the text message through branching logic
```

**Key Messenger concepts:**
- **PSID** (Page-Scoped ID): Each person who messages your Page gets a unique ID. This is how you identify returning customers.
- **24-hour messaging window**: You can only reply within 24 hours of the customer's last message.
- **2000 character limit**: Each text reply can be max 2000 characters. Keep responses concise.

---

## The 6 Branches

Every customer message goes to **one** of these branches:

| Branch # | Name | When To Use | Version |
|----------|------|-------------|---------|
| 1 | Direct Reply | Simple greetings, thanks, goodbyes | Future (v3) |
| 2 | **Product Lookup** | Customer is asking about products | **v1 — Built** |
| 3 | FAQ Answer | Customer has a general question | Future (v2) |
| 4 | Draft Order Creation | Customer wants to buy something | Future (v4) |
| 5 | Order Tracking | Customer wants to check their order status | Future (v5) |
| 6 | Human Support Handoff | Customer needs a real person | Future (v6) |

### Current v1 Behavior

In version 1, there is **no classifier** yet. Every text message is treated as a product search. If the search finds nothing, the bot returns a helpful fallback with category suggestions.

---

## How Classification Will Work (v2+)

When the message classifier is added, it will check messages against **keywords** and **patterns**. The first match wins.

### Decision Flowchart

```
Message text extracted from Messenger payload
    |
    v
Does it contain order tracking keywords?
    YES --> Branch 5: Order Tracking
    NO  --> continue
    |
    v
Does it contain ordering/buying keywords?
    YES --> Branch 4: Draft Order Creation
    NO  --> continue
    |
    v
Does it contain human support keywords?
    YES --> Branch 6: Human Support Handoff
    NO  --> continue
    |
    v
Does it contain product keywords?
    YES --> Branch 2: Product Lookup
    NO  --> continue
    |
    v
Does it match FAQ keywords?
    YES --> Branch 3: FAQ Answer
    NO  --> continue
    |
    v
Is it a simple greeting/thanks/goodbye?
    YES --> Branch 1: Direct Reply
    NO  --> continue
    |
    v
Fallback: Send menu of options
```

**Why this order?** More specific intents (tracking, ordering) are checked first. Greetings are checked last because phrases like "Hi, I want to track my order" should go to tracking, not be treated as just a greeting.

---

## Branch 2: Product Lookup (v1 — BUILT)

### Purpose
Find and display products from the Google Sheets Products tab. Since each row is a **variant** (specific size + color), the bot groups results by Product_ID to show a clean product listing.

### How It Works in v1

```
Customer message arrives via Messenger webhook
    |
    v
Extract message text + sender PSID
    |
    v
Read ALL rows from Products tab (Google Sheets)
    |
    v
Filter: Active = "YES" only
    |
    v
Score each variant against search terms from message
    |
    v
Group matched variants by Product_ID
    |
    v
Format top 5 products into a reply (< 2000 chars)
    |
    v
Send reply via Messenger Send API (HTTP Request to Graph API)
```

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Browsing | show me, do you have, what products, browse, catalog, collection |
| Product types | t-shirt, tshirt, jeans, hoodie, jacket, dress, skirt, pants, shorts, sweater, shirt |
| Attributes | black, white, red, blue, small, medium, large, xl, cotton, size |
| Price-related | how much, price, cost, cheap, affordable, expensive, under |
| Availability | in stock, available, do you carry, do you sell |
| Specific | SKU, variant, stock, how many left |

### Search Logic (Code Node)

1. Convert customer message to lowercase
2. Remove special characters
3. Split into words
4. Remove common words (the, and, for, with, have, show, want, etc.) and words under 3 characters
5. Match remaining words against each product's searchable fields:
   - Product_Name
   - Category
   - Description
   - Size
   - Color
   - Search_Keywords
6. Score = number of matching terms
7. Filter products with score > 0
8. Group by Product_ID, collecting in-stock sizes and colors
9. Sort by score (highest first)
10. Return top 5 products

### Reply Format (Messenger)

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

**If search terms couldn't be extracted (message too short/generic):**
```
I'd love to help you find something! Try asking about a product like:

• "Show me t-shirts"
• "Do you have hoodies?"
• "Black jeans in size 32"
• "What do you have under $40?"

What are you looking for?
```

### Messenger-Specific Considerations

| Constraint | How We Handle It |
|-----------|-----------------|
| 2000 character limit | Limit to top 5 products; truncate descriptions |
| No markdown/bold | Use plain text, bullets (•), and line breaks |
| No clickable buttons in text | Future: use Messenger button templates |
| Reply must use Send API | HTTP Request to `graph.facebook.com/v18.0/me/messages` |

---

## Branch 1: Direct Reply (Future — v3)

### Purpose
Respond to simple conversational messages that do not need any data lookup.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Greetings | hi, hello, hey, good morning, good afternoon, good evening |
| Thanks | thank you, thanks, thx, appreciate it |
| Goodbyes | bye, goodbye, see you, take care |
| Affirmations | ok, okay, sure, got it, understood, perfect |

### Personalization via PSID

When the classifier routes to Direct Reply, the bot can check if this PSID exists in the Customers tab:

- **Known customer:** "Hi Sarah! Welcome back. How can I help?"
- **Unknown customer:** "Hello! Welcome to [Brand Name]. I can help you find products, answer questions, or place orders. What would you like to do?"

### Messenger Reply Format
```
{
  "recipient": { "id": "SENDER_PSID" },
  "messaging_type": "RESPONSE",
  "message": { "text": "Hello! Welcome to [Brand Name]..." }
}
```

---

## Branch 3: FAQ Answer (Future — v2)

### Purpose
Answer common questions using the FAQ tab in Google Sheets.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Shipping | shipping, delivery, how long, ship, arrive, shipping cost |
| Returns | return, refund, exchange, send back, money back |
| Sizing | size, sizing, fit, measurement, size guide |
| Payment | payment, pay, credit card, paypal, apple pay |
| Discount | discount, coupon, promo code, sale, deal |
| Care | wash, care, instructions, laundry |

### Matching Logic
1. Only return FAQs where Active = "YES"
2. Score by keyword match count
3. Tie-break by Sort_Order (lower = higher priority)
4. Return the best-matching answer

---

## Branch 4: Draft Order Creation (Future — v4)

### Purpose
Collect order details and save to Google Sheets Orders tab.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Buying intent | I want to buy, I want to order, purchase, I'll take |
| Order language | order, buy, purchase, get me, I need, I'd like |

### Order Flow (Messenger-Specific)

In Messenger, multi-step conversations work like this:

```
Customer: "I want to order the black t-shirt"
Bot: "Great! What size? (S, M, L, XL)"     ← Reply within 24h window
Customer: "Large"                             ← Resets 24h window
Bot: "Size L, Black. To ship your order..."  ← Reply within 24h window
...and so on
```

**Important:** Each customer reply resets the 24-hour messaging window, so multi-step order collection works naturally in Messenger.

### Auto-Filled Fields
- **Channel**: "Messenger" (always, for this workflow)
- **Customer_ID**: Look up by PSID in Customers tab (Chat_User_ID column)

---

## Branch 5: Order Tracking (Future — v5)

### Purpose
Look up order status from Tracking and Orders tabs.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Tracking | track, tracking, where is my order, order status |
| Order reference | order number, ORD-, my order |
| Shipping | has it shipped, when will it arrive, delivery |

### Lookup via PSID

Since we know the customer's PSID, we can:
1. Look up their Customer_ID from the Customers tab (Chat_User_ID = PSID)
2. Search Orders tab by Customer_ID
3. Search Tracking tab by Order_ID

This means known customers don't need to provide their order number — we can find it automatically.

---

## Branch 6: Human Support Handoff (Future — v6)

### Purpose
Create a support ticket and tell the customer a human will follow up.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Direct request | talk to a human, real person, speak to someone, agent |
| Frustration | this isn't helping, useless, doesn't work, frustrated |
| Complex issues | complaint, wrong item, damaged, broken, overcharged |

### Messenger-Specific Handoff

In Messenger, "handoff" means:
1. Bot creates a Support_Tickets row with full context (including PSID)
2. Bot sends a message: "I've created ticket TKT-XXXXX. Our team will reply in this same chat within [X hours]."
3. A human team member uses Facebook Page Inbox (or a tool like Respond.io) to continue the conversation
4. The 24-hour window applies — human must reply within 24h of customer's last message

**Handoff_Reason values:**
- `customer_requested` — customer asked for a human
- `bot_failed_3x` — bot couldn't understand 3 messages in a row
- `angry_customer` — frustration/anger detected
- `complex_issue` — payment, legal, damaged items
- `policy_exception` — request outside standard policy

---

## Fallback Logic

### When No Branch Matches (v2+)

```
I'm not sure I understood that. Here's what I can help with:

1. Products — "Show me t-shirts"
2. Questions — "What's your return policy?"
3. Place an order — "I want to buy..."
4. Track an order — "Where is my order ORD-XXXXX?"
5. Talk to our team — "Connect me with support"

Which would you like?
```

### Fallback Escalation (v6)
- 1st unknown: Show menu
- 2nd unknown in a row: Show menu + "Say 'support' to talk to a person"
- 3rd unknown in a row: Auto-create support ticket with Handoff_Reason = "bot_failed_3x"

---

## Messenger Events the Bot Ignores

These events arrive via the POST webhook but should NOT trigger a reply:

| Event Type | How to Detect | Why Ignore |
|-----------|--------------|------------|
| Delivery receipt | `messaging.delivery` exists | Just confirms message was delivered |
| Read receipt | `messaging.read` exists | Just confirms message was seen |
| Echo | `message.is_echo = true` | This is a message YOUR page sent (not the customer) |
| Image/sticker/audio | `message.text` is missing | v1 only handles text; future versions may handle images |
| Postback | `messaging.postback` exists | For button clicks — not implemented in v1 |
| Referral | `messaging.referral` exists | When someone clicks a ref link — future feature |

The "Extract Message Data" Code node in the workflow handles all of these by returning `skip: true`.

---

## Keyword Matching Setup (For v2+ Classifier)

### Option A: Switch Node (No AI, Free)

Convert message to lowercase, then check conditions in priority order:

```
1. Contains "track" OR "where is my order" OR "ORD-" OR "order status"
   → Branch 5 (Order Tracking)

2. Contains "buy" OR "order" OR "purchase" OR "I want" OR "I'll take"
   → Branch 4 (Draft Order)

3. Contains "speak to" OR "human" OR "agent" OR "representative"
   OR angry patterns (ALL CAPS, "!!!", "terrible", "useless")
   → Branch 6 (Human Handoff)

4. Contains product keywords (t-shirt, jeans, hoodie, dress, etc.)
   OR "show me" OR "in stock" OR "available" OR "price"
   → Branch 2 (Product Lookup)

5. Contains FAQ keywords (shipping, return, size, payment, discount)
   → Branch 3 (FAQ)

6. Contains greeting keywords (hi, hello, thanks, bye, ok)
   → Branch 1 (Direct Reply)

7. Default
   → Fallback
```

### Option B: AI Classification (More Accurate)

Use an AI node with this prompt:

```
Classify the following customer message into exactly one category.
Return ONLY the category name, nothing else.

Categories:
- greeting (simple hi, thanks, bye, ok)
- product_inquiry (asking about products, browsing, availability, sizes, colors, price)
- faq (questions about shipping, returns, sizing policy, payment methods, discounts)
- order_create (wants to buy/order something, add to cart, purchase)
- order_track (asking about order status, delivery, tracking, where is my order)
- human_support (wants a real person, is frustrated, complex issue, complaint)
- unknown (cannot determine intent)

Customer message: "{message_text}"
```

---

## Summary: Build Order

| Version | What's Built | Classifier Needed? |
|---------|-------------|-------------------|
| **v1** | Product Lookup only (every message = product search) | No |
| **v2** | + FAQ answers | Yes (add Switch or AI node) |
| **v3** | + Greeting replies (personalized via PSID) | Yes |
| **v4** | + Order creation (multi-step in Messenger) | Yes |
| **v5** | + Order tracking (auto-lookup by PSID) | Yes |
| **v6** | + Human handoff (ticket creation + escalation) | Yes |

Each version adds one branch. The classifier is added in v2 and routes to all subsequent branches.
