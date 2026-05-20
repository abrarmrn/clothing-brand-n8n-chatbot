# Build Tasks — Step by Step

## Overview

This is your **step-by-step checklist** to build the entire chatbot system. Follow the tasks in order. Each task builds on the previous one.

**Estimated total time:** 4-6 hours (for a complete beginner)

**Rule:** Complete each task fully before moving to the next one. Test as you go!

---

## Phase 1: Set Up Your Tools

*Time estimate: 30-45 minutes*

### Task 1.1: Set Up n8n

| Detail | Info |
|--------|------|
| **What to do** | Install n8n or create an n8n Cloud account |
| **Why** | n8n is where you will build the entire chatbot workflow |
| **How** | Option A: Go to [n8n.io](https://n8n.io) and sign up for n8n Cloud (easiest). Option B: Self-host using Docker (more advanced). |
| **Done when** | You can open n8n and see the workflow editor (empty canvas with a "+" button) |

### Task 1.2: Create Your Google Sheets Database

| Detail | Info |
|--------|------|
| **What to do** | Create the Google Sheets spreadsheet with all 6 tabs |
| **Why** | This is your chatbot's database |
| **How** | Follow the instructions in `google-sheets-structure.md` |
| **Done when** | You have a spreadsheet with 6 tabs, each with the correct column headers in Row 1 |

**Tabs to create:**
1. **Products** — 15 columns (Product_ID through Active)
2. **FAQ** — 8 columns (FAQ_ID through Last_Updated)
3. **Orders** — 25 columns (Order_ID through Notes)
4. **Tracking** — 11 columns (Order_ID through Updated_By)
5. **Customers** — 12 columns (Customer_ID through Notes)
6. **Support_Tickets** — 18 columns (Ticket_ID through Resolution_Notes)

### Task 1.3: Add Sample Product Variants

| Detail | Info |
|--------|------|
| **What to do** | Add test product variants to the Products tab |
| **Why** | You need data to test your chatbot's product search |
| **How** | Create at least 5 products with multiple variants each (different size + color combos) |
| **Done when** | Products tab has 15-20 rows of variant data |

**Remember:** Each row = one variant (one specific size + color combination). Use the examples from `google-sheets-structure.md`.

**Sample products to add (with variants):**

| Product | Variants to create |
|---------|-------------------|
| Classic Black T-Shirt (PROD-001) | 3 sizes × 3 colors = 9 rows |
| Slim Fit Jeans (PROD-002) | 4 sizes × 2 colors = 8 rows |
| Hoodie Pullover (PROD-003) | 3 sizes × 3 colors = 9 rows |
| Summer Dress (PROD-004) | 3 sizes × 2 colors = 6 rows |
| Baseball Cap (PROD-005) | 1 size × 4 colors = 4 rows |

**For each variant row, fill in:**
- Product_ID (same for all variants of one product)
- Variant_SKU (unique! format: PROD-001-BLK-M)
- Product_Name, Category, Description, Price, Currency
- Size, Color (this is what makes each variant unique)
- Stock_Qty (a number — some should be 0 for testing)
- In_Stock (YES if Stock_Qty > 0, NO if Stock_Qty = 0)
- Search_Keywords (extra words customers might use)
- Active (YES for most, set 1-2 to NO for testing)

### Task 1.4: Add Sample FAQ Entries

| Detail | Info |
|--------|------|
| **What to do** | Add at least 10 FAQ entries to the FAQ tab |
| **Why** | The chatbot needs answers to search through |
| **How** | Use the examples from `google-sheets-structure.md` and add your own |
| **Done when** | FAQ tab has 10+ rows with Keywords filled in |

**For each FAQ row, fill in:**
- FAQ_ID, Category, Question, Keywords, Answer
- Active = "YES" (set 1-2 to "NO" for testing)
- Sort_Order = a number (1, 2, 3...) within each category
- Last_Updated = today's date

### Task 1.5: Connect Google Sheets to n8n

| Detail | Info |
|--------|------|
| **What to do** | Set up the Google Sheets credential in n8n |
| **Why** | n8n needs permission to read/write your spreadsheet |
| **How** | In n8n: go to Settings > Credentials > Add New > Google Sheets OAuth2. Follow the sign-in prompts. |
| **Done when** | You can add a Google Sheets node and see your spreadsheet listed |

**Troubleshooting:**
- If you see "Access Denied": make sure you authorized the correct Google account
- If spreadsheet doesn't appear: check that you shared it with the correct account
- If connection fails: try removing the credential and adding it again

---


## Phase 2: Build the Core Workflow Structure

*Time estimate: 45-60 minutes*

### Task 2.1: Create a New Workflow

| Detail | Info |
|--------|------|
| **What to do** | Create a new blank workflow in n8n |
| **Why** | This will be your main chatbot workflow |
| **How** | In n8n: Click "Add Workflow" (or the "+" icon). Name it "Clothing Brand Chatbot" |
| **Done when** | You have an empty workflow canvas with the name set |

### Task 2.2: Add the Chat Trigger Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Chat Trigger node as the first node |
| **Why** | This starts the workflow when a message arrives |
| **How** | Click "+" on the canvas > search "Chat Trigger" > add it |
| **Done when** | You have a Chat Trigger node on the canvas. You can click "Chat" at the bottom of the n8n editor to open a test chat window |

### Task 2.3: Add the Customer Identification Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets Search node that checks the Customers tab |
| **Why** | Recognizes returning customers by their Chat_User_ID |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: Customers > Search Column: Chat_User_ID > Value: sender ID from trigger |
| **Done when** | The node returns customer data if found, or empty results for new visitors |

**What this gives you:**
- If found: Customer_ID, Customer_Name, Email, Total_Orders, Total_Spent
- Use Customer_Name for personalized greetings
- Use Customer_ID to link orders and tickets later

### Task 2.4: Add the Message Classifier

| Detail | Info |
|--------|------|
| **What to do** | Add a node that classifies incoming messages into categories |
| **Why** | The chatbot needs to decide which branch to use |
| **How** | **Option A (AI):** Add an AI Agent or OpenAI node with the classification prompt from `chatbot-branching-logic.md`. **Option B (No AI):** Add a Switch node with keyword conditions (also in `chatbot-branching-logic.md`). |
| **Done when** | The node outputs one of: greeting, product_inquiry, faq, order_create, order_track, human_support, unknown |

**Testing checkpoint:** Send "Hi" through the test chat → should output "greeting". Send "Do you have t-shirts?" → should output "product_inquiry". Send "Track ORD-123" → should output "order_track".

### Task 2.5: Add the Router/Switch Node

| Detail | Info |
|--------|------|
| **What to do** | Route messages to different branches based on the classification |
| **Why** | Each category needs to go to a different set of nodes |
| **How** | Connect classifier output to a Switch node. Create 7 outputs (one per category). |
| **Done when** | Messages are correctly routed to 7 different outputs |

---

## Phase 3: Build Branch 1 — Direct Reply

*Time estimate: 15 minutes*

### Task 3.1: Create the Direct Reply Response

| Detail | Info |
|--------|------|
| **What to do** | Add a "Respond to Chat" node for greetings |
| **Why** | Simple messages need quick, friendly replies |
| **How** | Add "Respond to Chat" node. Connect to "greeting" output. Set response message. If customer was found in Task 2.3, use their Customer_Name for personalization. |
| **Done when** | Sending "Hi" returns a welcome message |

**Test messages:**
- "Hi" → Welcome message (personalized if Customer_Name is available)
- "Thanks" → "You're welcome" message
- "Bye" → Goodbye message

---

## Phase 4: Build Branch 2 — Product Lookup

*Time estimate: 30-45 minutes*

### Task 4.1: Add Product Search Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets node that searches the Products tab |
| **Why** | When customers ask about products, the bot needs to find matching variants |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: Products > Add filters: Active = "YES" AND search keywords against Product_Name, Category, Color, Size, Search_Keywords |
| **Done when** | Searching for "t-shirt" returns all t-shirt variants where Active = "YES" |

**Important:** Always filter by Active = "YES" so hidden products never show up.

### Task 4.2: Add Result Grouping Logic

| Detail | Info |
|--------|------|
| **What to do** | Group variant results by Product_ID so each product shows once |
| **Why** | Without grouping, searching "t-shirts" might return 12 rows (all variants) instead of showing 1 product with its options |
| **How** | Add a Code node or Set node that groups rows by Product_ID, then collects unique sizes and colors for each product |
| **Done when** | A search returns grouped results like: "Classic Black T-Shirt — Sizes: S, M, L | Colors: Black, White" |

**Grouping logic (what to extract per Product_ID):**
- Product_Name (same for all variants)
- Price (same for all variants)
- Available sizes (from in-stock variants only, where In_Stock = "YES")
- Available colors (from in-stock variants only)
- Stock note (e.g., "Low stock!" if any variant has Stock_Qty < 3)

### Task 4.3: Add "No Results" Check

| Detail | Info |
|--------|------|
| **What to do** | Add an IF node to handle empty search results |
| **Why** | The bot needs different responses for "found products" vs "no matches" |
| **How** | Add IF node > Condition: check if results array is not empty |
| **Done when** | Empty searches go to "no results" message, found products go to formatting |

### Task 4.4: Format and Send Product Reply

| Detail | Info |
|--------|------|
| **What to do** | Format grouped product data into a nice message |
| **Why** | Raw data is ugly — make it customer-friendly |
| **How** | Add Set node to build the message (product name, price, sizes, colors), then Respond to Chat node |
| **Done when** | "Show me t-shirts" returns a formatted, grouped product list |

**Test messages:**
- "Show me t-shirts" → Grouped product list with sizes/colors
- "Do you have hoodies in large?" → Filtered results for Hoodies, Size L
- "Show me purple sandals" → "Not found" message with category suggestions
- Ask for an out-of-stock variant → "Out of stock" message with alternatives

---


## Phase 5: Build Branch 3 — FAQ Answers

*Time estimate: 25 minutes*

### Task 5.1: Add FAQ Search Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets node that searches the FAQ tab |
| **Why** | Common questions need automatic answers |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: FAQ > Filter: Active = "YES" > Search the Keywords column for matches |
| **Done when** | Searching for "shipping" returns your shipping FAQ entries |

**Important:** Only return FAQs where Active = "YES". Use Sort_Order to prioritize when multiple match.

### Task 5.2: Add Keyword Matching Logic

| Detail | Info |
|--------|------|
| **What to do** | Score FAQ entries by how many keywords match the customer's message |
| **Why** | "How long does shipping take?" should match the shipping FAQ, not the returns FAQ |
| **How** | Add a Code node that: (1) splits customer message into words, (2) compares against each FAQ's Keywords column, (3) returns the FAQ with the most matches. If tied, use lowest Sort_Order. |
| **Done when** | "Return policy" matches the returns FAQ, not the shipping FAQ |

### Task 5.3: Add "No Answer" Check and Reply

| Detail | Info |
|--------|------|
| **What to do** | Handle found and not-found scenarios |
| **Why** | If no FAQ matches, offer alternatives |
| **How** | IF node: if match found → send answer. If not → send "I don't have an answer for that" with options. |
| **Done when** | Known questions get answers. Unknown questions get a helpful fallback. |

**Test messages:**
- "What's your return policy?" → Returns the return policy answer
- "How long does shipping take?" → Returns shipping FAQ
- "Do you accept Bitcoin?" → "No answer found" with suggestions

---

## Phase 6: Build Branch 4 — Order Creation

*Time estimate: 60-90 minutes*

### Task 6.1: Set Up Order Conversation Flow

| Detail | Info |
|--------|------|
| **What to do** | Build the multi-step order collection process |
| **Why** | Orders need: product, size, color, quantity, name, email, address, delivery, payment |
| **How** | **Recommended:** Use an AI Agent node with conversation memory. It can naturally ask follow-up questions. **Alternative:** Build a sub-workflow with multiple steps. |
| **Done when** | The bot can ask for each piece of info step by step |

**The bot must collect these fields (mapping to Orders tab columns):**

| Info to Ask | Maps to Column | Required? |
|------------|----------------|-----------|
| Which product | Product_ID, Product_Name | Yes |
| Size | Size | Yes |
| Color | Color | Yes |
| How many | Quantity | Yes (default: 1) |
| Customer name | Customer_Name | Yes |
| Email | Customer_Email | Yes |
| Phone | Customer_Phone | Optional |
| Street address | Shipping_Address | Yes |
| City | City | Yes |
| Country | Country | Yes |
| Delivery speed | Delivery_Method | Yes (default: Standard) |
| Payment method | Payment_Method | Yes |
| Special requests | Notes | Optional |

### Task 6.2: Add Variant Validation Node

| Detail | Info |
|--------|------|
| **What to do** | Verify the selected variant exists and has enough stock |
| **Why** | Can't sell products that don't exist or are out of stock |
| **How** | Add Google Sheets Search node > Sheet: Products > Search for matching Product_ID + Size + Color where Active = "YES" |
| **Done when** | Bot confirms variant exists and Stock_Qty >= Quantity before proceeding |

**Validation checks (in order):**
1. Does the Product_ID exist with Active = "YES"? → If no: "Product not found"
2. Does the Size + Color combination exist? → If no: "That variant isn't available. Options: [list]"
3. Is In_Stock = "YES"? → If no: "Out of stock. Alternatives: [list]"
4. Is Stock_Qty >= requested Quantity? → If no: "Only [X] left. Want [X] instead?"

**On success:** Retrieve the **Variant_SKU** and **Price** (Unit_Price) from the matched row.

### Task 6.3: Add Order Save Node

| Detail | Info |
|--------|------|
| **What to do** | Save the completed order to the Orders tab (all 25 columns) |
| **Why** | Once all info is collected and validated, the order must be stored |
| **How** | Add Google Sheets node > Operation: Append Row > Sheet: Orders |
| **Done when** | Completing an order creates a new row in Orders with all fields filled |

**Auto-generated fields (the bot creates these, not the customer):**
- **Order_ID**: Generate as ORD-YYYYMMDD-XXX
- **Customer_ID**: From Customers tab (or generate new CUST-XXX for new customers)
- **Variant_SKU**: From validation step
- **Unit_Price**: From Products tab
- **Total_Price**: Unit_Price × Quantity
- **Currency**: From Products tab
- **Payment_Status**: "Pending"
- **Order_Status**: "Draft"
- **Channel**: From chat trigger (WhatsApp, Website, etc.)
- **Created_Date**: Current timestamp
- **Updated_Date**: Current timestamp

### Task 6.4: Update Customer Record

| Detail | Info |
|--------|------|
| **What to do** | Create or update the customer in the Customers tab |
| **Why** | Track customer history, recognize them next time |
| **How** | If existing customer (Customer_ID found): Update Total_Orders (+1), Last_Contact_Date, Last_Message_Date. If new customer: Append new row to Customers tab. |
| **Done when** | After order, Customers tab reflects the new/updated customer |

**For new customers, fill in:**
- Customer_ID: Generate CUST-XXX
- Chat_User_ID: From chat trigger
- Customer_Name, Email, Phone: From order collection
- Preferred_Channel: From chat trigger
- First_Contact_Date: Now
- Last_Contact_Date: Now
- Last_Message_Date: Now
- Total_Orders: 1
- Total_Spent: This order's Total_Price
- Notes: (blank)

### Task 6.5: Add Order Confirmation Reply

| Detail | Info |
|--------|------|
| **What to do** | Send a complete order summary to the customer |
| **Why** | Customers need confirmation with all details |
| **How** | Add Respond to Chat node with the order summary template |
| **Done when** | After order saves, customer sees: Order_ID, product details, Variant_SKU, price breakdown, shipping info, and next steps |

**Test the full flow:**
1. Say "I want to order the black t-shirt in size M"
2. Provide quantity, name, email, address, delivery, payment when asked
3. Confirm the order
4. Check Google Sheets: new row in Orders tab with all 25 columns filled
5. Check Customers tab: customer record created/updated

---


## Phase 7: Build Branch 5 — Order Tracking

*Time estimate: 30 minutes*

### Task 7.1: Extract Order Number or Use Customer Email

| Detail | Info |
|--------|------|
| **What to do** | Detect if the message contains an order number, or use the known customer's email |
| **Why** | If they already gave the order number, don't ask again. If they're a known customer, look up by email. |
| **How** | Use a Set/Code node to: (1) check for "ORD-" pattern in message, (2) if not found, check if customer was identified in Task 2.3, (3) if neither, ask for order number or email |
| **Done when** | Bot correctly detects order numbers, uses known customer email, or asks politely |

### Task 7.2: Add Tracking Lookup Node

| Detail | Info |
|--------|------|
| **What to do** | Search the Tracking tab for the order |
| **Why** | Customers want shipping status, carrier, and delivery estimate |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: Tracking > Search by Order_ID or Customer_Email |
| **Done when** | Searching with a valid Order_ID returns: Shipping_Status, Carrier, Tracking_Number, Estimated_Delivery, Tracking_URL, Last_Updated, Updated_By |

### Task 7.3: Add Orders Tab Fallback

| Detail | Info |
|--------|------|
| **What to do** | If not in Tracking tab, check the Orders tab for order status |
| **Why** | The order might exist but hasn't shipped yet (still Draft/Confirmed/Processing) |
| **How** | Add another Google Sheets Search node > Sheet: Orders > Search by Order_ID. Return Order_Status and Payment_Status. |
| **Done when** | Orders that haven't shipped show their current status instead of "not found" |

### Task 7.4: Format and Send Tracking Reply

| Detail | Info |
|--------|------|
| **What to do** | Format tracking data into a customer-friendly message |
| **Why** | Show status, carrier, dates, and tracking link clearly |
| **How** | Add IF nodes for: (1) tracking found → full tracking reply, (2) order found but not shipped → status reply, (3) nothing found → error reply |
| **Done when** | All three scenarios respond correctly |

**Test messages:**
- Add a sample row to Tracking tab, then ask "Where is order ORD-20250120-001?" → Full tracking details
- Ask about an order that exists in Orders but NOT in Tracking → "Not shipped yet" status
- Ask about a non-existent order → "Not found" with helpful suggestions
- If known customer has multiple orders → List all their orders and ask which one

---

## Phase 8: Build Branch 6 — Human Support Handoff

*Time estimate: 30-40 minutes*

### Task 8.1: Add Issue Collection Logic

| Detail | Info |
|--------|------|
| **What to do** | Ask the customer what they need help with (unless they're frustrated — then skip) |
| **Why** | Gives the support team context without making angry customers repeat themselves |
| **How** | IF node: if Handoff_Reason is "angry_customer" or "bot_failed_3x" → skip to ticket creation. Otherwise → ask "Could you briefly tell me what you need help with?" |
| **Done when** | Calm customers are asked for details. Frustrated customers get immediate handoff. |

### Task 8.2: Add Priority Assignment Logic

| Detail | Info |
|--------|------|
| **What to do** | Automatically assign priority (High/Medium/Low) based on the issue |
| **Why** | Urgent issues need faster response |
| **How** | Add IF/Switch node. Check for: angry language/ALL CAPS/damage/payment → High. Exchange/complaint/sizing → Medium. Everything else → Low. |
| **Done when** | Different issues get appropriate priority levels |

### Task 8.3: Add Support Ticket Creation Node

| Detail | Info |
|--------|------|
| **What to do** | Append a row to the Support_Tickets tab with all 18 columns |
| **Why** | Your team needs this ticket to follow up |
| **How** | Add Google Sheets node > Operation: Append Row > Sheet: Support_Tickets |
| **Done when** | Triggering handoff creates a complete ticket row |

**Fields to fill (all 18 columns):**

| Column | Source |
|--------|--------|
| Ticket_ID | Generate: TKT-YYYYMMDD-XXX |
| Conversation_ID | From chat trigger session ID |
| Customer_ID | From Task 2.3 (or blank if unknown) |
| Customer_Name | From Customers tab or collected during handoff |
| Customer_Email | From Customers tab or collected during handoff |
| Customer_Phone | From Customers tab (or blank) |
| Channel | From chat trigger |
| Issue_Category | Detected from context |
| Issue_Description | Summary of the problem |
| Last_User_Message | Exact last message (copy verbatim) |
| Bot_Summary | Auto-generated conversation summary |
| Handoff_Reason | customer_requested / bot_failed_3x / angry_customer / complex_issue / policy_exception |
| Priority | From Task 8.2 |
| Status | "Open" |
| Created_Date | Current timestamp |
| Assigned_To | (blank) |
| Resolved_Date | (blank) |
| Resolution_Notes | (blank) |

### Task 8.4: Add Handoff Reply

| Detail | Info |
|--------|------|
| **What to do** | Tell the customer their ticket was created with expected response time |
| **Why** | Customer needs to know what happens next |
| **How** | Add Respond to Chat node with ticket number, priority, and response time based on priority level |
| **Done when** | Customer gets confirmation with ticket number and time expectation |

**Response times by priority:**
- High → "Within 2 hours"
- Medium → "Within 4 hours"
- Low → "Within 24 hours"

**Test messages:**
- "I want to speak to a human" → Ticket created (Low priority, Handoff_Reason: customer_requested)
- "THIS IS TERRIBLE! WRONG ORDER!" → Ticket created (High priority, Handoff_Reason: angry_customer)
- "I need help with an exchange" → Ticket created (Medium priority, Handoff_Reason: customer_requested)

---


## Phase 9: Build Fallback and Error Handling

*Time estimate: 20-30 minutes*

### Task 9.1: Add Fallback Reply with Counter

| Detail | Info |
|--------|------|
| **What to do** | Add a response for unrecognized messages with an escalation counter |
| **Why** | The bot should never leave a customer without a response, and should auto-escalate after 3 failures |
| **How** | Connect the "unknown/default" output to a Respond to Chat node with the menu. Track consecutive failures. After 3 → auto-trigger Branch 6. |
| **Done when** | 1st unknown → menu. 2nd → menu + human option hint. 3rd → auto-handoff with Handoff_Reason = "bot_failed_3x" |

### Task 9.2: Add Error Handling for Google Sheets Failures

| Detail | Info |
|--------|------|
| **What to do** | Add error outputs so the bot doesn't crash if Google Sheets is unreachable |
| **Why** | Sometimes connections fail — the bot should handle this gracefully |
| **How** | On each Google Sheets node, add an Error output connection to a "sorry" message |
| **Done when** | If Google Sheets times out, the customer gets: "I'm having trouble looking that up. Please try again in a moment, or I can connect you with our team." |

### Task 9.3: Add Variant Validation Error Messages

| Detail | Info |
|--------|------|
| **What to do** | Handle all product/variant validation failures gracefully |
| **Why** | Customers need clear messages when their selection isn't available |
| **How** | After the Validate Variant node (Task 6.2), add IF nodes for each failure type |
| **Done when** | Each validation failure produces a specific, helpful error message |

**Error messages:**
- Product not found → "I can't find that product. Want to see what we have?"
- Variant not available → "That size/color isn't available. Options: [list from same Product_ID]"
- Out of stock → "That's out of stock. Alternatives: [list in-stock variants]"
- Not enough stock → "We only have [Stock_Qty] left. Would you like [Stock_Qty]?"

---

## Phase 10: Testing

*Time estimate: 30-45 minutes*

### Task 10.1: Test Each Branch Individually

| # | Test | Message to Send | Expected Result |
|---|------|----------------|-----------------|
| 1 | Greeting (new customer) | "Hello!" | Generic welcome message |
| 2 | Greeting (known customer) | "Hi" (from known Chat_User_ID) | Personalized: "Hi [Name]!" |
| 3 | Product found | "Show me t-shirts" | Grouped product list with sizes/colors |
| 4 | Specific variant | "Do you have hoodie in large grey?" | Single variant with Stock_Qty |
| 5 | Product not found | "Show me bikinis" | Not found + category suggestions |
| 6 | Out of stock variant | Ask for variant with Stock_Qty = 0 | Out of stock + alternatives |
| 7 | Inactive product | Ask for product with Active = "NO" | Not found (hidden) |
| 8 | FAQ found | "Return policy?" | Correct answer |
| 9 | FAQ not found | "Do you do custom embroidery?" | No answer + options |
| 10 | Inactive FAQ | FAQ with Active = "NO" | Should NOT be returned |
| 11 | Order start | "I want to buy a hoodie" | Starts collection flow |
| 12 | Order complete | Provide all details | All 25 columns saved in Orders tab |
| 13 | Order — variant invalid | Order a non-existent size | Helpful error with alternatives |
| 14 | Order — out of stock | Order a variant with Stock_Qty = 0 | Out of stock + alternatives |
| 15 | Tracking found | "Track order ORD-20250120-001" | Full tracking details |
| 16 | Not shipped yet | Track an order in Orders but not Tracking | "Not shipped yet" with Order_Status |
| 17 | Tracking not found | "Track order ORD-99999" | Not found message |
| 18 | Human handoff (calm) | "Talk to a real person" | Low priority ticket + confirmation |
| 19 | Human handoff (angry) | "THIS IS USELESS!" | High priority ticket + immediate confirmation |
| 20 | Fallback x1 | "asdfjkl" | Menu of options |
| 21 | Fallback x3 | Three gibberish messages | Auto-handoff with bot_failed_3x |
| 22 | Error | Disconnect Google Sheets | Graceful error message |

### Task 10.2: Test Edge Cases

| # | Test | Message to Send | Expected Result |
|---|------|----------------|-----------------|
| 1 | Multi-intent | "Hi, I want to track my order" | Goes to tracking (not greeting) |
| 2 | Cancel mid-order | "Never mind" during order flow | Order cancelled gracefully |
| 3 | Empty message | "" | Fallback menu |
| 4 | Very long message | Paragraph of text | Bot identifies main intent |
| 5 | Emoji only | Thumbs up | Appropriate acknowledgment |
| 6 | Photo sent | (image) | "I can't analyze images" message |
| 7 | Returning customer orders | Known customer starts order | Pre-fills name/email from Customers tab |
| 8 | Multiple orders (tracking) | Known customer asks "my orders?" | Lists all their orders |

### Task 10.3: Verify Google Sheets Data Integrity

| Check | How to Verify |
|-------|---------------|
| Orders have all 25 columns filled | Open Orders tab after test orders |
| Variant_SKU matches correctly | Cross-reference with Products tab |
| Total_Price = Unit_Price × Quantity | Check math on each test order |
| Customer_ID links correctly | Same Customer_ID in Orders and Customers tabs |
| Support tickets have all 18 columns | Open Support_Tickets tab after handoff tests |
| Bot_Summary and Last_User_Message are filled | Check Support_Tickets entries |
| No blank rows between data | Scroll through each tab |
| Active = "NO" products never appear | Search for inactive products — should get "not found" |

---


## Phase 11: Connect to a Real Channel (Optional — Do Later)

*Time estimate: 1-2 hours per channel*

### Task 11.1: Choose Your First Channel

| Channel | Difficulty | Best For |
|---------|-----------|----------|
| Website chat widget | Easiest | If you have a website |
| WhatsApp Business | Medium | Direct customer messaging |
| Facebook Messenger | Medium | If you have a Facebook page |
| Instagram DMs | Medium | If you have an Instagram business account |

### Task 11.2: Replace Chat Trigger with Channel Trigger

| Detail | Info |
|--------|------|
| **What to do** | Replace the test Chat Trigger with a real channel trigger |
| **Why** | Real customers will message from WhatsApp/website/etc. |
| **How** | Remove the Chat Trigger and add the appropriate channel node |
| **Done when** | A real message from the chosen channel triggers your workflow |

**Important:** Make sure the new trigger provides:
- Message text (the customer's message)
- Sender ID (maps to Chat_User_ID in Customers tab)
- Channel name (fills the Channel column in Orders and Support_Tickets)

### Task 11.3: Test with Real Messages

| Detail | Info |
|--------|------|
| **What to do** | Have a friend or second account send test messages |
| **Why** | Real-world messages may differ from your test messages |
| **How** | Send 10+ real messages from the connected channel |
| **Done when** | All message types get correct responses through the real channel |

---

## Phase 12: Go-Live Checklist

Before letting real customers use your chatbot, verify everything:

### Functionality
- [ ] All 6 branches respond correctly
- [ ] Error messages are friendly and helpful
- [ ] Google Sheets connection is stable (test multiple times)
- [ ] Fallback escalation works (3 failures → auto-handoff)
- [ ] Returning customers are recognized and personalized

### Data Integrity
- [ ] Orders tab fills all 25 columns correctly
- [ ] Support_Tickets tab fills all 18 columns correctly
- [ ] Customers tab updates Total_Orders and Total_Spent
- [ ] Variant_SKU is always correct in orders
- [ ] Active = "NO" products never appear in search results

### Operations
- [ ] You have a plan to check Support_Tickets tab at least twice daily
- [ ] You have a plan to review Draft orders daily
- [ ] You know how to update stock (Stock_Qty and In_Stock) in Products tab
- [ ] You have a plan to update Tracking tab when orders ship
- [ ] You have told your team about the chatbot
- [ ] You have a backup plan if the chatbot goes down

### Content
- [ ] All Products have Search_Keywords filled in
- [ ] All FAQ entries have comprehensive Keywords
- [ ] FAQ answers are up-to-date (check Last_Updated dates)
- [ ] Bot messages have your brand name (not "[Brand Name]" placeholder)

---

## Quick Reference: Task Summary

| Phase | Tasks | Time Estimate |
|-------|-------|---------------|
| 1. Setup (tools + data) | 5 tasks | 30-45 min |
| 2. Core Workflow | 5 tasks | 45-60 min |
| 3. Direct Reply | 1 task | 15 min |
| 4. Product Lookup | 4 tasks | 30-45 min |
| 5. FAQ Answers | 3 tasks | 25 min |
| 6. Order Creation | 5 tasks | 60-90 min |
| 7. Order Tracking | 4 tasks | 30 min |
| 8. Human Handoff | 4 tasks | 30-40 min |
| 9. Error Handling | 3 tasks | 20-30 min |
| 10. Testing | 3 tasks | 30-45 min |
| 11. Channel Connect | 3 tasks | 1-2 hours |
| 12. Go Live | Checklist | 15 min |
| **Total** | **40 tasks** | **4-6 hours** |

---

## Tips for Success

1. **Build one branch at a time.** Don't try to build everything at once.
2. **Test constantly.** After every node you add, send a test message.
3. **Start with the easiest branches** (Direct Reply → FAQ → Products → Tracking → Orders → Handoff).
4. **Save your workflow often.** n8n autosaves, but click save manually too.
5. **If something breaks, undo.** n8n has Ctrl+Z.
6. **Keep your Google Sheets clean.** No blank rows, consistent YES/NO values.
7. **Fill in Search_Keywords generously.** The more keywords, the better product search works.
8. **Test with variant-level queries.** "Do you have the hoodie in large grey?" tests the full variant system.
9. **Check the data after every order test.** Make sure all 25 columns are filled correctly.
10. **Ask for help.** The n8n community forum is very helpful for beginners.

---

## What To Do After Go-Live

After your chatbot is live:

1. **Daily:** Check Support_Tickets tab for open tickets (respond within priority timeframes)
2. **Daily:** Review Draft orders and confirm/process them
3. **Weekly:** Check which FAQ questions the bot couldn't answer (improve Keywords)
4. **Weekly:** Review Handoff_Reason values — if "bot_failed_3x" is common, improve keywords
5. **Weekly:** Update Stock_Qty for products that sold
6. **Monthly:** Add new FAQ entries based on common questions
7. **Monthly:** Add new products/variants to the Products tab
8. **Monthly:** Review Total_Spent in Customers tab to identify VIPs

**Congratulations!** You will have built a fully functional, variant-aware chatbot for your clothing brand!
