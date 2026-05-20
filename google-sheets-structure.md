# Google Sheets Structure

## Overview

Your Google Sheets spreadsheet acts as the **database** for your chatbot. Think of it like a simple filing cabinet where each drawer (tab) holds different information.

You will create **one Google Sheets spreadsheet** with **6 tabs** (sheets) inside it.

---

## How To Set It Up

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank spreadsheet"** to create a new one
3. Name it: `Clothing Brand Chatbot Database`
4. You will see one tab at the bottom called "Sheet1" - rename it to "Products"
5. Click the **"+"** button at the bottom to add more tabs
6. Create all 6 tabs listed below

---

## Tab 1: Products

### What is this tab for?

This stores your entire product catalog. When a customer asks "Do you have black t-shirts?", the chatbot searches this tab to find matching products.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **Product_ID** | A unique code for each product | PROD-001 |
| B: **Product_Name** | The name of the product | Classic Black T-Shirt |
| C: **Category** | Type of clothing | T-Shirts |
| D: **Description** | A short description | 100% cotton crew neck t-shirt in solid black |
| E: **Price** | Price in your currency (number only) | 29.99 |
| F: **Currency** | Your currency code | USD |
| G: **Sizes_Available** | All available sizes, separated by commas | S, M, L, XL |
| H: **Colors_Available** | All available colors, separated by commas | Black, White, Navy |
| I: **In_Stock** | Is it available? (YES or NO) | YES |
| J: **Image_URL** | Link to product image (optional) | https://yoursite.com/images/black-tshirt.jpg |
| K: **Product_URL** | Link to product page (optional) | https://yoursite.com/products/black-tshirt |

### Example Rows

| Product_ID | Product_Name | Category | Description | Price | Currency | Sizes_Available | Colors_Available | In_Stock | Image_URL | Product_URL |
|-----------|-------------|----------|-------------|-------|----------|----------------|-----------------|----------|-----------|-------------|
| PROD-001 | Classic Black T-Shirt | T-Shirts | 100% cotton crew neck tee | 29.99 | USD | S, M, L, XL | Black, White, Navy | YES | | |
| PROD-002 | Slim Fit Jeans | Jeans | Stretch denim slim fit | 59.99 | USD | 28, 30, 32, 34, 36 | Blue, Black | YES | | |
| PROD-003 | Hoodie Pullover | Hoodies | Fleece-lined pullover hoodie | 49.99 | USD | S, M, L, XL, XXL | Grey, Black, Red | NO | | |

### Tips

- Keep Product_ID unique (never repeat the same ID)
- Use YES/NO for In_Stock (keeps it simple for the chatbot to read)
- Sizes and colors separated by commas make them easy to search

---

## Tab 2: FAQ

### What is this tab for?

This stores answers to common questions. When a customer asks "What is your return policy?", the chatbot searches this tab and sends back the answer.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **FAQ_ID** | A unique code for each question | FAQ-001 |
| B: **Category** | Topic area | Shipping |
| C: **Question** | The question customers ask | How long does shipping take? |
| D: **Keywords** | Words that help match this question (separated by commas) | shipping, delivery, how long, days, arrive |
| E: **Answer** | The full answer to send back | Standard shipping takes 5-7 business days. Express shipping takes 2-3 business days. |
| F: **Last_Updated** | When this answer was last checked | 2025-01-15 |

### Example Rows

| FAQ_ID | Category | Question | Keywords | Answer | Last_Updated |
|--------|----------|----------|----------|--------|-------------|
| FAQ-001 | Shipping | How long does shipping take? | shipping, delivery, how long, days, arrive | Standard shipping takes 5-7 business days. Express shipping takes 2-3 business days. | 2025-01-15 |
| FAQ-002 | Returns | What is your return policy? | return, refund, exchange, send back | You can return unworn items within 30 days for a full refund. Items must have tags attached. | 2025-01-15 |
| FAQ-003 | Sizing | How do I find my size? | size, sizing, fit, measurements, chart | Check our size guide at [your link]. Measure your chest and waist, then compare to the chart. | 2025-01-15 |
| FAQ-004 | Payment | What payment methods do you accept? | payment, pay, credit card, paypal | We accept Visa, Mastercard, PayPal, and Apple Pay. | 2025-01-15 |
| FAQ-005 | Shipping | Do you ship internationally? | international, worldwide, other countries, global | Yes! We ship to 50+ countries. International shipping takes 10-15 business days. | 2025-01-15 |

### Tips

- The **Keywords** column is very important - add as many words as customers might use
- Write answers in a friendly, helpful tone
- Update the Last_Updated date when you change an answer

---

## Tab 3: Orders

### What is this tab for?

This stores every order created through the chatbot. When a customer says "I want to buy the black t-shirt in size M", the chatbot creates a new row here.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **Order_ID** | A unique order number | ORD-20250120-001 |
| B: **Customer_Name** | Customer's name | Sarah Johnson |
| C: **Customer_Email** | Customer's email | sarah@example.com |
| D: **Customer_Phone** | Customer's phone (optional) | +1-555-0123 |
| E: **Product_ID** | Which product they want | PROD-001 |
| F: **Product_Name** | Name of the product | Classic Black T-Shirt |
| G: **Size** | Selected size | M |
| H: **Color** | Selected color | Black |
| I: **Quantity** | How many | 1 |
| J: **Total_Price** | Total cost | 29.99 |
| K: **Order_Status** | Current status | Draft |
| L: **Order_Date** | When the order was created | 2025-01-20 14:30:00 |
| M: **Notes** | Any special requests | Gift wrapping please |

### Order Status Values

| Status | What It Means |
|--------|--------------|
| **Draft** | Order created by chatbot, not yet confirmed |
| **Confirmed** | Customer confirmed, ready to process |
| **Processing** | Being prepared for shipping |
| **Shipped** | On the way to customer |
| **Delivered** | Customer received it |
| **Cancelled** | Order was cancelled |

### Tips

- Order_ID format suggestion: ORD-YYYYMMDD-XXX (date + number)
- All chatbot orders start as "Draft" status
- A human reviews draft orders before confirming them

---

## Tab 4: Tracking

### What is this tab for?

This stores shipping and tracking information. When a customer asks "Where is my order?", the chatbot searches this tab using their order number or email.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **Order_ID** | The order number (must match Orders tab) | ORD-20250120-001 |
| B: **Customer_Email** | Customer's email (for lookup) | sarah@example.com |
| C: **Tracking_Number** | Shipping carrier tracking number | 1Z999AA10123456784 |
| D: **Carrier** | Shipping company name | UPS |
| E: **Shipping_Status** | Current delivery status | In Transit |
| F: **Shipped_Date** | When it was shipped | 2025-01-21 |
| G: **Estimated_Delivery** | Expected arrival date | 2025-01-26 |
| H: **Delivered_Date** | Actual delivery date (blank if not delivered) | |
| I: **Tracking_URL** | Link to track the package | https://www.ups.com/track?tracknum=1Z999AA10123456784 |
| J: **Last_Updated** | When this was last updated | 2025-01-22 |

### Shipping Status Values

| Status | What It Means |
|--------|--------------|
| **Processing** | Order is being packed |
| **Shipped** | Picked up by carrier |
| **In Transit** | On the way |
| **Out for Delivery** | Arriving today |
| **Delivered** | Successfully delivered |
| **Failed Delivery** | Delivery attempt failed |
| **Returned** | Package sent back |

### Tips

- You update this tab manually or through your shipping provider
- The chatbot only READS from this tab (it does not write to it)
- Customers can look up orders by Order_ID or email address

---

## Tab 5: Customers

### What is this tab for?

This stores customer information so the chatbot can recognize returning customers and personalize responses.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **Customer_ID** | A unique customer code | CUST-001 |
| B: **Customer_Name** | Full name | Sarah Johnson |
| C: **Email** | Email address | sarah@example.com |
| D: **Phone** | Phone number (optional) | +1-555-0123 |
| E: **First_Contact_Date** | When they first messaged | 2025-01-15 |
| F: **Last_Contact_Date** | When they last messaged | 2025-01-20 |
| G: **Total_Orders** | How many orders they have made | 2 |
| H: **Notes** | Any notes about this customer | Prefers email communication |
| I: **Channel** | How they contact you | WhatsApp |

### Tips

- This tab helps you know your customers better
- The chatbot updates Last_Contact_Date each time someone messages
- You can use this to identify VIP customers (many orders)

---

## Tab 6: Support Tickets

### What is this tab for?

When the chatbot cannot help a customer (or the customer asks to speak to a human), it creates a support ticket here. Your team then checks this tab and follows up.

### Columns

| Column | What To Put Here | Example |
|--------|-----------------|---------|
| A: **Ticket_ID** | A unique ticket number | TKT-20250120-001 |
| B: **Customer_Name** | Who needs help | Sarah Johnson |
| C: **Customer_Email** | Their email | sarah@example.com |
| D: **Channel** | Where the conversation happened | WhatsApp |
| E: **Issue_Category** | Type of problem | Order Issue |
| F: **Issue_Description** | What the customer said/needs | Received wrong size, wants exchange |
| G: **Priority** | How urgent (Low, Medium, High) | Medium |
| H: **Status** | Current ticket status | Open |
| I: **Created_Date** | When the ticket was created | 2025-01-20 15:00:00 |
| J: **Assigned_To** | Team member handling it (blank initially) | |
| K: **Resolved_Date** | When it was resolved (blank until done) | |
| L: **Resolution_Notes** | How it was resolved (blank until done) | |

### Ticket Status Values

| Status | What It Means |
|--------|--------------|
| **Open** | New ticket, no one has looked at it yet |
| **In Progress** | A team member is working on it |
| **Waiting on Customer** | Waiting for the customer to reply |
| **Resolved** | Problem solved |
| **Closed** | Ticket completed and closed |

### Priority Rules

| Priority | When To Use |
|----------|-------------|
| **High** | Customer is angry, wrong item shipped, payment issues |
| **Medium** | Exchange requests, general complaints, sizing issues |
| **Low** | General questions the bot could not answer, feature requests |

### Tips

- Check this tab at least twice per day
- Respond to High priority tickets within 2 hours
- The chatbot creates tickets automatically during human handoff

---

## Quick Setup Checklist

- [ ] Create a new Google Sheets spreadsheet
- [ ] Name it "Clothing Brand Chatbot Database"
- [ ] Create the "Products" tab with all columns
- [ ] Create the "FAQ" tab with all columns
- [ ] Create the "Orders" tab with all columns
- [ ] Create the "Tracking" tab with all columns
- [ ] Create the "Customers" tab with all columns
- [ ] Create the "Support Tickets" tab with all columns
- [ ] Add at least 5 products to the Products tab
- [ ] Add at least 10 FAQ entries to the FAQ tab
- [ ] Add column headers in **Row 1** of every tab (this is important for n8n!)

---

## Important Rules

1. **Row 1 must ALWAYS be your column headers** - n8n reads column names from row 1
2. **Never leave blank rows** between data rows - this can confuse the lookup
3. **Be consistent** with spelling (e.g., always "YES" not sometimes "Yes" or "yes")
4. **Use the exact column names** shown above - the n8n workflow will reference these names
5. **Back up your spreadsheet** regularly (File > Make a copy)
