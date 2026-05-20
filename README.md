# Clothing Brand Chatbot — Facebook Messenger + n8n + Google Sheets

## What Is This Project?

This is a **Facebook Messenger chatbot** for a clothing brand. Customers message your Facebook Page, and the bot automatically answers questions, helps them find products, creates orders, tracks deliveries, and connects them to a human when needed.

**You do NOT need to know how to code.** This project uses:

- **Facebook Messenger** (where customers chat with your Page)
- **n8n** (a visual automation tool — receives messages and sends replies)
- **Google Sheets** (used as your database to store products, orders, FAQs, etc.)

---

## Current Status: v1 — Product Lookup

This project is being built incrementally. **Version 1 is ready to import:**

| Version | Feature | Status |
|---------|---------|--------|
| **v1** | Product Lookup via Messenger | ✅ Ready — importable JSON included |
| v2 | FAQ Answers | Planned |
| v3 | Direct Reply (greetings) | Planned |
| v4 | Order Creation | Planned |
| v5 | Order Tracking | Planned |
| v6 | Human Support Handoff | Planned |

### What v1 Does

1. Customer sends a message to your Facebook Page (e.g., "Show me black t-shirts")
2. Meta sends the message to your n8n webhook
3. n8n searches your Google Sheets Products tab
4. n8n sends the matching products back to the customer in Messenger

---

## How It Works (Architecture)

```
Customer (Messenger) → Facebook/Meta → n8n Webhook → Google Sheets → n8n → Messenger Send API → Customer
```

The chatbot uses **Facebook Messenger webhooks**, not n8n's built-in Chat Trigger:

| Component | Role |
|-----------|------|
| **Facebook Page** | What customers message |
| **Meta Webhooks** | Delivers messages from Messenger to your n8n URL |
| **n8n (GET Webhook)** | Verifies your webhook with Meta (one-time setup) |
| **n8n (POST Webhook)** | Receives every customer message |
| **Google Sheets** | Stores products, orders, FAQs, customers, tickets |
| **Messenger Send API** | Sends replies back to the customer |

---

## What Will This Chatbot Do? (Full Plan)

| # | Feature | What It Does |
|---|---------|--------------|
| 1 | **Direct Reply** | Answers simple messages like "Hi" or "Thanks" instantly |
| 2 | **Product Lookup** | Finds specific product variants (size + color) from your Google Sheets catalog |
| 3 | **FAQ Answers** | Answers common questions (shipping, returns, sizing, etc.) |
| 4 | **Draft Order Creation** | Creates a new order with full details and saves it to Google Sheets |
| 5 | **Order Tracking** | Looks up order status and shipping info from Google Sheets |
| 6 | **Human Support Handoff** | Connects the customer to a real person and creates a detailed support ticket |

---

## Quick Start (v1 — Product Lookup)

1. Set up your Google Sheets using `google-sheets-structure.md`
2. Add product data to the Products tab (at least 5 products with variants)
3. Import `n8n-messenger-product-lookup-v1.json` into n8n
4. Follow `messenger-import-instructions.md` to connect everything
5. Test by messaging your Facebook Page!

---

## Project Files

| File | What It Contains |
|------|-----------------|
| `README.md` | This file — project overview |
| **`n8n-messenger-product-lookup-v1.json`** | **Importable n8n workflow** — product lookup via Messenger |
| **`messenger-import-instructions.md`** | **Step-by-step setup guide** — import, credentials, Meta webhooks |
| `google-sheets-structure.md` | How to set up your Google Sheets database (6 tabs, all columns, examples) |
| `n8n-workflow-plan.md` | Full workflow architecture for all 6 branches |
| `chatbot-branching-logic.md` | How the chatbot decides what to do with each message |
| `build-tasks.md` | Step-by-step tasks to build the full system |

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

## What You Need Before Starting

1. **A Facebook Page** for your clothing brand
2. **A Meta Developer account** ([developers.facebook.com](https://developers.facebook.com))
3. **n8n running** with a public HTTPS URL (n8n Cloud or self-hosted with domain)
4. **A Google account** (for Google Sheets)
5. **No coding experience required**

---

## Key Design Decisions

| Decision | Why |
|----------|-----|
| **Facebook Messenger webhooks (not n8n Chat Trigger)** | Real production channel — customers message your actual Facebook Page |
| **GET webhook for Meta verification** | Required by Meta to prove you own the webhook URL |
| **POST webhook responds 200 immediately** | Meta retries if you don't respond within 5 seconds — so we ACK first, process after |
| **One row per product variant** | Precise stock tracking — know exactly how many size-M black t-shirts you have |
| **Sender PSID for customer identity** | Messenger uses Page-Scoped IDs to identify each customer uniquely |
| **HTTP Request node for Send API** | n8n sends replies back through Facebook's Graph API |
| **Incremental build (v1, v2, v3...)** | Start with working product lookup, add features one at a time |

---

## Important Notes

- This repository contains planning documents AND an importable workflow JSON
- **No API keys, passwords, or tokens are stored in this repo** — all credentials go inside n8n
- The JSON file uses placeholder values (`CHANGE_ME_VERIFY_TOKEN`, `REPLACE_WITH_YOUR_CREDENTIAL_ID`)
- All data is stored in YOUR Google Sheets (you control everything)
- The workflow requires n8n to have a **public HTTPS URL** (Meta won't send to localhost)

---

## Glossary (Terms You Might Not Know)

| Term | Meaning |
|------|---------|
| **n8n** | A free automation tool where you connect "nodes" (boxes) to build workflows |
| **Node** | A single step in an n8n workflow (like "Read from Google Sheets") |
| **Webhook** | A URL that receives data from another service (Meta sends messages here) |
| **Meta** | The company that owns Facebook, Messenger, Instagram, and WhatsApp |
| **PSID** | Page-Scoped ID — Messenger's unique identifier for each person messaging your Page |
| **Page Access Token** | A password that lets n8n send messages on behalf of your Facebook Page |
| **Verify Token** | A secret you create to prove webhook ownership during Meta setup |
| **Graph API** | Facebook's API for sending messages, reading data, etc. |
| **Variant** | A specific version of a product (e.g., "Black T-Shirt, Size M" is one variant) |
| **SKU** | Stock Keeping Unit — a unique code for each product variant |
| **Handoff** | When the bot transfers the conversation to a real human |
| **Google Sheets Tab** | A separate sheet within your spreadsheet (like pages in a book) |

---

## License

This project plan is free to use for your clothing brand.
