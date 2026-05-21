# Chatbot Branching Logic

## Overview

This document explains **how the chatbot decides what to do** when a customer sends a message. Think of it like a receptionist who listens to what someone says, figures out what they need, and directs them to the right department.

---

## The 7 Branches (+ Fallback)

Every customer message goes to **one** of these branches:

| Branch # | Name | When To Use |
|----------|------|-------------|
| 1 | Direct Reply | Simple greetings, thanks, goodbyes |
| 2 | Product Lookup | Customer is asking about products |
| 3 | FAQ Answer | Customer has a general question |
| 4 | Draft Order Creation | Customer wants to buy something |
| 5 | Order Tracking | Customer wants to check their order status |
| 6 | Human Support Handoff | Customer needs a real person |
| 7 | Style Recommendation | Customer wants outfit/styling advice |

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
Does it contain style/outfit/recommendation keywords?
    YES --> Branch 7: Style Recommendation
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

**Why this order?** More specific intents (tracking, style) are checked first. Style is checked BEFORE ordering because "What should I wear to a party?" is style advice, not a purchase intent. Greetings are checked last because phrases like "Hi, I want to track my order" should go to tracking, not be treated as just a greeting.

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

### Rules
- Always end with an invitation to continue the conversation
- Keep responses short and friendly
- If the greeting includes another intent (like "Hi, do you have jeans?"), route to the OTHER intent instead

---

## Branch 2: Product Lookup

### Purpose
Find and display products from the Google Sheets Products tab.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Browsing | show me, do you have, what products, browse, catalog, collection |
| Product types | t-shirt, tshirt, jeans, hoodie, jacket, dress, skirt, pants, shorts, sweater, shirt |
| Attributes | black, white, red, blue, small, medium, large, xl, cotton, size |
| Price-related | how much, price, cost, cheap, affordable, expensive, under $X |
| Availability | in stock, available, do you carry, do you sell |

### Example Messages

| Customer Says | What The Bot Does |
|--------------|-------------------|
| "Do you have black t-shirts?" | Searches Products tab for Category="T-Shirts" AND Colors_Available contains "Black" |
| "Show me your hoodies" | Searches Products tab for Category="Hoodies" |
| "What do you have under $40?" | Searches Products tab for Price < 40 |
| "Is the slim fit jeans available in size 32?" | Searches Products tab for Product_Name contains "Slim Fit Jeans" AND Sizes_Available contains "32" |
| "What colors does the hoodie come in?" | Searches Products tab for Category="Hoodies", returns Colors_Available |

### Reply Logic

**If products found (1-5 results):**
```
I found [X] product(s) matching your request:

1. [Product Name]
   Price: $[Price]
   Sizes: [Sizes_Available]
   Colors: [Colors_Available]
   Status: [In Stock / Out of Stock]

Would you like to order any of these, or would you like me to search for something else?
```

**If products found (more than 5):**
```
I found [X] products! Here are the top 5:

[show 5 products]

Would you like me to narrow it down? You can tell me a preferred size, color, or price range.
```

**If NO products found:**
```
I could not find any products matching "[search term]". 

Here are our available categories:
- T-Shirts
- Jeans
- Hoodies
- Dresses
- Accessories

Would you like to browse one of these, or can I help with something else?
```

**If product is out of stock:**
```
I found the [Product Name], but it is currently out of stock.

Would you like me to:
1. Show you similar products that ARE in stock?
2. Create a support ticket so we can notify you when it is back?
```

---

## Branch 3: FAQ Answer

### Purpose
Answer common questions using the FAQ tab in Google Sheets.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Shipping | shipping, delivery, how long, ship, when will it arrive, shipping cost, free shipping |
| Returns | return, refund, exchange, send back, money back, return policy |
| Sizing | size, sizing, fit, measurement, size guide, size chart, what size |
| Payment | payment, pay, credit card, debit, paypal, apple pay, payment methods |
| General | policy, hours, location, contact, about, where are you, who are you |
| Discount | discount, coupon, promo code, sale, deal, offer |

### Example Messages

| Customer Says | Bot Searches FAQ Keywords For |
|--------------|-------------------------------|
| "How long does shipping take?" | shipping, long, take |
| "Can I return something?" | return |
| "What size should I get?" | size |
| "Do you accept PayPal?" | payment, paypal |
| "Do you have any discounts?" | discount |
| "What are your business hours?" | hours |

### Matching Logic

1. Extract keywords from customer message (remove common words like "the", "a", "do", "you")
2. Compare against the Keywords column in the FAQ tab
3. Count how many keywords match each FAQ entry
4. Return the FAQ entry with the MOST matches
5. If multiple entries tie, return all tied entries

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

You can also ask about: shipping, returns, sizing, or payments.
```

---

## Branch 4: Draft Order Creation

### Purpose
Collect order details from the customer and save a draft order to Google Sheets.

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
| "I'll take 2 of the slim fit jeans in blue, size 32" | Most info provided, ask for name/email |
| "Can I buy that hoodie?" | Start order flow, ask for size and color |
| "Yes, I want to order it in large" | Continue order flow (customer already browsing) |

### Order Collection Flow

The bot needs to collect these pieces of information (in any order):

```
Step 1: Which product? (Product_ID or name)
        |
        v
Step 2: What size?
        |
        v
Step 3: What color?
        |
        v
Step 4: How many? (default: 1)
        |
        v
Step 5: Customer name
        |
        v
Step 6: Customer email
        |
        v
Step 7: Any special notes? (optional)
        |
        v
Step 8: Confirm order summary
        |
        v
Step 9: Save to Google Sheets
```

### Conversation Examples

**Example 1: Customer provides info gradually**
```
Customer: "I want to buy a black t-shirt"
Bot:      "Great choice! The Classic Black T-Shirt is $29.99. 
           What size would you like? Available: S, M, L, XL"
Customer: "Large please"
Bot:      "Size L, got it! To create your order, I'll need:
           - Your full name
           - Your email address"
Customer: "Sarah Johnson, sarah@email.com"
Bot:      "Here's your order summary:
           
           Product: Classic Black T-Shirt
           Size: L | Color: Black | Qty: 1
           Total: $29.99
           Name: Sarah Johnson
           Email: sarah@email.com
           
           Shall I confirm this order?"
Customer: "Yes"
Bot:      "Order created! Your order number is ORD-20250120-001.
           Status: Draft (our team will confirm shortly).
           You'll receive updates at sarah@email.com."
```

**Example 2: Customer provides most info at once**
```
Customer: "I'd like to order 2 Classic Black T-Shirts in size M"
Bot:      "2x Classic Black T-Shirt, Size M, Black - Total: $59.98
           To complete your order, I need your name and email address."
Customer: "John Smith, john@email.com"
Bot:      "[shows summary and confirms]"
```

### Validation Rules

| Check | What To Do If It Fails |
|-------|----------------------|
| Product does not exist | "I could not find that product. Would you like me to show you what we have?" |
| Size not available | "Sorry, that size is not available. Available sizes are: [list]. Which would you prefer?" |
| Color not available | "That color isn't available for this product. Available colors are: [list]." |
| Product out of stock | "Sorry, that product is currently out of stock. Can I show you similar items?" |
| Invalid email format | "That email doesn't look quite right. Could you double-check it?" |
| Customer wants to cancel mid-flow | "No problem! Order cancelled. Let me know if you change your mind." |

---

## Branch 5: Order Tracking

### Purpose
Look up order status and shipping information from Google Sheets.

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
| "Where is my order?" | Ask for order number or email |
| "Track order ORD-20250120-001" | Search Tracking tab directly |
| "Has my package shipped yet?" | Ask for order number or email |
| "When will order ORD-20250120-001 arrive?" | Search Tracking tab, return estimated delivery |

### Lookup Flow

```
Step 1: Does the message contain an order number (ORD-XXXXX)?
        |
        YES --> Search Tracking tab by Order_ID
        NO  --> Ask: "I'd be happy to check! Could you provide your 
                     order number (starts with ORD-) or your email address?"
        |
        v
Step 2: Search Google Sheets Tracking tab
        |
        v
Step 3: Return results or error message
```

### Reply Logic

**If tracking info found:**
```
Here's the status for order [Order_ID]:

Status: [Shipping_Status]
Carrier: [Carrier]
Shipped: [Shipped_Date]
Estimated Delivery: [Estimated_Delivery]
Tracking Number: [Tracking_Number]

Track your package: [Tracking_URL]

Is there anything else I can help with?
```

**If order found but NOT shipped yet:**
```
I found your order [Order_ID]!

Current Status: [Order_Status from Orders tab]

Your order hasn't shipped yet. Here's what the statuses mean:
- Draft: Awaiting confirmation
- Confirmed: Being prepared
- Processing: Almost ready to ship

You'll receive a notification when it ships. Anything else I can help with?
```

**If order NOT found:**
```
I could not find an order with that number/email. This could mean:
- The order number might have a typo (format is: ORD-YYYYMMDD-XXX)
- The order might be under a different email

Would you like to:
1. Try a different order number or email?
2. Connect with our support team?
```

---

## Branch 7: Style Recommendation (NEW in v4)

### Purpose
Provide outfit and styling advice based on the customer's needs, occasion, or preferences.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Style advice | style, outfit, what to wear, what goes with, how to style, look, fashion |
| Occasion | party, date, wedding, interview, casual, work, formal, weekend, brunch |
| Matching | pair with, match, combine, coordinate, goes well with, complement |
| Recommendations | recommend, suggest, advice, tips, ideas, inspiration |

### Example Messages

| Customer Says | Bot Action |
|--------------|------------|
| "What should I wear to a party?" | Suggest occasion-appropriate outfits from catalog |
| "What goes with black jeans?" | Suggest tops/accessories that pair well |
| "Can you recommend a casual weekend outfit?" | Suggest relaxed outfit combinations |
| "How do I style a hoodie?" | Suggest layering and pairing options |
| "I need outfit ideas for a job interview" | Suggest professional-looking combinations |

### Reply Logic

**Standard style response:**
```
Great question! Here are some styling ideas:

Based on your request for [occasion/item]:

1. [Outfit suggestion 1 - using products from catalog]
2. [Outfit suggestion 2 - using products from catalog]
3. [Outfit suggestion 3 - mix and match]

Would you like to:
- See more options?
- Order any of these items?
- Get advice for a different occasion?
```

### Rules
- Always reference actual products from the catalog when possible
- If no products match, give general style advice and mention what IS available
- Style intent takes priority over order_create (asking "what should I wear" is advice, not buying)
- If the customer says "I want that" after a style suggestion, route their next message to order_create

---

## Branch 6: Human Support Handoff

### Purpose
Connect the customer to a real human when the bot cannot help, or when they explicitly ask.

### Trigger Keywords

| Category | Keywords/Phrases |
|----------|-----------------|
| Direct request | talk to a human, real person, speak to someone, agent, representative, manager |
| Frustration | this isn't helping, useless, doesn't work, I'm frustrated, terrible service |
| Complex issues | complaint, wrong item, damaged, broken, overcharged, fraud, urgent |
| Repeated failure | (customer has asked 3+ times without resolution) |

### Automatic Handoff Triggers

The bot should AUTOMATICALLY hand off to a human (without the customer asking) when:

| Situation | Why |
|-----------|-----|
| Customer is clearly angry (profanity, ALL CAPS, exclamation marks!!!) | Humans handle emotions better |
| Bot has failed to understand 3 messages in a row | Bot is not helping |
| Customer mentions legal action or formal complaint | Needs human attention |
| Issue involves payment/billing problems | Too sensitive for a bot |
| Customer says something about damaged/wrong items | Requires human judgment |

### Priority Assignment Rules

| Condition | Priority | Response Time Goal |
|-----------|----------|-------------------|
| Customer is angry OR mentions damaged/wrong item | **High** | Within 2 hours |
| Customer asks for exchange, has sizing issues, general complaint | **Medium** | Within 4 hours |
| Bot could not answer a question, customer just prefers humans | **Low** | Within 24 hours |

### Handoff Conversation

```
Customer: "I want to talk to a real person"
Bot:      "Of course! I'll connect you with our team right away.
           
           To help them assist you faster, could you briefly tell me what you need help with?
           (Or just say 'skip' and I'll transfer you now)"
Customer: "I received the wrong size"
Bot:      "I'm sorry about that! I've created a support ticket:
           
           Ticket: TKT-20250120-001
           Issue: Received wrong size
           Priority: Medium
           
           Our team will contact you within 4 hours (during business hours: Mon-Fri, 9 AM - 6 PM).
           
           Is there anything else I can help with while you wait?"
```

### Information To Collect Before Handoff

Try to collect (but don't force if customer is frustrated):

1. Customer name
2. Customer email
3. Brief description of the issue
4. Related order number (if applicable)

If the customer says "skip" or is clearly frustrated, create the ticket with whatever info you have.

---

## Fallback Logic (When No Branch Matches)

### When It Triggers
- Message doesn't match any keyword patterns
- Message is gibberish or very short/unclear
- Message is in a language the bot doesn't support

### Fallback Response

```
I'm not sure I understood that. Here's what I can help with:

1. Products - "Show me t-shirts" or "What's in stock?"
2. Questions - "What's your return policy?" or "Shipping info"
3. Place an order - "I want to buy..." 
4. Track an order - "Where is my order ORD-XXXXX?"
5. Talk to our team - "Connect me with support"

Which of these can I help you with?
```

### Fallback Rules
- Show the fallback menu a maximum of 2 times
- If the customer triggers fallback 3 times in a row, automatically hand off to human support
- Log unclear messages so you can improve keyword lists later

---

## Edge Cases and Special Situations

### Customer sends multiple intents in one message

**Example:** "Hi, I want to track my order and also buy a new t-shirt"

**Rule:** Handle the FIRST actionable intent. After completing it, ask about the second.

```
Bot: "Let me help you track your order first! What's your order number?
     (I'll help you with the t-shirt right after)"
```

### Customer changes mind mid-conversation

**Example:** In the middle of placing an order, customer says "actually, never mind"

**Rule:** Acknowledge and reset.

```
Bot: "No problem at all! Order cancelled. Is there something else I can help you with?"
```

### Customer sends just an emoji

**Rule:** Treat as unclear/greeting depending on the emoji.

| Emoji | Treat As |
|-------|----------|
| Thumbs up, smiley | Affirmation ("Got it! Anything else?") |
| Sad face, angry face | Concern ("Is everything okay? How can I help?") |
| Question mark | Unclear ("What would you like to know?") |
| Other | Fallback menu |

### Customer sends a photo/image

**Rule:** The bot cannot analyze images.

```
Bot: "I received your image, but I'm not able to analyze photos.
     Could you describe what you need help with in text? 
     Or I can connect you with our team who can view the image."
```

### Message is very long (100+ words)

**Rule:** Focus on the first sentence or the most specific keywords. If unclear, summarize what you think they want and confirm.

```
Bot: "I want to make sure I help you correctly. It sounds like you need help with [topic]. Is that right?"
```

---

## Keyword Matching Tips for n8n

### How To Set Up Keyword Matching in n8n (Without AI)

Use a **Switch** node with these conditions:

1. **Check the message** (convert to lowercase first using a Set node)
2. **Use "contains" conditions** for each branch
3. **Order matters** - put more specific checks at the top

Example Switch conditions (checked in order):
```
1. Message contains "track" OR "where is my order" OR "ORD-" → Branch 5
2. Message contains "style" OR "outfit" OR "what to wear" OR "goes with" OR "recommend" → Branch 7
3. Message contains "buy" OR "order" OR "purchase" OR "I want" → Branch 4  
4. Message contains "speak to" OR "human" OR "help me" OR frustrated language → Branch 6
5. Message contains product keywords (t-shirt, jeans, hoodie, etc.) → Branch 2
6. Message contains FAQ keywords (shipping, return, size, payment) → Branch 3
7. Message contains greeting keywords (hi, hello, thanks, bye) → Branch 1
8. Default/else → Fallback
```

### How To Set Up With AI (Easier but requires API key)

Use an **AI Agent** node or **OpenAI** node with this prompt:

```
Classify the following customer message into exactly one category.
Return ONLY the category name, nothing else.

Categories (in priority order):
- order_track (asking about order status, delivery, tracking, mentions ORD- number)
- style_inquiry (asking for outfit advice, style recommendations, what goes with X, occasion-based suggestions)
- order_create (wants to buy/order something specific)
- human_support (wants a real person, is frustrated, complex issue)
- product_inquiry (asking about products, browsing, availability)
- faq (questions about shipping, returns, sizing, payment, policies)
- greeting (simple hi, thanks, bye, ok)
- unknown (cannot determine intent)

IMPORTANT: style_inquiry is checked BEFORE order_create. If the message asks for style advice or recommendations (even if it mentions products), classify as style_inquiry.

Customer message: "{customer_message}"
```

---

## Summary Decision Table

| If the message is about... | Go to Branch | Priority in checking |
|---------------------------|-------------|---------------------|
| Tracking an existing order | 5 - Order Tracking | 1st (most specific) |
| Style advice, outfits, what to wear | 7 - Style Recommendation | 2nd (before ordering) |
| Buying/ordering a product | 4 - Draft Order | 3rd |
| Needing a real human | 6 - Human Handoff | 4th |
| Asking about products/browsing | 2 - Product Lookup | 5th |
| General questions (shipping, returns, etc.) | 3 - FAQ | 6th |
| Simple greeting/thanks/bye | 1 - Direct Reply | 7th |
| Cannot determine | Fallback | Last resort |
