# Messenger Hybrid AI v4 - Import & Setup Instructions

## Overview

This guide walks you through importing the **n8n-messenger-hybrid-ai-v4.json** workflow into your n8n instance and connecting it to Facebook Messenger.

**What this workflow does:**

- Receives messages from Facebook Messenger via webhook
- Deduplicates messages using `message.mid` to prevent double-replies
- Classifies intent using OpenAI (GPT-4o-mini) with 8 categories
- Routes to the correct branch (greeting, product, FAQ, order, tracking, support, style, fallback)
- Every branch outputs a canonical `{ replyText, senderId }` object
- A **single** final HTTP node sends the reply via the Messenger Send API

**Architecture:** Webhook → Extract → Dedup → AI Classify → Route → Branch → Single Send Node

---

## Prerequisites

Before importing, make sure you have:

| Requirement | Details |
|-------------|---------|
| **n8n instance** | Self-hosted (v1.30+) or n8n Cloud |
| **Facebook Page** | A Facebook Page for your brand |
| **Facebook App** | A Facebook Developer App with Messenger product enabled |
| **Page Access Token** | Generated from your Facebook App's Messenger settings |
| **Verify Token** | A secret string you choose (used during webhook setup) |
| **OpenAI API Key** | For the AI intent classifier (GPT-4o-mini) |
| **Google Sheets** | Set up per `google-sheets-structure.md` (for production branches) |

---

## Step 1: Import the Workflow

1. Open your n8n instance
2. Click **"Add Workflow"** (or the "+" icon)
3. Click the **"..."** menu (top-right) → **"Import from File"**
4. Select `n8n-messenger-hybrid-ai-v4.json`
5. The workflow will appear on your canvas with all 21 nodes connected

**What you should see:**
- Left side: Two webhook triggers (GET for verification, POST for messages)
- Middle: Extract → Dedup → AI Classifier → Router
- Right side: 8 branch nodes all connecting to one "Send Messenger Reply" node

---

## Step 2: Set Up OpenAI Credential

1. In n8n, go to **Settings → Credentials → Add Credential**
2. Search for **"OpenAI"** and select it
3. Enter your OpenAI API key
4. Name it: `OpenAI API Key`
5. Save the credential
6. Open the **"AI Intent Classifier"** node on the canvas
7. In the credential dropdown, select your newly created OpenAI credential

**Cost note:** GPT-4o-mini is very cheap (~$0.15 per 1M input tokens). A typical classification call uses ~200 tokens, so 10,000 messages ≈ $0.30.

---

## Step 3: Configure Your Tokens

You need to replace **3 placeholder values** in the workflow:

### 3a. Page Access Token

1. Open the **"Send Messenger Reply"** node
2. Find the Query Parameter: `access_token`
3. Replace `YOUR_PAGE_ACCESS_TOKEN_HERE` with your actual Page Access Token

**Where to get it:**
- Go to [Facebook Developers](https://developers.facebook.com)
- Open your App → Messenger → Settings
- Under "Access Tokens", generate a token for your Page

### 3b. Verify Token

1. Open the **"Check Verify Token"** node
2. Find the condition checking `hub.verify_token`
3. Replace `YOUR_VERIFY_TOKEN_HERE` with a secret string you choose

**Example:** `my_clothing_brand_verify_2025` (any random string - you'll use this same string when setting up the webhook in Facebook)

### 3c. OpenAI Credential ID

This is handled automatically when you select the credential in Step 2. No manual editing needed.

---

## Step 4: Activate the Workflow

1. Click the **"Active"** toggle (top-right) to turn the workflow ON
2. n8n will display the webhook URLs. Copy the **POST** webhook URL.

**Your webhook URL will look like:**
```
https://your-n8n-domain.com/webhook/messenger-webhook
```

Or for n8n Cloud:
```
https://your-instance.app.n8n.cloud/webhook/messenger-webhook
```

---

## Step 5: Register Webhook with Facebook

1. Go to [Facebook Developers](https://developers.facebook.com) → Your App
2. Navigate to **Messenger → Settings → Webhooks**
3. Click **"Add Callback URL"**
4. Enter:
   - **Callback URL:** Your n8n webhook URL (the POST one from Step 4)
   - **Verify Token:** The same string you used in Step 3b
5. Click **"Verify and Save"**
6. Subscribe to these webhook fields:
   - ✅ `messages`
   - ✅ `messaging_postbacks` (optional, for button responses)
7. Under "Webhooks", select your **Page** to subscribe to

**If verification fails:**
- Make sure the workflow is ACTIVE in n8n
- Check that the verify token matches exactly (case-sensitive)
- Ensure your n8n instance is publicly accessible (not localhost)

---

## Step 6: Test the Integration

### Quick Test via Facebook Page

1. Open your Facebook Page
2. Click **"New Message"** (or message your page from a personal account)
3. Send: `Hi`
4. You should receive a welcome message back within 1-2 seconds

### Test Each Intent

| Send This | Expected Intent | Expected Response |
|-----------|----------------|-------------------|
| `Hi` | greeting | Welcome message with menu |
| `Do you have black t-shirts?` | product_inquiry | Product search response |
| `What's your return policy?` | faq | FAQ answer |
| `I want to buy a hoodie in large` | order_create | Order collection prompt |
| `Where is my order ORD-20250120-001?` | order_track | Tracking lookup response |
| `I want to talk to a human` | human_support | Support ticket confirmation |
| `What should I wear to a party?` | style_inquiry | Style recommendations |
| `asdfghjkl` | unknown | Fallback menu |

### Test Deduplication

Facebook sometimes sends the same message event twice. The dedup node uses `message.mid` to catch this. In production, connect this to Redis or a Google Sheets "ProcessedMessages" tab for true cross-execution deduplication.

---

## Step 7: Production Enhancements (Optional)

The imported workflow is fully functional for routing and responding. To connect real data:

### Connect Google Sheets (Product Lookup, FAQ, Orders, Tracking)

1. Add a **Google Sheets credential** in n8n (OAuth2)
2. In each Branch node that references Google Sheets, replace the placeholder Code node with:
   - **Google Sheets → Search Rows** (for Products, FAQ, Tracking lookups)
   - **Google Sheets → Append Row** (for Order creation, Support Tickets)
3. Keep the same output shape: `{ replyText, senderId }`

### Implement Real Deduplication

Replace the "Dedup Check (message.mid)" Code node with:

**Option A - Redis:**
- Add a Redis node to check/set the `messageId` with a 5-minute TTL

**Option B - Google Sheets:**
- Add a "ProcessedMessages" tab
- Search for the `messageId` before processing
- Append the `messageId` after processing

### Add Conversation Memory (for Order Flow)

For the Order Create branch to work as a multi-turn conversation:
1. Replace the Code node with an **AI Agent** node
2. Enable **Window Buffer Memory** (stores last N messages per sender)
3. Add a system prompt with order collection instructions

---

## Workflow Node Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  [GET Webhook] → [Verify Token?] → [Respond Challenge] / [Reject 403]  │
│                                                                         │
│  [POST Webhook] → [ACK 200] ─────────────────────────────────────────  │
│        │                                                                │
│        └──→ [Extract Message] → [Dedup Check] → [Is Not Duplicate?]    │
│                                                       │                 │
│                                          [AI Classifier] → [Normalize]  │
│                                                               │         │
│                                                    [Intent Router]       │
│                                                    /  |  |  |  \        │
│                                        greeting ─/   |  |  |   \─ unknown
│                                        product ─────/   |  |              │
│                                        faq ────────────/   |              │
│                                        order_create ──────/               │
│                                        order_track                        │
│                                        human_support                      │
│                                        style_inquiry                      │
│                                                    \  |  |  |  /        │
│                                                     v v  v  v v         │
│                                              [Send Messenger Reply]      │
│                                              (SINGLE FINAL NODE)         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Webhook verification fails | Ensure workflow is active, verify token matches exactly, n8n is publicly accessible |
| No response from bot | Check n8n execution log for errors. Verify Page Access Token is valid |
| "Message failed to send" | Page Access Token may be expired. Regenerate in Facebook Developer Console |
| Double replies | Dedup node needs external store (Redis/Sheets) for cross-execution dedup |
| Wrong intent classification | Check OpenAI credential is connected. Review the AI classifier system prompt |
| 403 from Facebook API | Your app may need App Review for pages you don't own/admin |
| Slow responses (>5s) | OpenAI API latency. Consider caching common classifications |

---

## Security Notes

- **Never commit real tokens** to this repository. The JSON uses placeholders only.
- **Page Access Tokens** should be stored as n8n credentials, not hardcoded.
- **Verify Token** should be a strong random string (16+ characters recommended).
- **HTTPS required** - Facebook only sends webhooks to HTTPS endpoints.
- The **message.mid dedup** prevents replay attacks from duplicate delivery.

---

## File Reference

| File | Purpose |
|------|---------|
| `n8n-messenger-hybrid-ai-v4.json` | The importable n8n workflow (this guide helps you set it up) |
| `google-sheets-structure.md` | How to set up the Google Sheets database |
| `chatbot-branching-logic.md` | Intent classification logic and keyword lists |
| `n8n-workflow-plan.md` | Full node-by-node workflow architecture |
| `build-tasks.md` | Step-by-step build checklist |

---

## Version History

| Version | Description |
|---------|-------------|
| v1 | Planning docs only (no workflow file) |
| v2 | Generic chat trigger, keyword-based routing |
| v3 | Added AI classification, Google Sheets integration |
| **v4** | **Messenger-native webhook, message.mid dedup, style intent (before order), single Send API node, canonical output shape** |
