# Chatbot Branching Logic

## Overview

This document explains **how the chatbot decides what to do** when a customer sends a message. Think of it like a receptionist who listens to what someone says, figures out what they need, and directs them to the right department.

---

## The 6 Branches

Every customer message goes to **one** of these branches:

| Branch # | Name | When To Use |
|----------|------|-------------|
| 1 | Direct Reply | Simple greetings, thanks, goodbyes |
| 2 | Product Lookup | Customer is asking about products |
| 3 | FAQ Answer | Customer has a general question |
| 4 | Draft Order Creation | Customer wants to buy something |
| 5 | Order Tracking | Customer wants to check their order status |
| 6 | Human Support Handoff | Customer needs a real person |

---

## How Classification Works

When a message arrives, the chatbot checks it against **keywords** and **patterns**. The first match wins.

### Decision Flowchart

```
Message arrives
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


## Branch 1: Direct Reply

### Purpose
Respond to simple conversational messages that do not need any data lookup.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Greetings | hi, hello, hey, good morning, good afternoon, good evening, howdy, what's up |
| Thanks | thank you, thanks, thx, appreciate it, great thanks |
| Goodbyes | bye, goodbye, see you, take care, have a good day |
| Affirmations | ok, okay, sure, got it, understood, perfect, great |

### Example Messages and Responses

| Customer Says | Bot Responds |
|--------------|-------------|
| "Hi" | "Hello! Welcome to [Brand Name]! I can help you with: browsing products, answering questions, placing orders, or tracking deliveries. What would you like to do?" |
| "Hello there!" | "Hey there! Great to have you. How can I help you today?" |
| "Thank you so much" | "You're welcome! Happy to help. Is there anything else I can do for you?" |
| "Bye" | "Goodbye! Thanks for chatting with us. Come back anytime!" |
| "Ok got it" | "Great! Let me know if you need anything else." |

### Personalization (Using Customers Tab)

If the customer is recognized (their **Chat_User_ID** was found in the Customers tab):
- Use their **Customer_Name**: "Hi Sarah! Welcome back. How can I help?"
- If **Total_Orders** > 5: "Hey Sarah! Great to see one of our VIP customers. What can I do for you today?"

If the customer is NOT recognized:
- Use generic: "Hello! Welcome to [Brand Name]!"

### Rules
- Always end with an invitation to continue the conversation
- Keep responses short and friendly
- If the greeting includes another intent (like "Hi, do you have jeans?"), route to the OTHER intent instead

---


## Branch 2: Product Lookup

### Purpose
Find and display products from the Google Sheets Products tab. Since each row is a **variant** (specific size + color), the bot groups results by Product_ID to show a clean product listing.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Browsing | show me, do you have, what products, browse, catalog, collection |
| Product types | t-shirt, tshirt, jeans, hoodie, jacket, dress, skirt, pants, shorts, sweater, shirt |
| Attributes | black, white, red, blue, small, medium, large, xl, cotton, size |
| Price-related | how much, price, cost, cheap, affordable, expensive, under $X |
| Availability | in stock, available, do you carry, do you sell |
| Specific | SKU, variant, stock, how many left |

### How the Search Works (Variant-Based)

1. Bot extracts keywords from the customer message
2. Searches the Products tab filtering: **Active = "YES"**
3. Matches against: Product_Name, Category, Color, Size, Search_Keywords
4. Groups results by **Product_ID** (so one product doesn't show 12 times)
5. For each product group, lists available sizes and colors from in-stock variants

### Example Messages and Bot Actions

| Customer Says | What The Bot Does |
|--------------|-------------------|
| "Do you have black t-shirts?" | Searches: Category="T-Shirts" AND Color="Black" AND Active="YES". Groups by Product_ID. Shows available sizes. |
| "Show me your hoodies" | Searches: Category="Hoodies" AND Active="YES". Groups by Product_ID. Lists all colors/sizes in stock. |
| "What do you have under $40?" | Searches: Price < 40 AND Active="YES" AND In_Stock="YES". Groups results. |
| "Is the slim fit jeans available in size 32?" | Searches: Product_Name contains "Slim Fit Jeans" AND Size="32" AND Active="YES". Returns specific variant with Stock_Qty. |
| "How many black t-shirts in medium do you have?" | Searches: exact variant match. Returns Stock_Qty for that Variant_SKU. |

### Reply Logic

**If products found (1-5 unique products):**
```
I found [X] product(s) matching your request:

1. Classic Black T-Shirt — $29.99
   Sizes in stock: S (15), M (22), L (8)
   Colors available: Black, White, Navy

2. Premium V-Neck Tee — $34.99
   Sizes in stock: S (10), M (12), L (6), XL (4)
   Colors available: Black, Charcoal

Would you like to order any of these? Just tell me the product, size, and color!
```

**If products found (more than 5 unique products):**
```
I found [X] products! Here are the top 5:

[show 5 products with sizes/colors]

Would you like me to narrow it down? You can tell me a preferred size, color, or price range.
```

**If NO products found:**
```
I couldn't find any products matching "[search term]" that are currently in stock.

Here are our available categories:
- T-Shirts
- Jeans
- Hoodies
- Dresses
- Accessories

Would you like to browse one of these, or can I help with something else?
```

**If specific variant is out of stock (Stock_Qty = 0 or In_Stock = "NO"):**
```
I found the Classic Black T-Shirt, but Size M in Black is currently out of stock.

Available alternatives for this product:
- Size M in White (15 in stock)
- Size M in Navy (8 in stock)
- Size L in Black (8 in stock)

Would you like one of these instead, or shall I create a support ticket to notify you when it's back?
```

---


## Branch 3: FAQ Answer

### Purpose
Answer common questions using the FAQ tab in Google Sheets. Only returns answers where **Active = "YES"**, and uses **Sort_Order** to prioritize when multiple FAQs match.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Shipping | shipping, delivery, how long, ship, when will it arrive, shipping cost, free shipping |
| Returns | return, refund, exchange, send back, money back, return policy |
| Sizing | size, sizing, fit, measurement, size guide, size chart, what size |
| Payment | payment, pay, credit card, debit, paypal, apple pay, payment methods |
| General | policy, hours, location, contact, about, where are you, who are you |
| Discount | discount, coupon, promo code, sale, deal, offer |
| Care | wash, care, instructions, laundry, shrink |

### Example Messages

| Customer Says | Bot Searches FAQ Keywords For |
|--------------|-------------------------------|
| "How long does shipping take?" | shipping, long, take |
| "Can I return something?" | return |
| "What size should I get?" | size |
| "Do you accept PayPal?" | payment, paypal |
| "Do you have any discounts?" | discount |
| "How do I wash my hoodie?" | wash, care |

### Matching Logic

1. Extract keywords from customer message (remove common words like "the", "a", "do", "you")
2. Compare against the **Keywords** column in the FAQ tab
3. Only consider rows where **Active = "YES"**
4. Count how many keywords match each FAQ entry
5. Return the FAQ entry with the MOST matches
6. If multiple entries tie, return the one with the lowest **Sort_Order**
7. If still tied, return all tied entries

### Reply Logic

**If FAQ match found:**
```
Great question! Here's what I found:

[Answer from FAQ tab]

Does this answer your question? If not, I can:
- Search for more information
- Connect you with our team
```

**If NO FAQ match found:**
```
I don't have a specific answer for that question in my knowledge base.

Would you like me to:
1. Connect you with our support team who can help?
2. Try rephrasing your question?

You can also ask about: shipping, returns, sizing, payments, or discounts.
```

---


## Branch 4: Draft Order Creation

### Purpose
Collect order details from the customer, validate the variant exists and is in stock, and save a complete order to the Orders tab in Google Sheets.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Buying intent | I want to buy, I want to order, add to cart, purchase, I'll take, I'll get |
| Order language | order, buy, purchase, get me, I need, I'd like, can I get |
| Quantity | one, two, three, 1, 2, 3, a pair of, a couple of |
| Confirmation | yes I want it, I'll order that, let me get that |

### Example Messages

| Customer Says | Bot Action |
|--------------|------------|
| "I want to order the black t-shirt" | Start order flow, ask for size |
| "I'll take 2 of the slim fit jeans in blue, size 32" | Most info provided, validate variant, ask for shipping details |
| "Can I buy that hoodie?" | Start order flow, ask for size and color |
| "Yes, I want to order it in large" | Continue order flow (customer already browsing) |

### Order Collection Flow

The bot needs to collect all this information. The order maps to these Google Sheets columns:

```
Step 1: Which product? → Product_ID, Product_Name
        |
        v
Step 2: What size? → Size
        |
        v
Step 3: What color? → Color
        |
        v
Step 4: [VALIDATE] Does this Variant_SKU exist? Is Stock_Qty > 0? Is Active = YES?
        |
        v
Step 5: How many? (default: 1) → Quantity
        |         [CHECK: Stock_Qty >= Quantity]
        v
Step 6: Customer name → Customer_Name
        |
        v
Step 7: Customer email → Customer_Email
        |
        v
Step 8: Customer phone (optional) → Customer_Phone
        |
        v
Step 9: Shipping address → Shipping_Address, City, Country
        |
        v
Step 10: Delivery method → Delivery_Method
         (Standard: 5-7 days, $5.99 | Express: 2-3 days, $12.99)
         |
         v
Step 11: Payment method → Payment_Method
         (Credit Card, PayPal, Apple Pay, Bank Transfer)
         |
         v
Step 12: Any special notes? (optional) → Notes
         |
         v
Step 13: Confirm order summary
         |
         v
Step 14: Save to Google Sheets → All 25 columns filled
```

### Auto-Filled Fields (Bot generates these)

| Field | How It's Generated |
|-------|-------------------|
| Order_ID | Format: ORD-YYYYMMDD-XXX (auto-generated) |
| Customer_ID | From Customers tab lookup, or generate new CUST-XXX |
| Variant_SKU | Looked up from Products tab based on Product_ID + Size + Color |
| Unit_Price | From Products tab (Price column for matched variant) |
| Total_Price | Unit_Price × Quantity |
| Currency | From Products tab |
| Payment_Status | Always "Pending" at creation |
| Order_Status | Always "Draft" at creation |
| Channel | From chat platform (WhatsApp, Website, Messenger, Instagram) |
| Created_Date | Current date/time |
| Updated_Date | Current date/time |

### Conversation Example

```
Customer: "I want to buy a black t-shirt"
Bot:      "Great choice! The Classic Black T-Shirt is $29.99.
           What size would you like? Available: S, M, L, XL"

Customer: "Large please"
Bot:      "Size L in Black — checking stock... 8 available!
           How many would you like?"

Customer: "Just 1"
Bot:      "Perfect! To ship your order, I'll need:
           1. Your full name
           2. Email address
           3. Shipping address (street, city, country)"

Customer: "Sarah Johnson, sarah@email.com, 123 Main St, New York, US"
Bot:      "Got it! How would you like it delivered?
           • Standard — 5-7 business days ($5.99)
           • Express — 2-3 business days ($12.99)"

Customer: "Standard"
Bot:      "And payment method?
           • Credit Card
           • PayPal
           • Apple Pay
           • Bank Transfer"

Customer: "PayPal"
Bot:      "Here's your order summary:

           Product: Classic Black T-Shirt
           Variant: Size L, Black (SKU: PROD-001-BLK-L)
           Quantity: 1
           Unit Price: $29.99
           Shipping: Standard ($5.99)
           Total: $35.98
           Payment: PayPal

           Ship to: Sarah Johnson
           123 Main St, New York, US

           Shall I confirm this order?"

Customer: "Yes!"
Bot:      "Order created! Your order number is ORD-20250120-001.
           Status: Draft (our team will confirm and send payment details shortly).
           You'll receive updates at sarah@email.com.

           Is there anything else I can help with?"
```

### Validation Rules

| Check | What To Do If It Fails |
|-------|----------------------|
| Product not found (no matching Product_ID with Active="YES") | "I can't find that product. Would you like me to show you what we have?" |
| Variant doesn't exist (Size + Color combo not in Products) | "That size/color combination isn't available. Available options: [list variants for that Product_ID]" |
| Variant out of stock (In_Stock="NO" or Stock_Qty=0) | "Sorry, that variant is out of stock. Alternatives: [list in-stock variants]" |
| Not enough stock (Stock_Qty < requested Quantity) | "We only have [X] of that variant left. Would you like [X] instead?" |
| Invalid email format | "That email doesn't look quite right. Could you double-check it?" |
| Missing required info | "I still need your [field] to complete the order. Could you provide that?" |
| Customer wants to cancel mid-flow | "No problem! Order cancelled. Let me know if you change your mind." |

### Returning Customer Shortcut

If the customer is recognized (Customer_ID found from Customers tab):
- Pre-fill Customer_Name, Customer_Email, Customer_Phone
- Ask: "Should I ship to the same address as last time?" (if previous order exists)
- Skip asking for info you already have

---


## Branch 5: Order Tracking

### Purpose
Look up order status and shipping information from Google Sheets (Tracking tab and Orders tab).

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Tracking | track, tracking, where is my order, order status, delivery status |
| Order reference | order number, ORD-, my order, order update |
| Shipping | has it shipped, when will it arrive, estimated delivery, shipping update |
| Delivery | delivered, delivery, package, parcel |

### Example Messages

| Customer Says | Bot Action |
|--------------|------------|
| "Where is my order?" | Check if customer is known → use email. Otherwise ask for order number. |
| "Track order ORD-20250120-001" | Search Tracking tab by Order_ID directly |
| "Has my package shipped yet?" | Check if customer is known → lookup by Customer_Email |
| "When will order ORD-20250120-001 arrive?" | Search Tracking tab, return Estimated_Delivery |

### Lookup Flow

```
Step 1: Does the message contain an order number (ORD-XXXXX)?
        |
        YES --> Search Tracking tab by Order_ID
        NO  --> continue
        |
        v
Step 2: Is the customer recognized? (Customer_ID found from Customers tab)
        |
        YES --> Search Tracking tab by Customer_Email
        NO  --> Ask: "Could you provide your order number (starts with ORD-)
                      or your email address?"
        |
        v
Step 3: Search Tracking tab
        |
        FOUND --> Return tracking info
        NOT FOUND --> Check Orders tab for Order_Status
        |
        v
Step 4: Return results or helpful error
```

### Reply Logic

**If tracking info found (from Tracking tab):**
```
Here's the status for order [Order_ID]:

Status: [Shipping_Status]
Carrier: [Carrier]
Tracking Number: [Tracking_Number]
Shipped: [Shipped_Date]
Estimated Delivery: [Estimated_Delivery]
Last Updated: [Last_Updated] by [Updated_By]

Track your package: [Tracking_URL]

Is there anything else I can help with?
```

**If order found in Orders tab but NOT in Tracking tab (not shipped yet):**
```
I found your order [Order_ID]!

Current Status: [Order_Status]
Payment: [Payment_Status]
Product: [Product_Name] — [Size], [Color]

Your order hasn't shipped yet. Here's what the statuses mean:
- Draft → Awaiting confirmation from our team
- Confirmed → Payment received, being prepared
- Processing → Almost ready to ship!

You'll receive tracking info once it ships. Anything else I can help with?
```

**If order NOT found anywhere:**
```
I couldn't find an order with that number/email. This could mean:
- The order number might have a typo (format is: ORD-YYYYMMDD-XXX)
- The order might be under a different email address

Would you like to:
1. Try a different order number or email?
2. Connect with our support team?
```

**If customer has multiple orders (searched by email):**
```
I found [X] orders for your account:

1. ORD-20250120-001 — Classic Black T-Shirt (M, Black)
   Status: In Transit | ETA: Jan 26

2. ORD-20250118-003 — Slim Fit Jeans (32, Blue)
   Status: Delivered on Jan 20

Which order would you like details on?
```

---


## Branch 6: Human Support Handoff

### Purpose
Connect the customer to a real human when the bot cannot help, or when they explicitly ask. Creates a detailed ticket in the **Support_Tickets** tab with full conversation context.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Direct request | talk to a human, real person, speak to someone, agent, representative, manager |
| Frustration | this isn't helping, useless, doesn't work, I'm frustrated, terrible service |
| Complex issues | complaint, wrong item, damaged, broken, overcharged, fraud, urgent |
| Repeated failure | (customer has asked 3+ times without resolution) |

### Automatic Handoff Triggers

The bot should AUTOMATICALLY hand off to a human (without the customer asking) when:

| Situation | Handoff_Reason Value | Priority |
|-----------|---------------------|----------|
| Customer is clearly angry (profanity, ALL CAPS, exclamation marks!!!) | angry_customer | High |
| Bot has failed to understand 3 messages in a row | bot_failed_3x | Low |
| Customer mentions legal action or formal complaint | complex_issue | High |
| Issue involves payment/billing problems | complex_issue | High |
| Customer mentions damaged/wrong items received | complex_issue | High |
| Customer explicitly asks for a human | customer_requested | Low |
| Customer needs something outside standard policy | policy_exception | Medium |

### Information Collected for Support_Tickets Tab

The bot fills these columns automatically:

| Column | How Bot Fills It |
|--------|-----------------|
| Ticket_ID | Generate: TKT-YYYYMMDD-XXX |
| Conversation_ID | From chat platform session ID |
| Customer_ID | From Customers tab (if known), otherwise blank |
| Customer_Name | From Customers tab or ask during handoff |
| Customer_Email | From Customers tab or ask during handoff |
| Customer_Phone | From Customers tab or ask during handoff |
| Channel | From chat platform (WhatsApp, Website, Messenger, Instagram) |
| Issue_Category | Detected from context (Order Issue, Product Question, Complaint, Payment, Shipping, Returns, Other) |
| Issue_Description | Bot's summary of the problem |
| Last_User_Message | Exact copy of the customer's last message |
| Bot_Summary | 1-2 sentence summary of the full conversation |
| Handoff_Reason | Why handoff happened (see table above) |
| Priority | Based on rules below |
| Status | "Open" (always) |
| Created_Date | Current date/time |
| Assigned_To | (blank — team assigns later) |
| Resolved_Date | (blank) |
| Resolution_Notes | (blank) |

### Priority Assignment Rules

| Condition | Priority | Response Time Goal |
|-----------|----------|-------------------|
| Customer is angry OR mentions damaged/wrong item OR payment issue OR legal | **High** | Within 2 hours |
| Customer asks for exchange, has sizing issues, general complaint, policy exception | **Medium** | Within 4 hours |
| Bot could not answer a question, customer just prefers humans, general inquiry | **Low** | Within 24 hours |

### Handoff Conversation

**When customer explicitly requests:**
```
Customer: "I want to talk to a real person"
Bot:      "Of course! I'll connect you with our team.

           To help them assist you faster, could you briefly tell me
           what you need help with? (Or just say 'skip' and I'll transfer you now)"

Customer: "I received the wrong size"
Bot:      "I'm sorry about that! I've created a support ticket:

           Ticket: TKT-20250120-001
           Issue: Received wrong size
           Priority: Medium

           Our team will contact you within 4 hours
           (during business hours: Mon-Fri, 9 AM - 6 PM).

           Is there anything else I can help with while you wait?"
```

**When bot auto-triggers handoff (angry customer):**
```
Bot:      "I can see this is frustrating, and I'm sorry for the trouble.
           I'm connecting you with our support team right now — they'll
           be able to help you directly.

           Ticket: TKT-20250120-002
           Priority: High — our team will reach out within 2 hours.

           If it's urgent, you can also email support@yourbrand.com."
```

**When bot auto-triggers handoff (3 failed attempts):**
```
Bot:      "I'm having trouble understanding what you need, and I don't
           want to waste your time. Let me connect you with a team member
           who can help.

           Ticket: TKT-20250120-003
           Our team will follow up within 24 hours.

           In the meantime, you can email us at support@yourbrand.com."
```

### Information Collection Rules

Try to collect before handoff (but don't force if customer is frustrated):

1. Customer name (skip if already known from Customers tab)
2. Customer email (skip if already known)
3. Brief description of the issue
4. Related order number (if applicable)

**If customer says "skip" or is clearly frustrated:** Create the ticket with whatever info you have. Use **Last_User_Message** and **Bot_Summary** to give the support team context.

---


## Fallback Logic (When No Branch Matches)

### When It Triggers
- Message doesn't match any keyword patterns
- Message is gibberish or very short/unclear
- Message is in a language the bot doesn't support

### Fallback Response

```
I'm not sure I understood that. Here's what I can help with:

1. Products — "Show me t-shirts" or "What's in stock?"
2. Questions — "What's your return policy?" or "Shipping info"
3. Place an order — "I want to buy..."
4. Track an order — "Where is my order ORD-XXXXX?"
5. Talk to our team — "Connect me with support"

Which of these can I help you with?
```

### Fallback Escalation Rules
- **1st unknown message:** Show the menu above
- **2nd unknown message in a row:** Show menu + "If you'd prefer to speak with a person, just say 'support'"
- **3rd unknown message in a row:** Auto-trigger Branch 6 (Human Handoff) with Handoff_Reason = "bot_failed_3x", Priority = "Low"

---

## Edge Cases and Special Situations

### Customer sends multiple intents in one message

**Example:** "Hi, I want to track my order and also buy a new t-shirt"

**Rule:** Handle the FIRST actionable intent (based on priority order). After completing it, ask about the second.

```
Bot: "Let me help you track your order first! What's your order number?
     (I'll help you with the t-shirt right after)"
```

### Customer changes mind mid-conversation

**Example:** In the middle of placing an order, customer says "actually, never mind"

**Rule:** Acknowledge and reset.

```
Bot: "No problem at all! Order cancelled. Is there something else I can help with?"
```

### Customer sends just an emoji

**Rule:** Treat based on emoji type:

| Emoji | Treat As | Response |
|-------|----------|----------|
| Thumbs up, smiley, heart | Affirmation | "Got it! Anything else I can help with?" |
| Sad face, angry face | Concern | "Is everything okay? How can I help?" |
| Question mark | Unclear | "What would you like to know?" |
| Other | Fallback | Show menu |

### Customer sends a photo/image

**Rule:** The bot cannot analyze images.

```
Bot: "I received your image, but I'm not able to analyze photos.
     Could you describe what you need help with in text?
     Or I can connect you with our team who can view the image."
```

### Message is very long (100+ words)

**Rule:** Focus on the most specific keywords. If unclear, summarize and confirm.

```
Bot: "I want to make sure I help you correctly. It sounds like you need
     help with [topic]. Is that right?"
```

### Returning customer with existing order

**Rule:** If a recognized customer (Customer_ID found) has recent orders, offer proactive help:

```
Bot: "Hi Sarah! Welcome back. I see you have a recent order
     (ORD-20250120-001). Would you like an update on that,
     or is there something else I can help with?"
```

---

## Keyword Matching Setup for n8n

### Option A: Using a Switch Node (No AI, Free)

Set up a **Switch** node with conditions checked in this priority order. Always convert the message to lowercase first.

```
Priority 1: Message contains "track" OR "where is my order" OR "ORD-" OR "order status"
            → Branch 5 (Order Tracking)

Priority 2: Message contains "buy" OR "order" OR "purchase" OR "I want" OR "I'll take" OR "add to cart"
            → Branch 4 (Draft Order)

Priority 3: Message contains "speak to" OR "human" OR "agent" OR "representative"
            OR frustrated language (ALL CAPS, "!!!", "terrible", "useless")
            → Branch 6 (Human Handoff)

Priority 4: Message contains product keywords (t-shirt, jeans, hoodie, dress, etc.)
            OR "show me" OR "in stock" OR "available" OR "price" OR "how much"
            → Branch 2 (Product Lookup)

Priority 5: Message contains FAQ keywords (shipping, return, size, payment, discount, wash, care)
            → Branch 3 (FAQ)

Priority 6: Message contains greeting keywords (hi, hello, thanks, bye, ok)
            → Branch 1 (Direct Reply)

Priority 7: Default/else
            → Fallback
```

### Option B: Using AI Classification (More Accurate, Requires API Key)

Use an **AI Agent** node or **OpenAI** node with this prompt:

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

Customer message: "{customer_message}"
```

---

## Summary Decision Table

| If the message is about... | Go to Branch | Priority in checking |
|---------------------------|-------------|---------------------|
| Tracking an existing order | 5 — Order Tracking | 1st (most specific) |
| Buying/ordering a product | 4 — Draft Order | 2nd |
| Needing a real human | 6 — Human Handoff | 3rd |
| Asking about products/browsing | 2 — Product Lookup | 4th |
| General questions (shipping, returns, etc.) | 3 — FAQ | 5th |
| Simple greeting/thanks/bye | 1 — Direct Reply | 6th |
| Cannot determine | Fallback | Last resort |
