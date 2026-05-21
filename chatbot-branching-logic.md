# Chatbot Branching Logic — Hybrid AI v2

## Overview

This document explains **how the chatbot decides what to do** when a customer sends a message on Facebook Messenger. The v2 system uses a **hybrid approach**: it checks rules and Google Sheets data first (free), and only calls AI as a last resort (costs money).

**Platform:** Facebook Messenger
**Design:** Rules-first, AI-fallback

> **v5 Update (Latest):** The v5 Decision Engine (`n8n-messenger-hybrid-ai-v5.json`) upgrades routing with:
> - **Style/occasion routing BEFORE order intent** — "I need something stylish for Eid under 2000 BDT" routes to style recommendation, NOT order
> - **Weak order keywords removed** — "need", "want", "chai", "lagbe" alone no longer trigger order flow. Only strong signals like "order korte chai", "buy", "confirm order" trigger orders
> - **Size guide branch** — height/weight queries get size recommendations from Size_Guide sheet
> - **Objection handling** — "dam beshi", "last price", "quality kemon" get warm value-focused replies from Sales_Objections sheet
> - **Typo normalization** — "delivry", "tshart", "hudie", "jins" auto-corrected before routing
> - **Budget-aware** — detects "under 2000 BDT" and filters products accordingly
>
> v5 routing priority: greeting → support → tracking → **style/occasion** → **size** → **objection** → strong order → FAQ → product → AI fallback

---

## The Hybrid Routing Principle

```
┌─────────────────────────────────────────────────────────────┐
│  PRIORITY ORDER (checked top to bottom, first match wins)   │
├─────────────────────────────────────────────────────────────┤
│  1. Static direct replies (greetings/thanks/bye)    FREE    │
│  2. Human support detection (angry/complaint)       FREE    │
│  3. Order tracking detection (ORD-XXXXX/status)     FREE    │
│  4. Order/buy intent detection                      FREE    │
│  5. FAQ match from Google Sheets                    FREE    │
│  6. Product match from Google Sheets                FREE    │
│  7. AI Agent fallback (only if nothing above works) PAID    │
└─────────────────────────────────────────────────────────────┘
```

**Why this order?**
- Cheapest/fastest checks run first (static replies = instant, no data lookup)
- More specific intents (support, tracking) before general ones (FAQ, products)
- AI is the most expensive option — only used when everything else fails

---

## How Messages Arrive (Messenger-Specific)

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
Skip if: echo, delivery receipt, read receipt, or non-text
        |
        v
Normalize message (lowercase, remove special chars)
        |
        v
Run through hybrid routing logic (this document)
```

**Key Messenger concepts:**
- **PSID** (Page-Scoped ID): Each person who messages your Page gets a unique ID
- **24-hour messaging window**: You can only reply within 24 hours of customer's last message
- **2000 character limit**: Each text reply max 2000 characters

---

## Branch 1: Static Direct Replies (NO AI)

### Purpose
Respond instantly to simple conversational messages without any data lookup or AI call.

### Detection Rules

The message is treated as a greeting/thanks/bye **only if it is 3 words or fewer** AND contains a matching word. This prevents "Hi, I want to track my order" from being classified as just a greeting.

### Trigger Words

| Category | Words |
|----------|-------|
| **Greetings** | hi, hello, hey, hola, assalamualaikum, salam, yo, sup |
| **Thanks** | thanks, thank, thankyou, thx, jazakallah, shukria |
| **Goodbyes** | bye, goodbye, later, seeya, cya |

### Responses

| Detected As | Reply |
|------------|-------|
| Greeting | "Hello! Welcome to our store. I can help you find products, answer questions, or place an order. What are you looking for today?" |
| Thanks | "You are welcome! Let me know if there is anything else I can help with." |
| Bye | "Goodbye! Thanks for chatting with us. Come back anytime!" |

### Cost: FREE (no sheets read, no AI)

---

## Branch 2: Human Support Detection (NO AI)

### Purpose
Immediately flag conversations that need a real person — angry customers, complaints, refund requests, or explicit "talk to human" requests.

### Detection Rules

Checked BEFORE products/FAQ because "I want to return my order" should go to support, not product search.

### Trigger Words/Patterns

| Category | Words/Patterns |
|----------|---------------|
| **Direct request** | human, agent, person, representative, manager, speak to, talk to |
| **Complaints** | complaint, refund, fraud, overcharged, legal, lawyer |
| **Angry patterns** | Message is mostly UPPERCASE and longer than 10 characters |

### Response

```
I understand you need help from our team. I have flagged your conversation for priority support.

Our team will respond to you in this chat within 2-4 hours (business hours: Mon-Fri, 9 AM - 6 PM).

If urgent, you can also email: support@yourbrand.com

Is there anything else I can help with while you wait?
```

### Cost: FREE (no sheets read, no AI)

---

## Branch 3: Order Tracking Detection (NO AI)

### Purpose
Detect when a customer is asking about their order status or delivery.

### Detection Rules

| Category | Words/Patterns |
|----------|---------------|
| **Tracking words** | track, tracking, where is my order, order status, delivery status, has it shipped, my order, when will it arrive |
| **Order ID pattern** | Any text matching `ORD-XXXXXXXX-XXX` format |

### Response Logic

**If order ID detected in message:**
```
I found your order reference: ORD-20250120-001

Let me check the status for you. Our team will update you shortly with tracking details.

If you need immediate help, type "speak to a human".
```

**If no order ID provided:**
```
I can help you track your order! Please provide your order number (it looks like ORD-YYYYMMDD-XXX).

Example: "Track ORD-20250120-001"
```

### Cost: FREE (no AI — future versions will read Tracking tab automatically)

---

## Branch 4: Order/Buy Intent Detection (NO AI)

### Purpose
Detect when a customer wants to purchase something and guide them through the information needed.

### Detection Rules

| Category | Words |
|----------|-------|
| **Buy intent** | buy, order, purchase, i want, ill take, add to cart, i need, id like |

### Response

```
Great, I would love to help you place an order!

To create your order, I need:
1. Which product? (e.g., Classic Black T-Shirt)
2. What size?
3. What color?
4. Your name and email
5. Shipping address

Which product are you interested in? You can also say "show me t-shirts" to browse first.
```

### Cost: FREE (no AI — guides user to provide structured info)

---

## Branch 5: FAQ Match from Google Sheets (NO AI)

### Purpose
Answer common questions directly from the FAQ tab without calling AI.

### When It Runs
Only runs if Branches 1-4 didn't match. The FAQ sheet is read and searched.

### How FAQ Matching Works

1. **Normalize** the customer message (lowercase, remove special characters)
2. **Extract meaningful words** (remove common words like "the", "and", "for", "what", "how", etc.)
3. **Score each active FAQ row** against the search words:
   - Keywords column match: +2 points per keyword hit
   - Question column word match: +1 point per word hit
4. **Threshold:** Score must be ≥ 3 to count as a "strong match"
5. **If strong match found:** Reply with the Answer column directly — NO AI NEEDED
6. **If no strong match:** Fall through to Product search

### FAQ Tab Columns Used

| Column | Role in Matching |
|--------|-----------------|
| **Keywords** | Primary matching — comma-separated trigger words (weighted 2x) |
| **Question** | Secondary matching — words from the question itself (weighted 1x) |
| **Answer** | The reply text sent to the customer |
| **Active** | Must be "YES" — inactive FAQs are skipped |
| **Sort_Order** | Tie-breaker if two FAQs score equally |

### Example Matching

| Customer Says | Search Words | Best FAQ Match | Score | AI Used? |
|--------------|-------------|----------------|-------|----------|
| "What is your return policy?" | return, policy | FAQ with Keywords: "return, refund, exchange, send back, money back" | 4 (2×return + 2×policy match) | No |
| "How long does shipping take?" | long, shipping, take | FAQ with Keywords: "shipping, delivery, how long, days, arrive" | 6 | No |
| "Do you accept PayPal?" | accept, paypal | FAQ with Keywords: "payment, pay, credit card, paypal" | 4 | No |
| "What fabric is the hoodie made of?" | fabric, hoodie, made | No FAQ matches above threshold | 0 | Falls to Products |

### Reply Format (FAQ Match)

The Answer column text is sent directly — no modification needed:

```
[Answer from FAQ tab — exactly as written in your spreadsheet]
```

### Tips for Better FAQ Matching

- Add as many keyword variations as possible in the Keywords column
- Include misspellings customers commonly make
- Use the Sort_Order column to prioritize when two FAQs might match
- Set Active = "NO" for seasonal/outdated answers instead of deleting them

### Cost: FREE (reads Google Sheets only — no AI call)

---

## Branch 6: Product Match from Google Sheets (NO AI)

### Purpose
Find and display matching products when the customer is searching or browsing.

### When It Runs
Only runs if FAQ didn't find a strong match.

### How Product Matching Works

1. **Extract search terms** from customer message (remove words < 3 chars and common stop words)
2. **Filter:** Only products where Active = "YES"
3. **Score each variant** by checking search terms against:
   - Product_Name
   - Category
   - Description
   - Size
   - Color
   - Search_Keywords
4. **Filter:** Only variants with score > 0
5. **Group** matched variants by Product_ID (so one product doesn't show 12 times)
6. **For each group:** Collect in-stock sizes and colors
7. **Sort** by score (highest first), limit to top 5 products
8. **Format** a friendly reply

### Products Tab Columns Used

| Column | Role in Matching |
|--------|-----------------|
| **Product_Name** | Searched against message terms |
| **Category** | Searched (e.g., "t-shirts", "jeans", "hoodies") |
| **Description** | Searched for additional context |
| **Size** | Searched if customer mentions a size |
| **Color** | Searched if customer mentions a color |
| **Search_Keywords** | Extra searchable terms you add manually |
| **Active** | Must be "YES" — inactive products hidden |
| **In_Stock** | Used to show only available sizes/colors |
| **Price** | Displayed in results |

### Reply Format (Products Found)

```
I found 3 products for you:

1. Classic Black T-Shirt — $29.99
   Sizes: S, M, L
   Colors: Black, White, Navy

2. Premium V-Neck Tee — $34.99
   Sizes: S, M, L, XL
   Colors: Black, Charcoal

3. Graphic Print Tee — $24.99
   Sizes: M, L, XL
   Colors: White, Grey

Would you like to order any of these? Just tell me the product, size, and color!
```

### Reply Format (No Products Found)

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

### Cost: FREE (reads Google Sheets only — no AI call)

---

## Branch 7: AI Agent Fallback (COSTS MONEY)

### Purpose
Handle complex or conversational messages that cannot be answered from rules or data. This is the ONLY branch that calls AI.

### When It Runs
Only when ALL of these are true:
- Not a greeting/thanks/bye
- Not a support/tracking/order keyword match
- No FAQ strong match found (score < 3)
- No product match found (0 results)
- Search terms exist (not an empty/very short message)

### What the AI Agent Can Do

| Allowed | Examples |
|---------|---------|
| General style advice | "What should I wear to a wedding?" |
| Fabric/material questions | "Is cotton better than polyester for summer?" |
| Styling tips | "How do I style a hoodie?" |
| General brand questions | "What's your brand about?" |
| Friendly conversation | "Can you recommend something comfortable?" |

### What the AI Agent Must NOT Do

| Forbidden | Why | What It Says Instead |
|-----------|-----|---------------------|
| Invent product names or prices | Could mislead customer | "Let me search our catalog — try asking 'show me [category]'" |
| Make up stock/availability | Could cause order issues | "I can't check specific stock — try 'do you have [product] in [size]'" |
| Provide order status | Could give wrong info | "For order status, please type 'track ORD-XXXXX'" |
| Process refunds/complaints | Needs human judgment | "For refunds, type 'speak to a human' and our team will help" |
| Discuss non-brand topics | Off-brand risk | "I'm best at helping with fashion and our store! What can I help you find?" |

### AI System Prompt (in the workflow)

```
You are a friendly customer support assistant for a clothing brand. Your job is to help with general style advice, sizing guidance, and store-related questions.

RULES:
1) Keep answers short (under 200 words).
2) Do NOT invent product names, prices, stock levels, or delivery dates.
3) Do NOT provide order status or tracking info — tell the customer to ask 'track my order ORD-XXXXX'.
4) Do NOT process refunds or complaints — tell the customer to say 'speak to a human'.
5) If you are unsure, recommend the customer speak with our support team.
6) Stay within clothing brand support topics only.
```

### Cost: PAID (every AI Agent call costs API credits)

---

## Special Case: No Search Terms Extracted

If the message is too short or contains only common words (e.g., "ok", "hmm", "??"), no meaningful search terms can be extracted. In this case, the bot replies with a helpful prompt WITHOUT calling AI:

```
I'd love to help! You can ask me about our products, shipping, returns, sizing, or place an order. What would you like to know?
```

### Cost: FREE (handled in the Code node, no AI call)

---

## Messages the Bot Ignores (No Reply)

These events arrive from Meta but should NOT trigger any reply:

| Event Type | How Detected | Why Ignore |
|-----------|-------------|------------|
| Delivery receipt | `messaging.delivery` exists | Just confirms message delivered |
| Read receipt | `messaging.read` exists | Just confirms message seen |
| Echo | `message.is_echo = true` | Message sent BY your page (not customer) |
| Image/sticker/audio | `message.text` is missing | v2 only handles text |
| Postback | `messaging.postback` exists | Button clicks — not yet implemented |

---

## Messenger Constraints

| Constraint | How We Handle It |
|-----------|-----------------|
| 2000 character limit | Prepare Reply node truncates to 1950 chars |
| No markdown/bold | All replies use plain text + line breaks + bullet (•) |
| 5-second response window | 200 OK sent immediately in parallel with processing |
| 24-hour messaging window | All replies are type "RESPONSE" (within window) |

---

## Decision Examples

| Customer Message | Route Taken | Why | AI? |
|-----------------|-------------|-----|-----|
| "Hi" | Branch 1 (greeting) | ≤3 words, contains "hi" | No |
| "Hello, show me jeans" | Branch 6 (products) | >3 words, "hello" not sole intent | No |
| "Thanks a lot" | Branch 1 (thanks) | ≤3 words, contains "thanks" | No |
| "What's your return policy?" | Branch 5 (FAQ) | FAQ Keywords match "return" (score ≥3) | No |
| "Show me black t-shirts" | Branch 6 (products) | Products match "black" + "t-shirts" | No |
| "Where is my order?" | Branch 3 (tracking) | Contains "where is my order" | No |
| "Track ORD-20250120-001" | Branch 3 (tracking) | Contains ORD-XXXXX pattern | No |
| "I want to buy the hoodie" | Branch 4 (order) | Contains "buy" | No |
| "I want to speak to someone" | Branch 2 (support) | Contains "speak to" | No |
| "THIS IS TERRIBLE!" | Branch 2 (support) | Uppercase + angry pattern | No |
| "What should I wear to a beach wedding?" | Branch 7 (AI) | No FAQ/product match | **Yes** |
| "Is linen good for hot weather?" | Branch 7 (AI) | No FAQ/product match | **Yes** |
| "hmm" | No-search-terms fallback | No meaningful words | No |

---

## How to Improve Matching (Reduce AI Usage)

If too many messages are hitting the AI fallback:

### Add more FAQ entries
- Check what customers are asking that goes to AI
- Create FAQ entries with good Keywords for those topics
- Example: If customers ask "What's your brand story?" — add an FAQ for it

### Add more Search_Keywords to products
- If "casual shirts" doesn't find your t-shirts, add "casual" to Search_Keywords
- If "summer clothes" doesn't match, add "summer" to relevant products

### Lower the FAQ threshold
- The current threshold is score ≥ 3
- If good FAQs are being missed, you could lower to ≥ 2 (but risk false matches)

### Add more greeting/support trigger words
- If customers say "hey there" and it goes to AI, add "hey" to greetings list
- If customers say "need assistance" and it misses support, add "assistance"

---

## Future Enhancements (v3+)

| Feature | What It Adds |
|---------|-------------|
| **Full order creation** | Multi-step order flow with variant validation and Google Sheets write |
| **Tracking tab lookup** | Actually read Tracking/Orders tabs for real status |
| **Customer recognition** | Look up PSID in Customers tab for personalization |
| **Support ticket creation** | Write to Support_Tickets tab during handoff |
| **Conversation memory** | Remember context across multiple messages |
| **Messenger buttons** | Send quick-reply buttons for common actions |
