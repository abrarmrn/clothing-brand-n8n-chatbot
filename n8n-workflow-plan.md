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
[1. Chat Trigger] - Receives the message
        |
        v
[2. Message Classifier] - Decides what type of message it is
        |
        +--> Simple greeting/thanks --> [3. Direct Reply]
        |
        +--> Product question -------> [4. Product Lookup] --> [5. Format & Reply]
        |
        +--> FAQ question -----------> [6. FAQ Lookup] --> [7. Format & Reply]
        |
        +--> Wants to order ---------> [8. Collect Order Info] --> [9. Save Order] --> [10. Confirm Reply]
        |
        +--> Order tracking ---------> [11. Tracking Lookup] --> [12. Format & Reply]
        |
        +--> Needs human help -------> [13. Create Ticket] --> [14. Handoff Reply]
        |
        +--> Unknown/unclear --------> [15. Fallback Reply]
```

---

## All Nodes Explained

### Node 1: Chat Trigger

| Setting | Value |
|---------|-------|
| **Node Type** | Chat Trigger (or Webhook) |
| **Purpose** | Receives incoming customer messages |
| **What It Does** | Starts the workflow every time a customer sends a message |

**Beginner Tip:** This is always the FIRST node. Later, you will replace or connect this to WhatsApp, Messenger, or your website widget.

For initial testing, use n8n's built-in **Chat Trigger** node which gives you a test chat window.

---

### Node 2: Message Classifier (AI or IF/Switch Node)

| Setting | Value |
|---------|-------|
| **Node Type** | AI Agent node OR Switch node |
| **Purpose** | Reads the customer message and decides which branch to take |
| **What It Does** | Categorizes the message into one of 6 types |

**Option A - Using AI (Recommended):**
Use an AI node (like OpenAI or a basic AI Agent) with a prompt that classifies messages into categories:
- `greeting` - Simple hello/thanks
- `product_inquiry` - Asking about products
- `faq` - General questions about policies
- `order_create` - Wants to buy something
- `order_track` - Asking about an existing order
- `human_support` - Wants to talk to a person or is frustrated

**Option B - Using Switch Node (No AI needed):**
Use keyword matching with a Switch node to check if the message contains specific words. (See `chatbot-branching-logic.md` for the keyword lists.)

---

### Node 3: Direct Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat (or Send Message) |
| **Purpose** | Replies to simple greetings and pleasantries |
| **What It Does** | Sends a friendly response without needing to look anything up |

**Example Responses:**
- "Hi" → "Hello! Welcome to [Brand Name]! How can I help you today? I can help with products, orders, tracking, or answer questions."
- "Thanks" → "You're welcome! Is there anything else I can help you with?"
- "Bye" → "Goodbye! Have a great day. Come back anytime!"

---

### Node 4: Product Lookup

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets - Search Rows |
| **Purpose** | Searches the Products tab for matching items |
| **What It Does** | Looks for products matching the customer's request |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Products
- Search Column: Product_Name, Category, Colors_Available, or Description
- Search Value: Keywords extracted from the customer message

**What to search for:**
- If customer says "black t-shirt" → search for "black" in Colors_Available AND "t-shirt" in Category
- If customer says "jeans" → search for "jeans" in Category or Product_Name

---

### Node 5: Format Product Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Set node + Respond to Chat |
| **Purpose** | Formats the search results into a nice message |
| **What It Does** | Takes raw spreadsheet data and creates a readable reply |

**Example Reply Format:**
```
I found these products for you:

1. Classic Black T-Shirt
   Price: $29.99
   Sizes: S, M, L, XL
   Colors: Black, White, Navy
   In Stock: Yes

2. Premium Black T-Shirt V-Neck
   Price: $34.99
   Sizes: S, M, L, XL
   Colors: Black, Charcoal
   In Stock: Yes

Would you like to order any of these? Just tell me which one, your size, and color!
```

**If no products found:**
```
I could not find products matching your request. Could you try different keywords? Or I can show you our categories: T-Shirts, Jeans, Hoodies, Dresses, Accessories.
```

---

### Node 6: FAQ Lookup

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets - Search Rows |
| **Purpose** | Searches the FAQ tab for matching answers |
| **What It Does** | Finds the best FAQ answer based on keywords |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: FAQ
- Search Column: Keywords (Column D)
- Search Value: Words from the customer's message

**How matching works:**
The customer message is compared against the Keywords column. The FAQ entry with the most matching keywords wins.

---

### Node 7: Format FAQ Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Set node + Respond to Chat |
| **Purpose** | Sends the FAQ answer back to the customer |
| **What It Does** | Returns the answer from the FAQ tab in a friendly format |

**Example Reply:**
```
Great question! Here's what I found:

Q: How long does shipping take?
A: Standard shipping takes 5-7 business days. Express shipping takes 2-3 business days.

Does this answer your question? If not, I can connect you with our team!
```

---

### Node 8: Collect Order Information

| Setting | Value |
|---------|-------|
| **Node Type** | AI Agent or Set node (with memory/conversation) |
| **Purpose** | Gathers all details needed to create an order |
| **What It Does** | Asks the customer for: product, size, color, name, email |

**Information needed for an order:**
1. Which product (Product_ID or name)
2. Size
3. Color
4. Quantity
5. Customer name
6. Customer email
7. Any special notes

**Conversation flow:**
```
Customer: "I want to order the black t-shirt"
Bot: "Great choice! What size would you like? (S, M, L, XL)"
Customer: "Large"
Bot: "Size L, got it! Could I get your name and email to create the order?"
Customer: "Sarah Johnson, sarah@example.com"
Bot: "Let me confirm your order..."
```

---

### Node 9: Save Order to Google Sheets

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets - Append Row |
| **Purpose** | Saves the new order to the Orders tab |
| **What It Does** | Creates a new row with all order details |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Orders
- Operation: Append (add new row at the bottom)

**Data to save:**
- Generate Order_ID (format: ORD-YYYYMMDD-XXX)
- Customer_Name: from conversation
- Customer_Email: from conversation
- Product_ID: from product lookup
- Product_Name: from product lookup
- Size: from conversation
- Color: from conversation
- Quantity: from conversation
- Total_Price: Price x Quantity
- Order_Status: "Draft"
- Order_Date: Current date/time
- Notes: Any special requests

---

### Node 10: Order Confirmation Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Confirms the order was created |
| **What It Does** | Sends order summary back to customer |

**Example Reply:**
```
Your order has been created! Here's your summary:

Order Number: ORD-20250120-001
Product: Classic Black T-Shirt (Size L, Black)
Quantity: 1
Total: $29.99
Status: Draft (our team will confirm shortly)

You will receive a confirmation email at sarah@example.com.
Is there anything else I can help with?
```

---

### Node 11: Tracking Lookup

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets - Search Rows |
| **Purpose** | Finds shipping status for an order |
| **What It Does** | Searches the Tracking tab by Order_ID or email |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Tracking
- Search Column: Order_ID or Customer_Email
- Search Value: Order number or email from customer message

---

### Node 12: Format Tracking Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Set node + Respond to Chat |
| **Purpose** | Sends tracking information to customer |
| **What It Does** | Formats tracking data into a readable message |

**Example Reply (order found):**
```
Here is the tracking info for order ORD-20250120-001:

Status: In Transit
Carrier: UPS
Tracking Number: 1Z999AA10123456784
Shipped: January 21, 2025
Estimated Delivery: January 26, 2025

Track your package here: [tracking URL]

Is there anything else I can help with?
```

**Example Reply (order NOT found):**
```
I could not find tracking information for that order. This could mean:
- The order hasn't shipped yet
- The order number might be incorrect

Could you double-check your order number? It looks like: ORD-YYYYMMDD-XXX
Or I can look it up by your email address. Would you like me to connect you with our team?
```

---

### Node 13: Create Support Ticket

| Setting | Value |
|---------|-------|
| **Node Type** | Google Sheets - Append Row |
| **Purpose** | Creates a support ticket for human follow-up |
| **What It Does** | Adds a new row to the Support Tickets tab |

**Configuration:**
- Spreadsheet: Your "Clothing Brand Chatbot Database"
- Sheet/Tab: Support Tickets
- Operation: Append

**Data to save:**
- Generate Ticket_ID (format: TKT-YYYYMMDD-XXX)
- Customer_Name: from conversation
- Customer_Email: from conversation
- Channel: which platform they messaged from
- Issue_Category: type of problem
- Issue_Description: summary of what they said
- Priority: based on rules (see chatbot-branching-logic.md)
- Status: "Open"
- Created_Date: Current date/time

---

### Node 14: Human Handoff Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Tells the customer a human will follow up |
| **What It Does** | Confirms the ticket was created and sets expectations |

**Example Reply:**
```
I understand you'd like to speak with our team. I've created a support ticket for you.

Ticket Number: TKT-20250120-001
Our team will reach out to you within 2-4 hours during business hours (Mon-Fri, 9 AM - 6 PM).

If it's urgent, you can also email us at support@yourbrand.com.
Is there anything else I can help with in the meantime?
```

---

### Node 15: Fallback Reply

| Setting | Value |
|---------|-------|
| **Node Type** | Respond to Chat |
| **Purpose** | Handles messages the bot does not understand |
| **What It Does** | Asks the customer to rephrase or offers options |

**Example Reply:**
```
I'm not sure I understood that correctly. Here's what I can help you with:

1. Browse products - "Show me t-shirts" or "What do you have in black?"
2. Answer questions - "What's your return policy?" or "Do you ship internationally?"
3. Place an order - "I want to order..." or "I'd like to buy..."
4. Track an order - "Where is my order ORD-XXXXX?" or "Track my order"
5. Talk to a human - "I need help" or "Speak to someone"

Which would you like?
```

---

## Node Connections Summary

| From Node | To Node | Condition |
|-----------|---------|-----------|
| Chat Trigger | Message Classifier | Always (every message) |
| Message Classifier | Direct Reply | Category = greeting |
| Message Classifier | Product Lookup | Category = product_inquiry |
| Message Classifier | FAQ Lookup | Category = faq |
| Message Classifier | Collect Order Info | Category = order_create |
| Message Classifier | Tracking Lookup | Category = order_track |
| Message Classifier | Create Support Ticket | Category = human_support |
| Message Classifier | Fallback Reply | Category = unknown |
| Product Lookup | Format Product Reply | Always |
| FAQ Lookup | Format FAQ Reply | Always |
| Collect Order Info | Save Order | When all info collected |
| Save Order | Order Confirmation Reply | Always |
| Tracking Lookup | Format Tracking Reply | Always |
| Create Support Ticket | Human Handoff Reply | Always |

---

## Error Handling Nodes

You should add these extra nodes to handle problems:

### Error: Google Sheets Connection Failed

| Setting | Value |
|---------|-------|
| **Node Type** | Error Trigger + Respond to Chat |
| **Purpose** | Handles cases where Google Sheets cannot be reached |
| **Reply** | "I'm having trouble looking that up right now. Please try again in a moment, or I can connect you with our team." |

### Error: No Results Found

| Setting | Value |
|---------|-------|
| **Node Type** | IF node (check if results are empty) |
| **Purpose** | Handles cases where a search returns no results |
| **Reply** | A helpful message suggesting alternatives (shown in node descriptions above) |

### Error: Missing Information for Order

| Setting | Value |
|---------|-------|
| **Node Type** | IF node (check if required fields are filled) |
| **Purpose** | Makes sure all order details are collected before saving |
| **Reply** | "I still need your [missing field] to complete the order. Could you provide that?" |

---

## n8n Credentials You Will Need

| Credential | What For | How To Get It |
|-----------|----------|---------------|
| **Google Sheets OAuth2** | Reading/writing to your spreadsheet | In n8n, add a Google Sheets credential and follow the OAuth sign-in flow |
| **AI Service (Optional)** | For the Message Classifier (if using AI) | Sign up for OpenAI or use n8n's built-in AI features |

**Note:** Do NOT store API keys in this repository. You will set up credentials directly inside n8n.

---

## Testing Plan

Before connecting to real customers, test each branch:

| Test | What To Send | Expected Result |
|------|-------------|-----------------|
| Greeting | "Hi" | Friendly welcome message |
| Product search | "Show me black t-shirts" | Product list from Google Sheets |
| FAQ | "What is your return policy?" | Return policy answer |
| Order | "I want to buy the Classic Black T-Shirt in size M" | Order creation flow |
| Tracking | "Where is order ORD-20250120-001?" | Tracking info |
| Human handoff | "I want to speak to someone" | Support ticket created |
| Unknown | "asdfghjkl" | Fallback with menu options |
| Error | (disconnect Google Sheets temporarily) | Error message |
