# Clothing Brand Chatbot — Facebook Messenger + n8n + Google Sheets

## What Is This Project?

This is a **Facebook Messenger chatbot** for a clothing brand. Customers message your Facebook Page, and the bot automatically answers questions, helps them find products, creates orders, tracks deliveries, and connects them to a human when needed.

**You do NOT need to know how to code.** This project uses:

- **Facebook Messenger** (where customers chat with your Page)
- **n8n** (a visual automation tool — receives messages and sends replies)
- **Google Sheets** (used as your database to store products, orders, FAQs, etc.)

---

## Current Status: v6 — Voice + Short Replies

This project is built incrementally:

| Version | Feature | Status |
|---------|---------|--------|
| **v1** | Product Lookup only | ✅ Ready (backup) |
| **v2** | Hybrid AI — Greetings + FAQ + Products + Tracking + Support + AI Fallback | ✅ Ready (backup) |
| **v3** | Fixed AI + Language Detection + Telegram Alerts + BD Brand | ✅ Ready (backup) |
| **v4** | Style routing + message.mid dedup + canonical output (experimental) | ✅ Ready (backup) |
| **v5** | Style routing, dedup, memory, typo normalization, size guide, objection handling | ✅ Ready (backup) |
| **v6** | Voice/audio transcription, short mixed Bangla+English replies, fixed AI Agent, unclear message handling, improved memory | ✅ **Latest — recommended** |

### What v2 Does (Hybrid AI — Cost-Saving Design)

The bot answers **most messages for free** using rules and Google Sheets. AI is only used as a last resort.

| Message Type | How It's Handled | AI Cost? |
|-------------|-----------------|----------|
| Greetings (hi, hello, assalamualaikum) | Static reply — no lookup needed | **Free** |
| FAQ questions (shipping, returns, sizing) | Matched from FAQ tab in Google Sheets | **Free** |
| Product search (show me t-shirts) | Searched from Products tab in Google Sheets | **Free** |
| Order tracking (where is my order) | Rule-based detection + Tracking tab | **Free** |
| Human support (speak to a human, angry) | Rule-based detection + ticket handler | **Free** |
| Complex/unclear questions | AI Agent generates a helpful reply | **Costs money** |

**Expected:** 70-90% of messages are handled without AI. Only 10-30% use the AI fallback.

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
| 7 | **Style Recommendations** | Gives outfit and styling advice based on your catalog |

---

## Quick Start (v6 — Latest)

1. Set up your Google Sheets using `google-sheets-structure.md`
2. Add product data to Products tab + FAQ data to FAQ tab
3. (Optional) Add Size_Guide and Sales_Objections tabs
4. Import `n8n-messenger-hybrid-ai-v6.json` into n8n
5. Follow `messenger-hybrid-v6-import-instructions.md` to connect credentials
6. Run tests from `messenger-hybrid-v6-test-plan.md`
7. Activate and message your Facebook Page!

**Previous versions (kept as backup):**
- v5: `n8n-messenger-hybrid-ai-v5.json` + `messenger-hybrid-v5-import-instructions.md`
- v3: `n8n-messenger-hybrid-ai-v3.json` + `messenger-hybrid-v3-import-instructions.md`
- v2: `n8n-messenger-hybrid-ai-v2.json` + `messenger-hybrid-v2-import-instructions.md`
- v1: `n8n-messenger-product-lookup-v1.json` + `messenger-import-instructions.md`

---

## Project Files

| File | What It Contains |
|------|-----------------|
| `README.md` | This file — project overview |
| **`n8n-messenger-hybrid-ai-v6.json`** | **Importable n8n workflow v6** — voice support, short mixed replies, fixed AI Agent |
| **`messenger-hybrid-v6-import-instructions.md`** | **v6 setup guide** — credentials, voice, testing |
| **`messenger-hybrid-v6-test-plan.md`** | **v6 test plan** — 40+ test cases including voice/audio |
| `n8n-messenger-hybrid-ai-v5.json` | Workflow v5 (stable backup — rollback target) |
| `messenger-hybrid-v5-import-instructions.md` | v5 setup guide (backup) |
| `messenger-hybrid-v5-test-plan.md` | v5 test plan (backup) |
| `n8n-messenger-hybrid-ai-v4.json` | Workflow v4 (experimental backup) |
| `messenger-hybrid-v4-import-instructions.md` | v4 setup guide (backup) |
| `n8n-messenger-hybrid-ai-v3.json` | Workflow v3 (stable backup — rollback target) |
| `messenger-hybrid-v3-import-instructions.md` | v3 setup guide (backup) |
| `n8n-messenger-hybrid-ai-v2.json` | Workflow v2 (backup) |
| `messenger-hybrid-v2-import-instructions.md` | v2 setup guide (backup) |
| `n8n-messenger-product-lookup-v1.json` | Workflow v1 — product lookup only (backup) |
| `messenger-import-instructions.md` | v1 setup guide (backup) |
| `google-sheets-structure.md` | How to set up your Google Sheets database |
| `n8n-workflow-plan.md` | Full workflow architecture |
| `chatbot-branching-logic.md` | How the chatbot decides what to do with each message |
| `build-tasks.md` | Step-by-step tasks to build and test the system |

---

## Google Sheets Database Structure

Your chatbot uses **one Google Sheets spreadsheet** with **8 tabs** (6 required + 2 optional):

| Tab Name | What It Stores | Required? |
|----------|---------------|-----------|
| **Products** | Product catalog (one row per variant: size + color) | ✅ Required |
| **FAQ** | Common questions and answers | ✅ Required |
| **Orders** | Customer orders with full details | ✅ Required |
| **Tracking** | Shipping and delivery information | ✅ Required |
| **Customers** | Customer profiles and history | ✅ Required |
| **Support_Tickets** | Human support requests with conversation context | ✅ Required |
| **Size_Guide** | Size recommendations by height/weight (v5) | Optional |
| **Sales_Objections** | Objection handling responses (v5) | Optional |

See `google-sheets-structure.md` for the complete column list, examples, and setup instructions.

---

## What You Need Before Starting

1. **A Facebook Page** for your clothing brand
2. **A Meta Developer account** ([developers.facebook.com](https://developers.facebook.com))
3. **n8n running** with a public HTTPS URL (n8n Cloud or self-hosted with domain)
4. **A Google account** (for Google Sheets)
5. **An AI API key** (OpenAI or compatible) — for the fallback AI Agent only
6. **A Telegram Bot** (optional) — for human support alerts to your team
7. **No coding experience required**

---

## Key Design Decisions

| Decision | Why |
|----------|-----|
| **Language-aware replies** | Detects Bangla/Banglish/English and replies in same style |
| **Telegram support alerts** | Team gets instant Telegram notification when human needed |
| **Hybrid AI (rules first, AI fallback only)** | Saves money — 70-90% of messages answered free from Google Sheets |
| **Facebook Messenger webhooks (not n8n Chat Trigger)** | Real production channel — customers message your actual Facebook Page |
| **GET webhook for Meta verification** | Required by Meta to prove you own the webhook URL |
| **POST webhook responds 200 immediately** | Meta retries if you don't respond within 5 seconds — so we ACK first, process after |
| **One row per product variant** | Precise stock tracking — know exactly how many size-M black t-shirts you have |
| **Sender PSID for customer identity** | Messenger uses Page-Scoped IDs to identify each customer uniquely |
| **HTTP Request node for Send API** | n8n sends replies back through Facebook's Graph API |
| **AI Agent system prompt restricts topics** | Prevents AI from inventing stock levels, prices, or order statuses |
| **Incremental build (v1 → v2 → v3 → v4 → v5)** | Start simple, add complexity gradually, always have a working backup |
| **v5: Style routed before order intent** | "What should I wear?" is advice, not a purchase — prevents misclassification |
| **v5: Single final Send node** | All branches output canonical `{replyText, senderPsid}` shape |
| **v5: message.mid deduplication** | Prevents double-replies when Meta retries webhook delivery |
| **v5: Weak order keywords removed** | "i want", "chai", "lagbe" alone no longer trigger order flow |
| **v5: Customer memory** | Remembers product interest, size, color across messages |
| **v5: Typo normalization** | Auto-corrects "delivry", "tshart", "hudie" before routing |
| **v5: Size guide + objection handling** | Dedicated branches for common BD shopping patterns |

**Fastest path to a working chatbot:** Import `n8n-messenger-hybrid-ai-v5.json` and follow `messenger-hybrid-v5-import-instructions.md`.

---

## Important Notes

- This repository contains planning documents AND importable workflow JSONs (v1, v2, v3, v4, and v5)
- **No API keys, passwords, or tokens are stored in this repo** — all credentials go inside n8n
- JSON files use placeholder values (`CHANGE_ME_VERIFY_TOKEN`, `GOOGLE_SHEETS_CREDENTIAL_PLACEHOLDER`, `FACEBOOK_PAGE_ACCESS_TOKEN_PLACEHOLDER`, `AI_CHAT_MODEL_CREDENTIAL_PLACEHOLDER`, `TELEGRAM_BOT_TOKEN_PLACEHOLDER`)
- All data is stored in YOUR Google Sheets (you control everything)
- The workflow requires n8n to have a **public HTTPS URL** (Meta won't send to localhost)
- v1, v2, v3, and v4 JSONs are kept as backups — do not delete them
- v5 is the recommended production version (upgraded from stable v3)

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
| **AI Agent** | An n8n node that uses an AI model (like GPT) to generate responses |
| **Chat Model** | The AI model connected to the AI Agent (e.g., gpt-4o-mini) |
| **Hybrid routing** | Checking rules/sheets first, using AI only as a last resort |
| **Google Sheets Tab** | A separate sheet within your spreadsheet (like pages in a book) |

---

## License

This project plan is free to use for your clothing brand.
