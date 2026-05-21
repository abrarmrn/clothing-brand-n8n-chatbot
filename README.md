# Clothing Brand Chatbot - n8n + Google Sheets

## What Is This Project?

This is a **chatbot system** for a clothing brand. It automatically answers customer questions, helps them find products, creates orders, tracks deliveries, and connects them to a human when needed.

**You do NOT need to know how to code.** This project uses:

- **n8n** (a visual automation tool - you build workflows by dragging and connecting boxes)
- **Google Sheets** (used as your database to store products, orders, FAQs, etc.)

---

## What Can This Chatbot Do?

| # | Feature | What It Does |
|---|---------|--------------|
| 1 | **Direct Reply** | Answers simple messages like "Hi" or "Thanks" instantly |
| 2 | **Product Lookup** | Finds products from your Google Sheets catalog |
| 3 | **FAQ Answers** | Answers common questions (shipping, returns, sizing, etc.) |
| 4 | **Draft Order Creation** | Creates a new order and saves it to Google Sheets |
| 5 | **Order Tracking** | Looks up order status from Google Sheets |
| 6 | **Human Support Handoff** | Connects the customer to a real person when the bot cannot help |
| 7 | **Style Recommendations** | Gives outfit and styling advice based on your catalog |

---

## Where Can This Chatbot Work?

Once built, you can connect it to:

- Website live chat widget
- WhatsApp Business
- Facebook Messenger
- Instagram DMs

You start by building the logic first, then connect your preferred channel later.

---

## Project Files Explained

| File | What It Contains |
|------|-----------------|
| `README.md` | This file - project overview |
| `google-sheets-structure.md` | How to set up your Google Sheets database |
| `n8n-workflow-plan.md` | Which n8n nodes you need and how they connect |
| `chatbot-branching-logic.md` | How the chatbot decides what to do with each message |
| `build-tasks.md` | Step-by-step tasks to build everything |
| `n8n-messenger-hybrid-ai-v4.json` | **Importable n8n workflow** - Messenger + AI hybrid (v4) |
| `messenger-hybrid-v4-import-instructions.md` | Setup guide for importing and configuring the v4 workflow |

---

## What You Need Before Starting

1. **A Google account** (for Google Sheets)
2. **n8n installed** (free self-hosted version or n8n Cloud)
3. **About 2-4 hours** to build the full system
4. **No coding experience required**

---

## How To Use This Plan

1. Read this README first (you are here!)
2. **Fastest path:** Import `n8n-messenger-hybrid-ai-v4.json` into n8n and follow `messenger-hybrid-v4-import-instructions.md`
3. **Learning path:** Set up Google Sheets using `google-sheets-structure.md`, then follow `build-tasks.md` step by step
4. Understand the chatbot logic in `chatbot-branching-logic.md`
5. Review the full architecture in `n8n-workflow-plan.md`

---

## Important Notes

- This repository contains **planning documents** and a **ready-to-import n8n workflow** (v4)
- The v4 workflow (`n8n-messenger-hybrid-ai-v4.json`) can be imported directly into n8n
- No API keys, passwords, or private information are stored here (only placeholders)
- All data is stored in YOUR Google Sheets (you control everything)
- Follow `messenger-hybrid-v4-import-instructions.md` for the fastest path to a working chatbot

---

## Glossary (Terms You Might Not Know)

| Term | Meaning |
|------|---------|
| **n8n** | A free automation tool where you connect "nodes" (boxes) to build workflows |
| **Node** | A single step in an n8n workflow (like "Read from Google Sheets") |
| **Workflow** | A series of connected nodes that run automatically |
| **Trigger** | The first node - it starts the workflow (e.g., when a message arrives) |
| **Google Sheets Tab** | A separate sheet within your spreadsheet (like pages in a book) |
| **Handoff** | When the bot transfers the conversation to a real human |
| **API** | A way for two apps to talk to each other (n8n handles this for you) |

---

## License

This project plan is free to use for your clothing brand.
