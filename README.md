# Clothing Brand Chatbot — n8n + Google Sheets

## What Is This Project?

This is a **chatbot system** for a clothing brand. It automatically answers customer questions, helps them find products, creates orders, tracks deliveries, and connects them to a human when needed.

**You do NOT need to know how to code.** This project uses:

- **n8n** (a visual automation tool — you build workflows by dragging and connecting boxes)
- **Google Sheets** (used as your database to store products, orders, FAQs, etc.)

---

## What Can This Chatbot Do?

| # | Feature | What It Does |
|---|---------|--------------|
| 1 | **Direct Reply** | Answers simple messages like "Hi" or "Thanks" instantly |
| 2 | **Product Lookup** | Finds specific product variants (size + color) from your Google Sheets catalog |
| 3 | **FAQ Answers** | Answers common questions (shipping, returns, sizing, etc.) |
| 4 | **Draft Order Creation** | Creates a new order with full details and saves it to Google Sheets |
| 5 | **Order Tracking** | Looks up order status and shipping info from Google Sheets |
| 6 | **Human Support Handoff** | Connects the customer to a real person and creates a detailed support ticket |

---

## Where Can This Chatbot Work?

Once built, you can connect it to:

- Website live chat widget
- WhatsApp Business
- Facebook Messenger
- Instagram DMs

You start by building the logic first, then connect your preferred channel later.

---

## Google Sheets Database Structure

Your chatbot uses **one Google Sheets spreadsheet** with **6 tabs**:

| Tab Name | What It Stores | Key Columns |
|----------|---------------|-------------|
| **Products** | Product catalog (one row per variant: size + color) | Product_ID, Variant_SKU, Size, Color, Stock_Qty, Search_Keywords, Active |
| **FAQ** | Common questions and answers | FAQ_ID, Keywords, Answer, Active, Sort_Order |
| **Orders** | Customer orders with full details | Order_ID, Customer_ID, Variant_SKU, Payment_Status, Order_Status, Channel |
| **Tracking** | Shipping and delivery information | Order_ID, Tracking_Number, Shipping_Status, Updated_By |
| **Customers** | Customer profiles and history | Customer_ID, Chat_User_ID, Total_Orders, Total_Spent |
| **Support_Tickets** | Human support requests with conversation context | Ticket_ID, Conversation_ID, Bot_Summary, Handoff_Reason, Priority |

See `google-sheets-structure.md` for the complete column list, examples, and setup instructions.

---

## Project Files Explained

| File | What It Contains |
|------|-----------------|
| `README.md` | This file — project overview |
| `google-sheets-structure.md` | How to set up your Google Sheets database (6 tabs, all columns, examples) |
| `n8n-workflow-plan.md` | Which n8n nodes you need and how they connect |
| `chatbot-branching-logic.md` | How the chatbot decides what to do with each message |
| `build-tasks.md` | Step-by-step tasks to build everything |

---

## What You Need Before Starting

1. **A Google account** (for Google Sheets)
2. **n8n installed** (free self-hosted version or n8n Cloud)
3. **About 3-5 hours** to build the full system
4. **No coding experience required**

---

## How To Use This Plan

1. Read this README first (you are here!)
2. Set up your Google Sheets using `google-sheets-structure.md`
3. Understand the chatbot logic in `chatbot-branching-logic.md`
4. Review the n8n plan in `n8n-workflow-plan.md`
5. Follow the tasks in `build-tasks.md` to build it step by step

---

## Key Design Decisions

| Decision | Why |
|----------|-----|
| **One row per product variant** | Precise stock tracking — know exactly how many size-M black t-shirts you have |
| **Customer_ID links across tabs** | See a customer's full history (orders, tickets, spending) in one place |
| **Variant_SKU in orders** | Know exactly which size + color was ordered, no ambiguity |
| **Bot_Summary in support tickets** | Your team gets instant context without re-reading the whole chat |
| **Handoff_Reason tracking** | Helps you improve the bot over time by seeing why it fails |
| **Active/In_Stock flags** | Hide products or FAQs without deleting them |
| **Channel tracking everywhere** | Know which platform (WhatsApp, web, etc.) works best for your customers |

---

## Important Notes

- This repository contains the **planning documents only**
- The actual n8n workflow will be built later following these plans
- No API keys, passwords, or private information are stored here
- All data is stored in YOUR Google Sheets (you control everything)

---

## Glossary (Terms You Might Not Know)

| Term | Meaning |
|------|---------|
| **n8n** | A free automation tool where you connect "nodes" (boxes) to build workflows |
| **Node** | A single step in an n8n workflow (like "Read from Google Sheets") |
| **Workflow** | A series of connected nodes that run automatically |
| **Trigger** | The first node — it starts the workflow (e.g., when a message arrives) |
| **Google Sheets Tab** | A separate sheet within your spreadsheet (like pages in a book) |
| **Variant** | A specific version of a product (e.g., "Black T-Shirt, Size M" is one variant) |
| **SKU** | Stock Keeping Unit — a unique code for each product variant |
| **Handoff** | When the bot transfers the conversation to a real human |
| **Channel** | The platform a customer uses to chat (WhatsApp, website, Messenger, Instagram) |
| **API** | A way for two apps to talk to each other (n8n handles this for you) |

---

## License

This project plan is free to use for your clothing brand.
