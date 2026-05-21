# Build Tasks - Step by Step

## Overview

This is your **step-by-step checklist** to build the entire chatbot system. Follow the tasks in order. Each task builds on the previous one.

**Estimated total time:** 3-5 hours (for a complete beginner)

> **Fast-Track Option (v4):** If you want a working Messenger chatbot immediately, import `n8n-messenger-hybrid-ai-v4.json` and follow `messenger-hybrid-v4-import-instructions.md`. You can be live in ~30 minutes. The tasks below are the manual learning path.

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

### Task 1.3: Add Sample Data to Google Sheets

| Detail | Info |
|--------|------|
| **What to do** | Add test data to Products and FAQ tabs |
| **Why** | You need data to test your chatbot against |
| **How** | Add at least 5 products and 10 FAQ entries using the examples from `google-sheets-structure.md` |
| **Done when** | Products tab has 5+ rows of data, FAQ tab has 10+ rows of data |

**Sample products to add:**
1. Classic Black T-Shirt - $29.99
2. Slim Fit Jeans - $59.99
3. Hoodie Pullover - $49.99
4. Summer Dress - $44.99
5. Baseball Cap - $19.99

**Sample FAQ topics to add:**
1. Shipping time
2. International shipping
3. Return policy
4. Size guide
5. Payment methods
6. Discount codes
7. Business hours
8. Contact info
9. Gift wrapping
10. Care instructions

### Task 1.4: Connect Google Sheets to n8n

| Detail | Info |
|--------|------|
| **What to do** | Set up the Google Sheets credential in n8n |
| **Why** | n8n needs permission to read/write your spreadsheet |
| **How** | In n8n: go to Settings > Credentials > Add New > Google Sheets OAuth2. Follow the sign-in prompts to authorize your Google account. |
| **Done when** | You can add a Google Sheets node and see your spreadsheet listed |

**Troubleshooting:**
- If you see "Access Denied": make sure you authorized the correct Google account
- If spreadsheet doesn't appear: check that you shared it with the correct account
- If connection fails: try removing the credential and adding it again

---

## Phase 2: Build the Core Workflow

*Time estimate: 45-60 minutes*

### Task 2.1: Create a New Workflow

| Detail | Info |
|--------|------|
| **What to do** | Create a new blank workflow in n8n |
| **Why** | This will be your main chatbot workflow |
| **How** | In n8n: Click "Add Workflow" (or the "+" icon). Name it "Clothing Brand Chatbot" |
| **Done when** | You have an empty workflow canvas open with the name set |

### Task 2.2: Add the Chat Trigger Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Chat Trigger node as the first node |
| **Why** | This is the entry point - it starts the workflow when a message arrives |
| **How** | Click "+" on the canvas > search "Chat Trigger" > add it |
| **Done when** | You have a Chat Trigger node on the canvas. You can click "Chat" at the bottom of the n8n editor to open a test chat window |

### Task 2.3: Add the Message Classifier

| Detail | Info |
|--------|------|
| **What to do** | Add a node that classifies incoming messages into categories |
| **Why** | The chatbot needs to decide which branch to use |
| **How** | **Option A (AI):** Add an AI Agent or OpenAI node. Use the classification prompt from `chatbot-branching-logic.md`. **Option B (No AI):** Add a Switch node with keyword-based conditions. |
| **Done when** | The node correctly outputs a category (greeting, product_inquiry, faq, order_create, order_track, human_support, or unknown) |

**Testing checkpoint:** Send "Hi" through the test chat. The classifier should output "greeting". Send "Do you have t-shirts?" - it should output "product_inquiry".

### Task 2.4: Add the Switch/Router Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Switch node (or use the classifier outputs) to route messages to different branches |
| **Why** | Each category needs to go to a different set of nodes |
| **How** | Connect the classifier output to a Switch node. Create one output for each category. |
| **Done when** | Messages are correctly routed to 7 different outputs based on category |

---

## Phase 3: Build Branch 1 - Direct Reply

*Time estimate: 15 minutes*

### Task 3.1: Create the Direct Reply Response

| Detail | Info |
|--------|------|
| **What to do** | Add a "Respond to Chat" node for greetings |
| **Why** | Simple messages need quick, friendly replies |
| **How** | Add a "Respond to Chat" node. Connect it to the "greeting" output of your Switch. Set the response message (use examples from `chatbot-branching-logic.md`). |
| **Done when** | Sending "Hi" in the test chat returns a welcome message |

**Test messages to try:**
- "Hi" → Should get a welcome message
- "Thanks" → Should get a "you're welcome" message
- "Bye" → Should get a goodbye message

---

## Phase 4: Build Branch 2 - Product Lookup

*Time estimate: 30-45 minutes*

### Task 4.1: Add Product Search Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets node that searches the Products tab |
| **Why** | When customers ask about products, the bot needs to find them |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: Products > Set up search based on keywords from the message |
| **Done when** | Searching for "t-shirt" returns your t-shirt products |

### Task 4.2: Add "No Results" Check

| Detail | Info |
|--------|------|
| **What to do** | Add an IF node to check if products were found |
| **Why** | The bot needs to respond differently if nothing matches |
| **How** | Add IF node after Google Sheets > Condition: check if results array is not empty |
| **Done when** | Empty searches go to "no results" path, found products go to "format reply" path |

### Task 4.3: Format and Send Product Reply

| Detail | Info |
|--------|------|
| **What to do** | Format the product data into a nice message and send it |
| **Why** | Raw spreadsheet data looks ugly - make it customer-friendly |
| **How** | Add a Set node to format the message (use template from `chatbot-branching-logic.md`), then a Respond to Chat node |
| **Done when** | Asking "Show me t-shirts" returns a formatted product list |

**Test messages to try:**
- "Show me t-shirts" → Should return t-shirt products
- "Do you have hoodies?" → Should return hoodies
- "Show me purple sandals" → Should return "no products found" message

---

## Phase 5: Build Branch 3 - FAQ Answers

*Time estimate: 30 minutes*

### Task 5.1: Add FAQ Search Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets node that searches the FAQ tab |
| **Why** | Common questions need automatic answers |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: FAQ > Search the Keywords column |
| **Done when** | Searching for "shipping" returns your shipping FAQ entry |

### Task 5.2: Add "No Answer" Check

| Detail | Info |
|--------|------|
| **What to do** | Add an IF node for when no FAQ matches |
| **Why** | If no FAQ matches, offer alternatives |
| **How** | Same pattern as Task 4.2 - check if results are empty |
| **Done when** | Unmatched questions get a helpful fallback |

### Task 5.3: Format and Send FAQ Reply

| Detail | Info |
|--------|------|
| **What to do** | Send the FAQ answer back to the customer |
| **Why** | Deliver the answer in a friendly format |
| **How** | Add Set node + Respond to Chat node. Include the answer and a "was this helpful?" follow-up |
| **Done when** | Asking "What's your return policy?" returns the correct answer |

**Test messages to try:**
- "What's your return policy?" → Should return return policy
- "How long does shipping take?" → Should return shipping info
- "Do you accept Bitcoin?" → Should return "no answer found" message

---

## Phase 6: Build Branch 4 - Order Creation

*Time estimate: 45-60 minutes*

### Task 6.1: Plan the Order Conversation Flow

| Detail | Info |
|--------|------|
| **What to do** | Design how the bot will collect order information step by step |
| **Why** | Orders need multiple pieces of info (product, size, color, name, email) |
| **How** | Use an AI Agent node with memory (to remember previous messages in the conversation) OR build a multi-step form using sub-workflows |
| **Done when** | You have a plan for how the bot will ask each question |

**Beginner recommendation:** Use an AI Agent node with conversation memory. It is the easiest way to handle multi-turn order conversations.

### Task 6.2: Add Product Validation

| Detail | Info |
|--------|------|
| **What to do** | Before creating an order, verify the product exists and is in stock |
| **Why** | Cannot sell products that don't exist or are out of stock |
| **How** | Add a Google Sheets Search node to check the Products tab for the requested item |
| **Done when** | Bot confirms the product exists before proceeding, or tells customer it is unavailable |

### Task 6.3: Add Order Save Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets Append node to save orders |
| **Why** | Once all info is collected, the order needs to be stored |
| **How** | Add Google Sheets node > Operation: Append Row > Sheet: Orders > Map all fields |
| **Done when** | Completing an order flow creates a new row in the Orders tab |

### Task 6.4: Add Order Confirmation Reply

| Detail | Info |
|--------|------|
| **What to do** | Send the customer a confirmation message with their order number |
| **Why** | Customers need confirmation and an order number to track later |
| **How** | Add Respond to Chat node with order summary (see template in `chatbot-branching-logic.md`) |
| **Done when** | After order is saved, customer receives a confirmation with order number |

**Test the full flow:**
1. Say "I want to order the black t-shirt"
2. Provide size when asked
3. Provide name and email when asked
4. Confirm the order
5. Check Google Sheets - a new row should appear in the Orders tab!

---

## Phase 7: Build Branch 5 - Order Tracking

*Time estimate: 30 minutes*

### Task 7.1: Extract Order Number from Message

| Detail | Info |
|--------|------|
| **What to do** | Detect if the customer included an order number in their message |
| **Why** | If they already gave the order number, don't ask again |
| **How** | Use a Set node or Code node to look for pattern "ORD-" in the message. If not found, ask for it. |
| **Done when** | Bot correctly detects "ORD-20250120-001" in messages or asks for it |

### Task 7.2: Add Tracking Lookup Node

| Detail | Info |
|--------|------|
| **What to do** | Search the Tracking tab for the order |
| **Why** | Customers want to know where their package is |
| **How** | Add Google Sheets node > Operation: Search Rows > Sheet: Tracking > Search by Order_ID |
| **Done when** | Searching with a valid order number returns tracking data |

### Task 7.3: Add Tracking Reply

| Detail | Info |
|--------|------|
| **What to do** | Format and send tracking information to customer |
| **Why** | Show delivery status in a clear format |
| **How** | Add Set node + Respond to Chat. Include status, carrier, dates, and tracking link |
| **Done when** | Asking "Where is order ORD-20250120-001?" returns formatted tracking info |

**To test this:** Add a sample row to your Tracking tab first, then ask the bot about that order.

---

## Phase 8: Build Branch 6 - Human Support Handoff

*Time estimate: 30 minutes*

### Task 8.1: Add Support Ticket Creation Node

| Detail | Info |
|--------|------|
| **What to do** | Add a Google Sheets Append node for the Support Tickets tab |
| **Why** | When handoff happens, a ticket must be created for follow-up |
| **How** | Add Google Sheets node > Operation: Append Row > Sheet: Support Tickets > Map fields |
| **Done when** | Triggering handoff creates a new row in Support Tickets tab |

### Task 8.2: Add Priority Assignment Logic

| Detail | Info |
|--------|------|
| **What to do** | Automatically assign priority (High/Medium/Low) based on the issue |
| **Why** | Urgent issues need faster response |
| **How** | Add IF/Switch node before the ticket creation. Check for angry language → High, complaints → Medium, other → Low |
| **Done when** | Frustrated customers get High priority tickets, normal requests get Low/Medium |

### Task 8.3: Add Handoff Reply

| Detail | Info |
|--------|------|
| **What to do** | Tell the customer their ticket was created and set expectations |
| **Why** | Customer needs to know what happens next |
| **How** | Add Respond to Chat node with ticket number, expected response time, and alternative contact methods |
| **Done when** | Saying "I want to talk to someone" creates a ticket AND sends a confirmation to the customer |

**Test messages to try:**
- "I want to speak to a human" → Should create ticket (Low priority)
- "THIS IS TERRIBLE! I got the wrong order!" → Should create ticket (High priority)
- "I need help with an exchange" → Should create ticket (Medium priority)

---

## Phase 9: Build Fallback and Error Handling

*Time estimate: 20 minutes*

### Task 9.1: Add Fallback Reply

| Detail | Info |
|--------|------|
| **What to do** | Add a response for unrecognized messages |
| **Why** | The bot should never leave a customer without a response |
| **How** | Connect the "unknown/default" output of your Switch to a Respond to Chat node with the menu of options (see template in `chatbot-branching-logic.md`) |
| **Done when** | Sending random gibberish returns a helpful menu |

### Task 9.2: Add Error Handling for Google Sheets Failures

| Detail | Info |
|--------|------|
| **What to do** | Add error handling so the bot doesn't crash if Google Sheets is unreachable |
| **Why** | Sometimes connections fail - the bot should handle this gracefully |
| **How** | On each Google Sheets node, add an Error Output connection to a "sorry" message node |
| **Done when** | If Google Sheets times out, the customer gets a helpful error message instead of silence |

### Task 9.3: Add Automatic Handoff After 3 Failed Attempts

| Detail | Info |
|--------|------|
| **What to do** | Track failed understanding attempts and auto-handoff after 3 |
| **Why** | If the bot keeps failing, a human needs to step in |
| **How** | Use a counter variable (or conversation memory) to count fallback triggers. After 3, route to Branch 6. |
| **Done when** | Sending 3 unrecognized messages in a row triggers automatic human handoff |

---

## Phase 10: Testing

*Time estimate: 30-45 minutes*

### Task 10.1: Test Each Branch Individually

| Test | Message to Send | Expected Result |
|------|----------------|-----------------|
| Greeting | "Hello!" | Welcome message |
| Product found | "Show me t-shirts" | Product list |
| Product not found | "Show me bikinis" | Not found + suggestions |
| FAQ found | "Return policy?" | Return policy answer |
| FAQ not found | "Do you do custom embroidery?" | No answer + options |
| Order start | "I want to buy a hoodie" | Starts order collection |
| Order complete | (provide all details) | Order saved + confirmation |
| Tracking found | "Track order ORD-20250120-001" | Tracking details |
| Tracking not found | "Track order ORD-99999" | Not found message |
| Human handoff | "Talk to a real person" | Ticket created + confirmation |
| Angry handoff | "THIS IS USELESS!" | High priority ticket |
| Fallback | "asdfjkl" | Menu of options |
| Error | (disconnect Google Sheets) | Graceful error message |

### Task 10.2: Test Edge Cases

| Test | Message to Send | Expected Result |
|------|----------------|-----------------|
| Multi-intent | "Hi, I want to track my order" | Goes to tracking (not greeting) |
| Cancel mid-order | "Never mind" during order flow | Order cancelled gracefully |
| Empty message | "" | Fallback menu |
| Very long message | (paragraph of text) | Bot identifies main intent |
| Emoji only | Thumbs up emoji | Appropriate response |

### Task 10.3: Verify Google Sheets Data

| Check | How |
|-------|-----|
| Orders are being saved correctly | Open Orders tab after test orders |
| Support tickets are being created | Open Support Tickets tab after handoff tests |
| Data formatting is correct | Check dates, IDs, and statuses are filled correctly |
| No duplicate or blank rows | Scroll through each tab |

---

## Phase 11: Connect to a Real Channel (Optional - Do Later)

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
| **How** | In n8n, remove the Chat Trigger and add the appropriate channel trigger (WhatsApp node, Webhook for website, etc.) |
| **Done when** | A real message from the chosen channel triggers your workflow |

### Task 11.3: Test with Real Messages

| Detail | Info |
|--------|------|
| **What to do** | Have a friend or second account send test messages |
| **Why** | Real-world messages may differ from your test messages |
| **How** | Send messages from the connected channel and verify responses |
| **Done when** | 10+ real messages all get correct responses |

---

## Phase 12: Go Live Checklist

Before letting real customers use your chatbot, verify:

- [ ] All 6 branches respond correctly
- [ ] Error messages are friendly and helpful
- [ ] Google Sheets connection is stable
- [ ] Support tickets are being created with correct priority
- [ ] Order confirmation messages include all details
- [ ] Fallback message is helpful (not confusing)
- [ ] You have a plan to check Support Tickets tab daily
- [ ] You have a plan to review draft orders daily
- [ ] You have told your team about the chatbot
- [ ] You have a backup plan if the chatbot goes down (e.g., email support)

---

## Quick Reference: Task Summary

| Phase | Tasks | Time |
|-------|-------|------|
| 1. Setup | 4 tasks | 30-45 min |
| 2. Core Workflow | 4 tasks | 45-60 min |
| 3. Direct Reply | 1 task | 15 min |
| 4. Product Lookup | 3 tasks | 30-45 min |
| 5. FAQ Answers | 3 tasks | 30 min |
| 6. Order Creation | 4 tasks | 45-60 min |
| 7. Order Tracking | 3 tasks | 30 min |
| 8. Human Handoff | 3 tasks | 30 min |
| 9. Error Handling | 3 tasks | 20 min |
| 10. Testing | 3 tasks | 30-45 min |
| 11. Channel Connect | 3 tasks | 1-2 hours |
| 12. Go Live | Checklist | 15 min |
| **Total** | **34 tasks** | **3-5 hours** |

---

## Tips for Success

1. **Build one branch at a time.** Don't try to build everything at once.
2. **Test constantly.** After every node you add, test it with the chat window.
3. **Start with the easiest branches first** (Direct Reply, then FAQ, then Products).
4. **Save your workflow often.** n8n autosaves, but click save manually too.
5. **If something breaks, undo.** n8n has an undo button (Ctrl+Z).
6. **Keep your Google Sheets data clean.** No blank rows, consistent formatting.
7. **Ask for help.** The n8n community forum is very helpful for beginners.
8. **Don't skip testing.** A broken chatbot is worse than no chatbot.

---

## What To Do Next

After completing all tasks above:

1. Monitor the chatbot daily for the first week
2. Check Support Tickets tab for issues the bot could not handle
3. Add more FAQ entries based on common questions you notice
4. Add more products as your catalog grows
5. Consider adding more features (order updates, promotional messages, etc.)

**Congratulations!** You will have built a fully functional chatbot for your clothing brand!
