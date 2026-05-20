# n8n Workflow Plan

## Overview

This document explains **which n8n nodes you need** and **how they connect together** to build the chatbot workflow.

Think of an n8n workflow like a flowchart: a message comes in, goes through decision points, and the right action happens automatically.

---

## What Is an n8n Node?

A **node** is a single step in your workflow. Each node does one specific job:

- Receive a message
- Make a decision
- Look up data in Google Sheets
- Send a reply

You connect nodes together with lines (called "connections") to build the full workflow.

---

## Workflow Architecture (The Big Picture)

Here is the flow from start to finish:

```
Customer sends message
        |
        v
[1. Chat Trigger] ─── Receives the message
        |
        v
[2. Identify Customer] ─── Check if returning customer (Customers tab)
        |
        v
[3. Message Classifier] ─── Decides what type of message it is
        |
        ├──> greeting ──────────> [4. Direct Reply]
        |
        ├──> product_inquiry ──> [5. Product Search] ──> [6. Format Product Reply]
        |
        ├──> faq ──────────────> [7. FAQ Search] ──> [8. Format FAQ Reply]
        |
        ├──> order_create ─────> [9. Collect Order Info] ──> [10. Validate Variant]
        |                              ──> [11. Save Order] ──> [12. Confirm Reply]
        |
        ├──> order_track ──────> [13. Tracking Lookup] ──> [14. Format Tracking Reply]
        |
        ├──> human_support ────> [15. Create Support Ticket] ──> [16. Handoff Reply]
        |
        └──> unknown ──────────> [17. Fallback Reply]
```

---

## All Nodes Explained

### Node 1: Chat Trigger

| Setting | Value |
|---------|-------|
| **Node Type** | Chat Trigger (or Webhook) |
| **Purpose** | Receives incoming customer messages |
| **What It Does** | Starts the workflow every time a customer sends a message |
| **Output Data** | Message text, sender ID (Chat_User_ID), channel name, timestamp |

**Beginner Tip:** This is always the FIRST node. Later, you will replace or connect this to WhatsApp, Messenger, or your website widget.

For initial testing, use n8n's built-in **Chat Trigger** node which gives you a test chat window.

---

### Node 2: Identify Customer

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Search Rows |
| **Purpose** | Check if this person has messaged before |
| **What It Does** | Searches the Customers tab by Chat_User_ID |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Customers
- Search Column: Chat_User_ID
- Search Value: The sender ID from the Chat Trigger

**If customer found:**
- Load their Customer_ID, Customer_Name, Total_Orders, Total_Spent
- Update Last_Contact_Date and Last_Message_Date
- Use their name in responses ("Hi Sarah!")

**If customer NOT found:**
- Continue without personalization (use "Hi there!")
- A new Customers row will be created later during order or support flows

**Why this node exists:** Returning customers get a personalized experience, and you can identify VIPs (high Total_Orders or Total_Spent).

---

### Node 3: Message Classifier

| Setting | Value |
|---------|-------|
| **Node Type** | AI Agent node OR Switch node |
| **Purpose** | Reads the customer message and decides which branch to take |
| **What It Does** | Categorizes the message into one of 7 types |

**Option A — Using AI (Recommended for accuracy):**

Use an AI node (like OpenAI or n8n's built-in AI) with this classification prompt:

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

**Option B — Using Switch Node (No AI needed, free):**

Use keyword matching with a Switch node. Check conditions in this priority order:
1. Contains "ORD-" or "track" or "where is my order" → `order_track`
2. Contains "buy" or "order" or "purchase" or "I want" or "I'll take" → `order_create`
3. Contains "human" or "person" or "agent" or angry language → `human_support`
4. Contains product words (t-shirt, jeans, hoodie, etc.) or "show me" or "in stock" → `product_inquiry`
5. Contains FAQ words (shipping, return, size guide, payment) → `faq`
6. Contains greetings (hi, hello, thanks, bye) → `greeting`
7. Default → `unknown`

---

### Node 4: Direct Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Replies to simple greetings and pleasantries |
| **What It Does** | Sends a friendly response without needing any data lookup |

**Personalization:** If Node 2 found the customer, use their name:
- Known customer: "Hi Sarah! Welcome back. How can I help you today?"
- Unknown customer: "Hello! Welcome to [Brand Name]. How can I help you today?"

**Example Responses:**
- "Hi" → "Hello! Welcome to [Brand Name]! I can help you with: browsing products, answering questions, placing orders, or tracking deliveries. What would you like to do?"
- "Thanks" → "You're welcome! Is there anything else I can help you with?"
- "Bye" → "Goodbye! Have a great day. Come back anytime!"

---

### Node 5: Product Search

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Search Rows |
| **Purpose** | Searches the Products tab for matching variants |
| **What It Does** | Finds products matching the customer's request by category, name, color, size, or keywords |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Products
- Filter: Active = "YES" (never show inactive products)
- Search Columns: Product_Name, Category, Color, Size, Search_Keywords

**Search Logic:**

| Customer Says | Search Strategy |
|--------------|----------------|
| "Show me t-shirts" | Category = "T-Shirts" AND Active = "YES" |
| "Black hoodies in large" | Category = "Hoodies" AND Color = "Black" AND Size = "L" AND Active = "YES" |
| "What do you have under $40?" | Price < 40 AND Active = "YES" AND In_Stock = "YES" |
| "Do you have the classic tee in medium?" | Product_Name contains "Classic" AND Size = "M" |

**Important:** Since each row is a variant, you may get multiple rows for the same product. Group results by Product_ID so you show each product once with all its available sizes/colors.

**Output:** List of matching product variants (or empty if nothing matches).

---

### Node 6: Format Product Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Set node + IF node + Respond to Chat |
| **Purpose** | Formats search results into a customer-friendly message |
| **What It Does** | Groups variants by product, shows availability, sends reply |

**Logic:**

1. **Group results by Product_ID** (so "Classic Black T-Shirt" shows once, not 12 times)
2. **For each product, list:** Name, Price, available sizes, available colors, stock status
3. **Show In_Stock variants only** (skip variants where In_Stock = "NO")

**If products found (1-5 unique products):**
```
I found [X] product(s) matching your request:

1. Classic Black T-Shirt — $29.99
   Sizes in stock: S, M, L
   Colors available: Black, White, Navy
   
2. Premium V-Neck Tee — $34.99
   Sizes in stock: S, M, L, XL
   Colors available: Black, Charcoal

Would you like to order any of these? Just tell me the product, size, and color!
```

**If products found (more than 5 unique products):**
```
I found [X] products! Here are the top 5:
[show 5 products]

Would you like me to narrow it down? Tell me your preferred size, color, or price range.
```

**If NO products found:**
```
I couldn't find any products matching "[search term]" that are currently in stock.

Our available categories are:
- T-Shirts
- Jeans  
- Hoodies
- Dresses
- Accessories

Would you like to browse one of these, or can I help with something else?
```

**If specific variant is out of stock:**
```
I found the Classic Black T-Shirt, but size M in Black is currently out of stock (Stock_Qty: 0).

Available alternatives:
- Size M in White (15 in stock)
- Size M in Navy (8 in stock)
- Size L in Black (8 in stock)

Would you like one of these instead, or shall I notify you when it's back?
```

---

### Node 7: FAQ Search

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Search Rows |
| **Purpose** | Searches the FAQ tab for matching answers |
| **What It Does** | Finds the best FAQ answer based on keywords |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: FAQ
- Filter: Active = "YES" (never show inactive FAQs)
- Search Column: Keywords
- Sort by: Sort_Order (ascending — lower numbers first)

**Matching Logic:**
1. Convert customer message to lowercase
2. Extract meaningful words (remove "the", "a", "do", "you", "is", etc.)
3. Compare extracted words against the Keywords column in each FAQ row
4. Count how many keywords match each FAQ entry
5. Return the FAQ entry with the MOST matches
6. If multiple entries tie, return the one with the lowest Sort_Order

---

### Node 8: Format FAQ Reply

| Setting | Value |
|---------|-------|
| **Node Type** | IF node + Respond to Chat |
| **Purpose** | Sends the FAQ answer or a helpful fallback |

**If FAQ match found:**
```
Great question! Here's what I found:

[Answer from FAQ tab]

Does this answer your question? If not, I can connect you with our team.
```

**If NO FAQ match found:**
```
I don't have a specific answer for that in my knowledge base.

Would you like me to:
1. Connect you with our support team?
2. Try rephrasing your question?

I can answer questions about: shipping, returns, sizing, payments, and discounts.
```

---

### Node 9: Collect Order Information

| Setting | Value |
|---------|-------|
| **Node Type** | AI Agent with memory OR multi-step Set nodes |
| **Purpose** | Gathers all details needed to create a complete order |
| **What It Does** | Asks the customer for product, size, color, shipping address, payment method |

**Information to collect (in order):**

| Step | Info Needed | Maps To Column | Required? |
|------|------------|----------------|-----------|
| 1 | Which product | Product_ID, Product_Name | Yes |
| 2 | What size | Size | Yes |
| 3 | What color | Color | Yes |
| 4 | How many | Quantity | Yes (default: 1) |
| 5 | Customer name | Customer_Name | Yes |
| 6 | Customer email | Customer_Email | Yes |
| 7 | Customer phone | Customer_Phone | Optional |
| 8 | Shipping address | Shipping_Address, City, Country | Yes |
| 9 | Delivery method | Delivery_Method | Yes (default: Standard) |
| 10 | Payment method | Payment_Method | Yes |
| 11 | Special notes | Notes | Optional |

**Conversation Example:**
```
Customer: "I want to buy the Classic Black T-Shirt in size M"
Bot:      "Great choice! Classic Black T-Shirt, Size M, Black — $29.99.
           How many would you like?"
Customer: "Just 1"
Bot:      "Got it! To complete your order I need:
           - Your full name
           - Email address
           - Shipping address (street, city, country)"
Customer: "Sarah Johnson, sarah@email.com, 123 Main St, New York, US"
Bot:      "Almost done! How would you like to pay?
           Options: Credit Card, PayPal, Apple Pay, Bank Transfer"
Customer: "PayPal"
Bot:      "And delivery preference? Standard (5-7 days, $5.99) or Express (2-3 days, $12.99)?"
Customer: "Standard"
Bot:      "[Shows full order summary for confirmation]"
```

---

### Node 10: Validate Variant

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Search Rows + IF node |
| **Purpose** | Verify the exact variant exists and is in stock before saving |
| **What It Does** | Searches Products tab for the specific Variant_SKU |

**Checks to perform:**

| Check | How | If Fails |
|-------|-----|----------|
| Product exists | Search by Product_ID, Active = "YES" | "I can't find that product. Want to see what we have?" |
| Variant exists | Search by Product_ID + Size + Color | "That size/color combo isn't available. Here's what we have: [list]" |
| Variant in stock | Check In_Stock = "YES" AND Stock_Qty > 0 | "Sorry, that's out of stock. Alternatives: [list other variants]" |
| Enough quantity | Stock_Qty >= requested Quantity | "We only have [X] left in that variant. Would you like [X] instead?" |

**On success:** Retrieve the Variant_SKU and Unit_Price, then proceed to Node 11.

---

### Node 11: Save Order to Google Sheets

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Append Row |
| **Purpose** | Saves the completed order to the Orders tab |
| **What It Does** | Creates a new row with all 25 order fields |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Orders
- Operation: Append (add new row at the bottom)

**Data mapping:**

| Column | Where The Data Comes From |
|--------|--------------------------|
| Order_ID | Generate: ORD-YYYYMMDD-XXX |
| Customer_ID | From Node 2 (if returning customer) or generate new CUST-XXX |
| Customer_Name | Collected in Node 9 |
| Customer_Email | Collected in Node 9 |
| Customer_Phone | Collected in Node 9 (or blank) |
| Product_ID | From Node 10 validation |
| Variant_SKU | From Node 10 validation |
| Product_Name | From Node 10 validation |
| Size | Collected in Node 9 |
| Color | Collected in Node 9 |
| Quantity | Collected in Node 9 |
| Unit_Price | From Node 10 (Products tab) |
| Total_Price | Unit_Price × Quantity |
| Currency | From Products tab |
| Shipping_Address | Collected in Node 9 |
| City | Collected in Node 9 |
| Country | Collected in Node 9 |
| Payment_Method | Collected in Node 9 |
| Payment_Status | "Pending" (always starts as pending) |
| Delivery_Method | Collected in Node 9 |
| Order_Status | "Draft" (always starts as draft) |
| Channel | From Chat Trigger (WhatsApp, Website, etc.) |
| Created_Date | Current date/time |
| Updated_Date | Current date/time |
| Notes | Collected in Node 9 (or blank) |

**Also do these updates:**
- If new customer: Append a row to the **Customers** tab with their info
- If existing customer: Update **Total_Orders** (+1) and **Last_Contact_Date** in Customers tab

---

### Node 12: Order Confirmation Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Confirms the order was created successfully |
| **What It Does** | Sends a complete order summary with next steps |

**Reply template:**
```
Your order has been created! Here's your summary:

Order Number: ORD-20250120-001
Product: Classic Black T-Shirt
Variant: Size M, Black
Quantity: 1
Unit Price: $29.99
Total: $29.99
Delivery: Standard (5-7 business days)
Payment: PayPal (pending)

Shipping to: 123 Main St, New York, US

Status: Draft — our team will confirm and send payment instructions shortly.
You'll receive updates at sarah@example.com.

Is there anything else I can help with?
```

---

### Node 13: Tracking Lookup

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Search Rows |
| **Purpose** | Finds shipping status for an order |
| **What It Does** | Searches the Tracking tab by Order_ID or Customer_Email |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Tracking
- Search Column: Order_ID (primary) or Customer_Email (fallback)

**Lookup flow:**
1. Check if message contains an order number (pattern: ORD-XXXXXXXX-XXX)
2. If yes → search Tracking tab by Order_ID
3. If no → check if we know the customer from Node 2
4. If known customer → search by their email
5. If unknown → ask: "Could you provide your order number (starts with ORD-) or email address?"

**If order not in Tracking tab:** Also check the Orders tab — the order may not have shipped yet. Return the Order_Status instead.

---

### Node 14: Format Tracking Reply

| Setting | Value |
|---------|-------|
| **Node Type** | IF node + Set node + Respond to Chat |
| **Purpose** | Sends tracking information or helpful alternatives |

**If tracking info found:**
```
Here's the status for order ORD-20250120-001:

Status: In Transit
Carrier: UPS
Tracking Number: 1Z999AA10123456784
Shipped: January 21, 2025
Estimated Delivery: January 26, 2025
Last Updated: January 22, 2025

Track your package: https://www.ups.com/track?tracknum=1Z999AA10123456784

Is there anything else I can help with?
```

**If order exists but NOT shipped yet:**
```
I found your order ORD-20250120-001!

Current Status: Processing
Payment: Paid

Your order hasn't shipped yet. Here's what the statuses mean:
- Draft → Awaiting confirmation
- Confirmed → Payment received, being prepared
- Processing → Almost ready to ship!

You'll get a notification with tracking info once it ships. Anything else I can help with?
```

**If order NOT found:**
```
I couldn't find an order with that number. This could mean:
- The order number might have a typo (format: ORD-YYYYMMDD-XXX)
- The order might be under a different email

Would you like to:
1. Try a different order number or email?
2. Connect with our support team?
```

---

### Node 15: Create Support Ticket

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets — Append Row |
| **Purpose** | Creates a detailed support ticket for human follow-up |
| **What It Does** | Adds a new row to the Support_Tickets tab with full context |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Support_Tickets
- Operation: Append

**Data mapping:**

| Column | Where The Data Comes From |
|--------|--------------------------|
| Ticket_ID | Generate: TKT-YYYYMMDD-XXX |
| Conversation_ID | From Chat Trigger (session/conversation ID) |
| Customer_ID | From Node 2 (or blank if unknown) |
| Customer_Name | From Node 2 or collected during handoff |
| Customer_Email | From Node 2 or collected during handoff |
| Customer_Phone | From Node 2 or collected during handoff |
| Channel | From Chat Trigger |
| Issue_Category | Detected from message (Order Issue, Product Question, Complaint, etc.) |
| Issue_Description | Brief summary of what the customer needs |
| Last_User_Message | The exact last message the customer sent (copy it verbatim) |
| Bot_Summary | Auto-generated summary of the conversation so far |
| Handoff_Reason | Why handoff is happening (customer_requested, bot_failed_3x, angry_customer, complex_issue, policy_exception) |
| Priority | Based on rules (see below) |
| Status | "Open" |
| Created_Date | Current date/time |
| Assigned_To | (blank — team will assign) |
| Resolved_Date | (blank — not resolved yet) |
| Resolution_Notes | (blank — not resolved yet) |

**Priority assignment logic:**

| Condition | Priority |
|-----------|----------|
| Angry language, ALL CAPS, profanity, mentions "legal" or "lawyer" | High |
| Wrong item received, damaged item, payment/billing issue | High |
| Exchange request, sizing issue, general complaint | Medium |
| Bot failed to understand, customer prefers human, general question | Low |

**Bot_Summary generation:** If using an AI node, ask it to summarize the conversation in 1-2 sentences. If not using AI, concatenate the last 3 customer messages.

---

### Node 16: Human Handoff Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Tells the customer a human will follow up |
| **What It Does** | Confirms the ticket and sets response time expectations |

**Reply template:**
```
I understand you need help from our team. I've created a support ticket for you.

Ticket Number: TKT-20250120-001
Priority: [Medium]
Issue: [Brief description]

Our team will reach out to you within [response time based on priority]:
- High priority: Within 2 hours
- Medium priority: Within 4 hours  
- Low priority: Within 24 hours

(Business hours: Mon-Fri, 9 AM - 6 PM)

If urgent, you can also email: support@yourbrand.com

Is there anything else I can help with while you wait?
```

---

### Node 17: Fallback Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Handles messages the bot does not understand |
| **What It Does** | Offers a menu of options |

**Reply:**
```
I'm not sure I understood that. Here's what I can help with:

1. 🛍️ Products — "Show me t-shirts" or "What's in stock?"
2. ❓ Questions — "What's your return policy?" or "Shipping info"
3. 🛒 Place an order — "I want to buy..." 
4. 📦 Track an order — "Where is my order ORD-XXXXX?"
5. 💬 Talk to our team — "Connect me with support"

Which of these can I help you with?
```

**Fallback counter logic:**
- First unknown message: Show menu above
- Second unknown message in a row: Show menu + "If you'd prefer to speak with a person, just say 'human support'"
- Third unknown message in a row: Automatically trigger Node 15 (Create Support Ticket) with Handoff_Reason = "bot_failed_3x"

---

## Node Connections Summary

| From Node | To Node | Condition |
|-----------|---------|-----------|
| 1. Chat Trigger | 2. Identify Customer | Always (every message) |
| 2. Identify Customer | 3. Message Classifier | Always |
| 3. Message Classifier | 4. Direct Reply | Category = greeting |
| 3. Message Classifier | 5. Product Search | Category = product_inquiry |
| 3. Message Classifier | 7. FAQ Search | Category = faq |
| 3. Message Classifier | 9. Collect Order Info | Category = order_create |
| 3. Message Classifier | 13. Tracking Lookup | Category = order_track |
| 3. Message Classifier | 15. Create Support Ticket | Category = human_support |
| 3. Message Classifier | 17. Fallback Reply | Category = unknown |
| 5. Product Search | 6. Format Product Reply | Always |
| 7. FAQ Search | 8. Format FAQ Reply | Always |
| 9. Collect Order Info | 10. Validate Variant | When all info collected |
| 10. Validate Variant | 11. Save Order | Validation passes |
| 10. Validate Variant | (back to 9) | Validation fails (ask again) |
| 11. Save Order | 12. Order Confirmation Reply | Always |
| 13. Tracking Lookup | 14. Format Tracking Reply | Always |
| 15. Create Support Ticket | 16. Handoff Reply | Always |

---

## Error Handling

### Error: Google Sheets Connection Failed

| Setting | Value |
|---------|-------|
| **Where** | On every Google Sheets node |
| **How** | Add an "Error" output connection from each Google Sheets node |
| **Reply** | "I'm having trouble looking that up right now. Please try again in a moment, or I can connect you with our team." |

### Error: No Search Results

| Setting | Value |
|---------|-------|
| **Where** | After Nodes 5, 7, 13 (any search node) |
| **How** | IF node checks if results array is empty |
| **Reply** | A helpful message with alternatives (shown in node descriptions above) |

### Error: Variant Out of Stock During Order

| Setting | Value |
|---------|-------|
| **Where** | Node 10 (Validate Variant) |
| **How** | Check Stock_Qty >= requested Quantity |
| **Reply** | "That variant is out of stock. Here are alternatives: [list other variants of same product]" |

### Error: Invalid Data from Customer

| Setting | Value |
|---------|-------|
| **Where** | Node 9 (Collect Order Info) |
| **How** | Validate email format, ensure required fields are filled |
| **Reply** | "That doesn't look quite right. Could you double-check your [email/address/etc.]?" |

### Error: Fallback Loop (3 failures)

| Setting | Value |
|---------|-------|
| **Where** | Node 17 (Fallback Reply) |
| **How** | Counter tracks consecutive unknown messages |
| **Action** | Auto-trigger Node 15 (Create Support Ticket) with Handoff_Reason = "bot_failed_3x" |

---

## n8n Credentials You Will Need

| Credential | What For | How To Get It |
|-----------|----------|---------------|
| **Google Sheets OAuth2** | Reading/writing to your spreadsheet | In n8n: Settings > Credentials > Add > Google Sheets OAuth2 > Sign in with Google |
| **AI Service (Optional)** | For the Message Classifier and Bot_Summary generation | Sign up at OpenAI or use n8n's built-in AI features |

**Note:** Do NOT store API keys in this repository. Set up credentials directly inside n8n.

---

## Data Flow Summary

Here is how data moves between Google Sheets tabs during the workflow:

| Action | Reads From | Writes To |
|--------|-----------|-----------|
| Identify customer | Customers | Customers (update dates) |
| Product search | Products | — |
| FAQ search | FAQ | — |
| Create order | Products (validate) | Orders, Customers (update totals) |
| Track order | Tracking, Orders | — |
| Human handoff | Customers | Support_Tickets |

---

## Testing Checklist

Before connecting to real customers, test each path:

| # | Test | What To Send | Expected Result |
|---|------|-------------|-----------------|
| 1 | Greeting | "Hi" | Welcome message (personalized if known customer) |
| 2 | Product found | "Show me black t-shirts" | Grouped product list with sizes/stock |
| 3 | Product not found | "Show me purple sandals" | Not found + category list |
| 4 | Specific variant | "Do you have the hoodie in large grey?" | Single variant result with stock qty |
| 5 | Out of stock | Ask for a variant with Stock_Qty = 0 | Out of stock + alternatives |
| 6 | FAQ found | "What's your return policy?" | Correct FAQ answer |
| 7 | FAQ not found | "Do you do custom embroidery?" | No answer + options |
| 8 | Order start | "I want to buy a Classic Black T-Shirt in M" | Starts order collection |
| 9 | Order complete | Provide all details | Order saved, confirmation with all fields |
| 10 | Order validation fail | Order a variant that is out of stock | Error + alternatives |
| 11 | Tracking found | "Track order ORD-20250120-001" | Full tracking details |
| 12 | Tracking not found | "Track order ORD-99999" | Not found + suggestions |
| 13 | Human handoff | "Talk to a real person" | Ticket created + confirmation |
| 14 | Angry handoff | "THIS IS TERRIBLE SERVICE!" | High priority ticket |
| 15 | Fallback | "asdfghjkl" | Menu of options |
| 16 | Fallback x3 | Three gibberish messages in a row | Auto-handoff to human |
| 17 | Error | Disconnect Google Sheets temporarily | Graceful error message |
