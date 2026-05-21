# Google Sheets Structure

## Overview

Your Google Sheets spreadsheet acts as the **database** for your chatbot. Think of it like a simple filing cabinet where each drawer (tab) holds different information.

You will create **one Google Sheets spreadsheet** with **6 tabs** (sheets) inside it.

---

## How To Set It Up

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **"Blank spreadsheet"** to create a new one
3. Name it: `Clothing Brand Chatbot Database`
4. You will see one tab at the bottom called "Sheet1" — rename it to "Products"
5. Click the **"+"** button at the bottom to add more tabs
6. Create all 6 tabs listed below

---

## Tab 1: Products

### What is this tab for?

This stores your entire product catalog. **Each row represents one product variant** (a specific size + color combination). This way the chatbot can instantly check if a specific variant is in stock without doing extra calculations.

### Why one row per variant?

If you sell a "Classic Black T-Shirt" in 4 sizes and 3 colors, that is 12 variants (12 rows). This makes stock tracking precise — you know exactly how many size-M black t-shirts you have left.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **Product_ID** | A unique code for the product (same across all its variants) | Groups all variants of the same product together |
| B | **Variant_SKU** | A unique code for this specific variant | Identifies the exact item (product + size + color) |
| C | **Product_Name** | The name of the product | What the customer sees |
| D | **Category** | Type of clothing | Helps the bot filter (e.g., "show me hoodies") |
| E | **Description** | A short product description | Bot can share this with customers |
| F | **Price** | Price for this variant (number only) | Used to calculate order totals |
| G | **Currency** | Your currency code | Shows the correct currency symbol |
| H | **Size** | The size of THIS variant | Exact size for stock checking |
| I | **Color** | The color of THIS variant | Exact color for stock checking |
| J | **Stock_Qty** | How many units you have in stock (number) | Bot can say "only 3 left!" or "out of stock" |
| K | **In_Stock** | Is it available? (YES or NO) | Quick check — bot skips items marked NO |
| L | **Image_URL** | Link to product image (optional) | Bot can show product photos |
| M | **Product_URL** | Link to product page (optional) | Bot can link to your website |
| N | **Search_Keywords** | Extra words customers might use to find this | Improves search accuracy (e.g., "tee, crew neck, casual") |
| O | **Active** | Is this product listed? (YES or NO) | Lets you hide products without deleting them |

### Example Rows

| Product_ID | Variant_SKU | Product_Name | Category | Description | Price | Currency | Size | Color | Stock_Qty | In_Stock | Image_URL | Product_URL | Search_Keywords | Active |
|-----------|-------------|--------------|----------|-------------|-------|----------|------|-------|-----------|----------|-----------|-------------|----------------|--------|
| PROD-001 | PROD-001-BLK-S | Classic Black T-Shirt | T-Shirts | 100% cotton crew neck tee | 29.99 | USD | S | Black | 15 | YES | | | tee, crew neck, basic, cotton | YES |
| PROD-001 | PROD-001-BLK-M | Classic Black T-Shirt | T-Shirts | 100% cotton crew neck tee | 29.99 | USD | M | Black | 22 | YES | | | tee, crew neck, basic, cotton | YES |
| PROD-001 | PROD-001-BLK-L | Classic Black T-Shirt | T-Shirts | 100% cotton crew neck tee | 29.99 | USD | L | Black | 8 | YES | | | tee, crew neck, basic, cotton | YES |
| PROD-001 | PROD-001-WHT-M | Classic Black T-Shirt | T-Shirts | 100% cotton crew neck tee | 29.99 | USD | M | White | 0 | NO | | | tee, crew neck, basic, cotton | YES |
| PROD-002 | PROD-002-BLU-32 | Slim Fit Jeans | Jeans | Stretch denim slim fit | 59.99 | USD | 32 | Blue | 10 | YES | | | denim, slim, stretch, pants | YES |
| PROD-003 | PROD-003-GRY-L | Hoodie Pullover | Hoodies | Fleece-lined pullover hoodie | 49.99 | USD | L | Grey | 5 | YES | | | fleece, pullover, warm, sweater | YES |

### Tips

- **Product_ID** is the same for all variants of one product (e.g., all "Classic Black T-Shirt" rows share PROD-001)
- **Variant_SKU** is always unique — never repeat it
- Set **In_Stock** to NO when Stock_Qty reaches 0
- Set **Active** to NO to temporarily hide a product (seasonal items, discontinued, etc.)
- **Search_Keywords** should include words customers might type that are NOT already in the product name or category

---

## Tab 2: FAQ

### What is this tab for?

This stores answers to common questions. When a customer asks "What is your return policy?", the chatbot searches this tab and sends back the answer.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **FAQ_ID** | A unique code for each question | Identifies each FAQ entry |
| B | **Category** | Topic area | Lets the bot group related answers |
| C | **Question** | The question customers ask | Shown to customer as confirmation |
| D | **Keywords** | Words that help match this question (comma-separated) | The bot uses these to find the right answer |
| E | **Answer** | The full answer to send back | What the customer actually receives |
| F | **Active** | Is this FAQ live? (YES or NO) | Lets you disable outdated answers without deleting them |
| G | **Sort_Order** | A number to control display order (1, 2, 3...) | If multiple FAQs match, shows them in this order |
| H | **Last_Updated** | When this answer was last checked | Helps you keep answers fresh |

### Example Rows

| FAQ_ID | Category | Question | Keywords | Answer | Active | Sort_Order | Last_Updated |
|--------|----------|----------|----------|--------|--------|-----------|-------------|
| FAQ-001 | Shipping | How long does shipping take? | shipping, delivery, how long, days, arrive, when | Standard shipping takes 5-7 business days. Express shipping takes 2-3 business days. | YES | 1 | 2025-01-15 |
| FAQ-002 | Returns | What is your return policy? | return, refund, exchange, send back, money back | You can return unworn items within 30 days for a full refund. Items must have tags attached. | YES | 1 | 2025-01-15 |
| FAQ-003 | Sizing | How do I find my size? | size, sizing, fit, measurements, chart, what size | Check our size guide at [your link]. Measure your chest and waist, then compare to the chart. | YES | 1 | 2025-01-15 |
| FAQ-004 | Payment | What payment methods do you accept? | payment, pay, credit card, paypal, apple pay | We accept Visa, Mastercard, PayPal, and Apple Pay. | YES | 1 | 2025-01-15 |
| FAQ-005 | Shipping | Do you ship internationally? | international, worldwide, other countries, global | Yes! We ship to 50+ countries. International shipping takes 10-15 business days. | YES | 2 | 2025-01-15 |
| FAQ-006 | Discount | Do you have discount codes? | discount, coupon, promo, code, sale, deal, offer | Sign up for our newsletter to get 10% off your first order! Follow us on Instagram for flash sales. | YES | 1 | 2025-01-15 |
| FAQ-007 | Care | How do I wash my clothes? | wash, care, instructions, laundry, shrink | Machine wash cold, tumble dry low. Check the label inside your item for specific care instructions. | YES | 1 | 2025-01-15 |
| FAQ-008 | Shipping | How much does shipping cost? | shipping cost, delivery fee, free shipping, how much ship | Orders over $75 get FREE shipping! Standard shipping is $5.99, Express is $12.99. | YES | 3 | 2025-01-15 |

### Tips

- The **Keywords** column is the most important — add as many words/phrases as customers might use
- Set **Active** to NO for seasonal FAQs (e.g., holiday shipping deadlines after the holidays pass)
- **Sort_Order** helps when a customer's question matches multiple FAQs — lower numbers show first
- Update **Last_Updated** whenever you change an answer so you know which ones to review

---

## Tab 3: Orders

### What is this tab for?

This stores every order created through the chatbot. When a customer says "I want to buy the black t-shirt in size M", the chatbot creates a new row here with all the details.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **Order_ID** | A unique order number | Customer uses this to track their order |
| B | **Customer_ID** | Links to the Customers tab | Connects order to customer history |
| C | **Customer_Name** | Customer's full name | For shipping label and communication |
| D | **Customer_Email** | Customer's email | For order confirmation emails |
| E | **Customer_Phone** | Customer's phone (optional) | Backup contact method |
| F | **Product_ID** | Which product (from Products tab) | Links order to product catalog |
| G | **Variant_SKU** | Exact variant ordered | Identifies the precise size + color |
| H | **Product_Name** | Name of the product | Human-readable reference |
| I | **Size** | Selected size | What to pick and pack |
| J | **Color** | Selected color | What to pick and pack |
| K | **Quantity** | How many units | Affects total price and stock |
| L | **Unit_Price** | Price per item | For calculating totals |
| M | **Total_Price** | Unit_Price × Quantity | What the customer pays |
| N | **Currency** | Currency code | Matches product pricing |
| O | **Shipping_Address** | Delivery address | Where to send the package |
| P | **City** | City name | For shipping calculations |
| Q | **Country** | Country name or code | For international shipping decisions |
| R | **Payment_Method** | How they will pay | Tells you how to collect payment |
| S | **Payment_Status** | Has payment been received? | Prevents shipping unpaid orders |
| T | **Delivery_Method** | Standard, Express, etc. | Affects shipping cost and timeline |
| U | **Order_Status** | Current order status | Tracks where the order is in your process |
| V | **Channel** | Where the customer messaged from | Helps you know which platform works best |
| W | **Created_Date** | When the order was created | For record keeping |
| X | **Updated_Date** | Last time anything changed | Track modifications |
| Y | **Notes** | Any special requests | Gift wrapping, delivery instructions, etc. |

### Order Status Values

| Status | What It Means |
|--------|--------------|
| **Draft** | Order created by chatbot, not yet confirmed |
| **Confirmed** | Customer confirmed, ready to process |
| **Processing** | Being prepared for shipping |
| **Shipped** | On the way to customer |
| **Delivered** | Customer received it |
| **Cancelled** | Order was cancelled |
| **Refunded** | Payment returned to customer |

### Payment Status Values

| Status | What It Means |
|--------|--------------|
| **Pending** | Payment not yet received |
| **Paid** | Payment confirmed |
| **Failed** | Payment attempt failed |
| **Refunded** | Money returned to customer |

### Example Row

| Order_ID | Customer_ID | Customer_Name | Customer_Email | Customer_Phone | Product_ID | Variant_SKU | Product_Name | Size | Color | Quantity | Unit_Price | Total_Price | Currency | Shipping_Address | City | Country | Payment_Method | Payment_Status | Delivery_Method | Order_Status | Channel | Created_Date | Updated_Date | Notes |
|----------|-------------|---------------|----------------|----------------|-----------|-------------|--------------|------|-------|----------|-----------|-------------|----------|-----------------|------|---------|---------------|---------------|----------------|-------------|---------|-------------|-------------|-------|
| ORD-20250120-001 | CUST-001 | Sarah Johnson | sarah@example.com | +1-555-0123 | PROD-001 | PROD-001-BLK-M | Classic Black T-Shirt | M | Black | 2 | 29.99 | 59.98 | USD | 123 Main St | New York | US | PayPal | Pending | Standard | Draft | WhatsApp | 2025-01-20 14:30:00 | 2025-01-20 14:30:00 | Gift wrap please |

### Tips

- **Order_ID** format suggestion: ORD-YYYYMMDD-XXX (date + sequential number)
- All chatbot orders start as **Draft** status with **Pending** payment
- A human reviews draft orders before confirming and collecting payment
- **Channel** tracks where the conversation happened (WhatsApp, Messenger, Website, Instagram)
- **Updated_Date** should change every time any field is modified

---

## Tab 4: Tracking

### What is this tab for?

This stores shipping and tracking information. When a customer asks "Where is my order?", the chatbot searches this tab using their order number or email.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **Order_ID** | The order number (must match Orders tab) | Links tracking to the correct order |
| B | **Customer_Email** | Customer's email | Allows lookup by email (customers often forget order numbers) |
| C | **Tracking_Number** | Shipping carrier tracking number | Customer can track on carrier website |
| D | **Carrier** | Shipping company name | Tells customer which company has their package |
| E | **Shipping_Status** | Current delivery status | The main info the customer wants |
| F | **Shipped_Date** | When it was shipped | Helps calculate delivery expectations |
| G | **Estimated_Delivery** | Expected arrival date | Answers "when will it arrive?" |
| H | **Delivered_Date** | Actual delivery date (blank if not yet delivered) | Confirms receipt |
| I | **Tracking_URL** | Full link to track the package | Customer can click and see live updates |
| J | **Last_Updated** | When this row was last changed | Know if info is current |
| K | **Updated_By** | Who changed it (person name or "system") | Audit trail — know who updated shipping info |

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

### Example Row

| Order_ID | Customer_Email | Tracking_Number | Carrier | Shipping_Status | Shipped_Date | Estimated_Delivery | Delivered_Date | Tracking_URL | Last_Updated | Updated_By |
|----------|----------------|-----------------|---------|-----------------|-------------|-------------------|---------------|-------------|-------------|-----------|
| ORD-20250120-001 | sarah@example.com | 1Z999AA10123456784 | UPS | In Transit | 2025-01-21 | 2025-01-26 | | https://www.ups.com/track?tracknum=1Z999AA10123456784 | 2025-01-22 | Sarah (staff) |

### Tips

- You (or your shipping provider) update this tab — the chatbot only READS from it
- Customers can look up orders by **Order_ID** or **email address**
- **Updated_By** helps if multiple team members manage shipping
- Leave **Delivered_Date** blank until the package is actually delivered

---

## Tab 5: Customers

### What is this tab for?

This stores customer information so the chatbot can recognize returning customers, personalize responses, and track customer value over time.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **Customer_ID** | A unique customer code | Links to Orders and Support_Tickets tabs |
| B | **Chat_User_ID** | The ID from the chat platform (WhatsApp number, Messenger ID, etc.) | Lets the bot recognize returning customers automatically |
| C | **Customer_Name** | Full name | Personalize greetings ("Hi Sarah!") |
| D | **Email** | Email address | Used for lookups and communication |
| E | **Phone** | Phone number (optional) | Backup contact |
| F | **Preferred_Channel** | Which platform they use most | Know where to reach them |
| G | **First_Contact_Date** | When they first messaged | Customer lifetime tracking |
| H | **Last_Contact_Date** | When they last messaged | Identify inactive customers |
| I | **Last_Message_Date** | Exact date of most recent message | More precise than Last_Contact_Date for active conversation tracking |
| J | **Total_Orders** | How many orders they have placed | Identify VIP/loyal customers |
| K | **Total_Spent** | Total money spent across all orders | Customer value tracking |
| L | **Notes** | Any notes about this customer | "Prefers express shipping", "VIP", etc. |

### Example Row

| Customer_ID | Chat_User_ID | Customer_Name | Email | Phone | Preferred_Channel | First_Contact_Date | Last_Contact_Date | Last_Message_Date | Total_Orders | Total_Spent | Notes |
|-------------|-------------|---------------|-------|-------|-------------------|-------------------|------------------|------------------|-------------|-------------|-------|
| CUST-001 | wa_15550123456 | Sarah Johnson | sarah@example.com | +1-555-0123 | WhatsApp | 2025-01-15 | 2025-01-20 | 2025-01-20 14:35:00 | 3 | 149.97 | VIP customer, prefers express |

### Tips

- **Chat_User_ID** is how the bot recognizes someone without asking "who are you?" every time
- **Total_Orders** and **Total_Spent** update each time an order is confirmed
- Use **Notes** for any personal preferences the customer has shared
- The bot updates **Last_Contact_Date** and **Last_Message_Date** automatically each time someone messages

---

## Tab 6: Support_Tickets

### What is this tab for?

When the chatbot cannot help a customer (or the customer asks to speak to a human), it creates a support ticket here. Your team then checks this tab and follows up. This tab also captures conversation context so your team has full information.

### Columns

| Column | Name | What To Put Here | Why It Matters |
|--------|------|-----------------|----------------|
| A | **Ticket_ID** | A unique ticket number | Reference for customer and team |
| B | **Conversation_ID** | ID of the chat conversation | Links ticket to the full chat history |
| C | **Customer_ID** | Links to Customers tab | See customer's full history |
| D | **Customer_Name** | Who needs help | Quick identification |
| E | **Customer_Email** | Their email | For follow-up communication |
| F | **Customer_Phone** | Their phone (optional) | Alternative contact |
| G | **Channel** | Where the conversation happened | Know where to respond |
| H | **Issue_Category** | Type of problem | Helps route to right team member |
| I | **Issue_Description** | What the customer needs help with | Main description of the issue |
| J | **Last_User_Message** | The exact last message the customer sent | Context for the support agent |
| K | **Bot_Summary** | What the bot understood about the issue | Saves the agent from re-reading the full chat |
| L | **Handoff_Reason** | Why the bot handed off | Helps improve the bot over time |
| M | **Priority** | How urgent (Low, Medium, High) | Determines response time |
| N | **Status** | Current ticket status | Track progress |
| O | **Created_Date** | When the ticket was created | For response time tracking |
| P | **Assigned_To** | Team member handling it (blank initially) | Accountability |
| Q | **Resolved_Date** | When it was resolved (blank until done) | Measure resolution time |
| R | **Resolution_Notes** | How it was resolved (blank until done) | Knowledge base for future issues |

### Ticket Status Values

| Status | What It Means |
|--------|--------------|
| **Open** | New ticket, no one has looked at it yet |
| **In Progress** | A team member is working on it |
| **Waiting on Customer** | Waiting for the customer to reply |
| **Resolved** | Problem solved |
| **Closed** | Ticket completed and closed |

### Handoff Reason Values

| Reason | When It Happens |
|--------|----------------|
| **customer_requested** | Customer explicitly asked for a human |
| **bot_failed_3x** | Bot failed to understand 3 messages in a row |
| **angry_customer** | Frustration/anger detected |
| **complex_issue** | Issue too complex for bot (payment, legal, damage) |
| **policy_exception** | Customer needs something outside standard policy |

### Priority Rules

| Priority | When To Use | Response Time Goal |
|----------|-------------|-------------------|
| **High** | Customer is angry, wrong item shipped, payment issues, mentions legal | Within 2 hours |
| **Medium** | Exchange requests, general complaints, sizing issues | Within 4 hours |
| **Low** | General questions bot could not answer, prefers human, feature requests | Within 24 hours |

### Example Row

| Ticket_ID | Conversation_ID | Customer_ID | Customer_Name | Customer_Email | Customer_Phone | Channel | Issue_Category | Issue_Description | Last_User_Message | Bot_Summary | Handoff_Reason | Priority | Status | Created_Date | Assigned_To | Resolved_Date | Resolution_Notes |
|-----------|----------------|-------------|---------------|----------------|----------------|---------|---------------|-------------------|-------------------|-------------|---------------|----------|--------|-------------|-------------|--------------|-----------------|
| TKT-20250120-001 | conv_abc123 | CUST-001 | Sarah Johnson | sarah@example.com | +1-555-0123 | WhatsApp | Order Issue | Received wrong size, wants exchange | "I got a large but I ordered medium!" | Customer received size L instead of size M for order ORD-20250120-001 | customer_requested | Medium | Open | 2025-01-20 15:00:00 | | | |

### Tips

- Check this tab **at least twice per day**
- **Last_User_Message** gives your team instant context without needing to read the full conversation
- **Bot_Summary** is auto-generated by the chatbot — it summarizes what happened before handoff
- **Handoff_Reason** helps you improve the bot — if many tickets say "bot_failed_3x", you need better keyword matching
- Respond to **High** priority tickets within 2 hours

---

## Quick Setup Checklist

- [ ] Create a new Google Sheets spreadsheet
- [ ] Name it "Clothing Brand Chatbot Database"
- [ ] Create the **Products** tab with all 15 columns
- [ ] Create the **FAQ** tab with all 8 columns
- [ ] Create the **Orders** tab with all 25 columns
- [ ] Create the **Tracking** tab with all 11 columns
- [ ] Create the **Customers** tab with all 12 columns
- [ ] Create the **Support_Tickets** tab with all 18 columns
- [ ] Add at least 10-15 product variants to the Products tab
- [ ] Add at least 10 FAQ entries to the FAQ tab
- [ ] Add column headers in **Row 1** of every tab (this is critical for n8n!)
- [ ] Freeze Row 1 in each tab (View > Freeze > 1 row) so headers stay visible

---

## Important Rules

1. **Row 1 must ALWAYS be your column headers** — n8n reads column names from row 1
2. **Never leave blank rows** between data rows — this can confuse lookups
3. **Be consistent** with spelling (e.g., always "YES" not sometimes "Yes" or "yes")
4. **Use the exact column names** shown above — the n8n workflow will reference these names
5. **Back up your spreadsheet** regularly (File > Make a copy)
6. **Freeze Row 1** in every tab so you can always see your headers while scrolling
7. **Use data validation** where possible (e.g., dropdown for In_Stock: YES/NO) to prevent typos

---

## Relationships Between Tabs (How They Connect)

Understanding how the tabs link together:

```
Products tab                    
  |                             
  | (Product_ID + Variant_SKU)  
  v                             
Orders tab  <--- Customer_ID --- Customers tab
  |                                    |
  | (Order_ID)                         | (Customer_ID)
  v                                    v
Tracking tab                    Support_Tickets tab
```

- **Products → Orders**: When a customer orders, the Product_ID and Variant_SKU from Products are saved in Orders
- **Customers → Orders**: Customer_ID links a customer to all their orders
- **Orders → Tracking**: Order_ID links an order to its shipping info
- **Customers → Support_Tickets**: Customer_ID links a customer to their support history

This means you can look up:
- All orders for a specific customer
- Tracking info for any order
- All support tickets for a customer
- Which product variant was ordered
