# Build Tasks — Facebook Messenger Chatbot

## Overview

This is your **step-by-step checklist** to build the Messenger chatbot. The project is built **incrementally** — you start with v1 (product lookup) and add features one version at a time.

**Current version: v1 — Product Lookup via Messenger**

| Version | Feature | Estimated Time |
|---------|---------|---------------|
| **v1** | Product Lookup (this guide) | 1-2 hours |
| v2 | FAQ Answers | +30 min |
| v3 | Greeting Replies | +15 min |
| v4 | Order Creation | +60-90 min |
| v5 | Order Tracking | +30 min |
| v6 | Human Support Handoff | +30-40 min |

**Rule:** Complete v1 fully before moving to v2. Test everything at each step!

---

## Phase 1: Set Up Your Accounts

*Time estimate: 20-30 minutes*

### Task 1.1: Set Up n8n

| Detail | Info |
|--------|------|
| **What to do** | Install n8n or create an n8n Cloud account |
| **Why** | n8n receives Messenger messages and sends replies |
| **How** | Option A: [n8n.io](https://n8n.io) Cloud (easiest — gives you a public URL automatically). Option B: Self-host with a public domain + HTTPS. |
| **Done when** | You can open n8n and see the workflow editor |

**Important:** Your n8n must have a **public HTTPS URL** (like `https://your-name.app.n8n.cloud` or `https://n8n.yourdomain.com`). Meta will NOT send webhooks to localhost.

### Task 1.2: Create a Facebook Page

| Detail | Info |
|--------|------|
| **What to do** | Create a Facebook Page for your clothing brand (if you don't have one) |
| **Why** | Customers message your Page — this is where the chatbot lives |
| **How** | Go to facebook.com → Pages → Create New Page → fill in your brand info |
| **Done when** | You have a Facebook Page where customers can click "Message" |

### Task 1.3: Create a Meta Developer Account

| Detail | Info |
|--------|------|
| **What to do** | Sign up at developers.facebook.com |
| **Why** | You need a Meta App to connect Messenger to your n8n webhook |
| **How** | Go to [developers.facebook.com](https://developers.facebook.com) → sign in with your Facebook account → accept developer terms |
| **Done when** | You can access the Meta Developer dashboard |

### Task 1.4: Create a Meta App

| Detail | Info |
|--------|------|
| **What to do** | Create a new app with Messenger product |
| **Why** | The app connects your Page to your webhook |
| **How** | My Apps → Create App → Business type → name it "Clothing Brand Chatbot" → Add Messenger product |
| **Done when** | You have an app with Messenger listed in the left sidebar |

### Task 1.5: Generate Page Access Token

| Detail | Info |
|--------|------|
| **What to do** | Generate a token that lets n8n send messages on behalf of your Page |
| **Why** | Without this token, n8n cannot reply to customers |
| **How** | In your Meta App: Messenger → Settings → Access Tokens → Add/Remove Pages → select your Page → Generate Token |
| **Done when** | You have a long token string starting with "EAA..." copied somewhere safe |

**Security:** Never share this token publicly. Never commit it to this repo.

---

## Phase 2: Set Up Google Sheets Database

*Time estimate: 20-30 minutes*

### Task 2.1: Create Your Google Sheets Spreadsheet

| Detail | Info |
|--------|------|
| **What to do** | Create the spreadsheet with all 6 tabs |
| **Why** | This is your chatbot's database |
| **How** | Follow `google-sheets-structure.md` |
| **Done when** | You have 6 tabs with correct column headers in Row 1 |

**For v1, you only need the Products tab filled with data.** The other tabs can have just headers for now.

### Task 2.2: Add Product Variants to Products Tab

| Detail | Info |
|--------|------|
| **What to do** | Add at least 15-20 product variant rows |
| **Why** | The bot needs products to search through |
| **How** | Each row = one variant (specific size + color). See examples in `google-sheets-structure.md` |
| **Done when** | Products tab has 15-20 rows with all 15 columns filled |

**Required columns for v1:**
Product_ID, Variant_SKU, Product_Name, Category, Description, Price, Currency, Size, Color, Stock_Qty, In_Stock, Image_URL, Product_URL, Search_Keywords, Active

**Tips:**
- Set some variants to `In_Stock = NO` and `Stock_Qty = 0` (for testing)
- Set 1-2 variants to `Active = NO` (should never appear in search results)
- Fill `Search_Keywords` generously — these improve search accuracy

---

## Phase 3: Import the Workflow into n8n

*Time estimate: 15-20 minutes*

### Task 3.1: Import the JSON File

| Detail | Info |
|--------|------|
| **What to do** | Import `n8n-messenger-product-lookup-v1.json` into n8n |
| **Why** | This gives you the complete v1 workflow pre-built |
| **How** | In n8n: menu (⋯) → Import from File → select the JSON file |
| **Done when** | You see 11 nodes on the canvas in two paths (GET and POST) |

### Task 3.2: Connect Google Sheets Credential

| Detail | Info |
|--------|------|
| **What to do** | Set up Google Sheets OAuth2 in the "Read Products Sheet" node |
| **Why** | n8n needs permission to read your spreadsheet |
| **How** | Double-click "Read Products Sheet" → Credential → Create New → Google Sheets OAuth2 → sign in |
| **Done when** | The node shows your "Clothing Brand Chatbot Database" spreadsheet and "Products" sheet |

**Test it:** Click "Execute Node" on the Read Products Sheet node. You should see your product rows in the output.

### Task 3.3: Set Your Verify Token

| Detail | Info |
|--------|------|
| **What to do** | Replace `CHANGE_ME_VERIFY_TOKEN` with your own secret |
| **Why** | Meta uses this to verify you own the webhook URL |
| **How** | Double-click "Verify Token Check" node → change the second condition value from `CHANGE_ME_VERIFY_TOKEN` to your own string (e.g., `my_brand_chatbot_2025_secret`) |
| **Done when** | The IF node shows your custom token |

**Remember this exact token — you'll paste the same value in Meta Developer Console in Phase 4.**

### Task 3.4: Create Facebook Page Access Token Credential

| Detail | Info |
|--------|------|
| **What to do** | Add your Page Access Token to the "Send Messenger Reply" node |
| **Why** | n8n needs this to send messages back to customers |
| **How** | Double-click "Send Messenger Reply" → Credential → Create New → Header Auth → Name: `Facebook Page Access Token` → Header: `Authorization` → Value: `Bearer EAAxxxxxx...` |
| **Done when** | The HTTP Request node has a valid credential attached |

### Task 3.5: Activate the Workflow

| Detail | Info |
|--------|------|
| **What to do** | Toggle the workflow ON (active) |
| **Why** | The production webhook URL only works when the workflow is active |
| **How** | Click the toggle in the top-right corner of the workflow editor |
| **Done when** | Toggle shows "Active" and the workflow is green/highlighted |

---

## Phase 4: Connect Meta Webhooks

*Time estimate: 10-15 minutes*

### Task 4.1: Copy Your Production Webhook URL

| Detail | Info |
|--------|------|
| **What to do** | Get the public URL of your POST Webhook node |
| **Why** | This is what you'll paste into Meta Developer Console |
| **How** | Click the "POST Webhook (Messages)" node → look for "Production URL" at the top → Copy it |
| **Done when** | You have a URL like `https://your-n8n.com/webhook/messenger-webhook` |

**Note:** The GET and POST webhooks share the same path (`messenger-webhook`). Meta uses the same URL for both verification (GET) and messages (POST).

### Task 4.2: Configure Webhook in Meta Developer Console

| Detail | Info |
|--------|------|
| **What to do** | Tell Meta where to send messages |
| **Why** | This connects Messenger to your n8n workflow |
| **How** | Meta App → Messenger → Settings → Webhooks → Add Callback URL |
| **Done when** | You see "Webhook verified" success message |

**Fill in:**
- **Callback URL:** Your production webhook URL from Task 4.1
- **Verify Token:** The exact same token you set in Task 3.3

**What happens:** Meta sends a GET request to your URL. Your workflow checks the token, returns the challenge, and Meta confirms verification.

### Task 4.3: Subscribe to Messenger Events

| Detail | Info |
|--------|------|
| **What to do** | Tell Meta which events to send to your webhook |
| **Why** | Without subscribing, you won't receive messages |
| **How** | After verification, check the box for **"messages"** → click Save |
| **Done when** | The "messages" subscription shows as active |

---

## Phase 5: Test Your Bot

*Time estimate: 10-15 minutes*

### Task 5.1: Send a Test Message

| Detail | Info |
|--------|------|
| **What to do** | Message your Facebook Page from a personal account |
| **Why** | Verify the full flow works end-to-end |
| **How** | Open Messenger → search for your Page → send "Show me t-shirts" |
| **Done when** | The bot replies with matching products from your Google Sheets |

**If the bot doesn't reply:**
1. Check n8n Executions tab — is the workflow being triggered?
2. Is the workflow ACTIVE? (toggle must be ON)
3. Check the execution details — which node failed?
4. Verify Google Sheets is connected (Execute "Read Products Sheet" manually)
5. Verify Page Access Token is correct (check for 401/403 errors in Send node)

### Task 5.2: Run the Full Test Suite

| # | Test | What to Send | Expected Reply |
|---|------|-------------|----------------|
| 1 | Basic search | "Show me t-shirts" | Product list with sizes/colors |
| 2 | Specific query | "Black hoodie in large" | Matching products |
| 3 | No results | "Purple sandals" | "Couldn't find" + category list |
| 4 | Price query | "Under $40" | Products under $40 |
| 5 | Generic message | "Hi" | Helpful prompt (no search terms) |
| 6 | Active filter | Search for product with Active=NO | Should NOT appear |
| 7 | Out of stock | Product with all variants Stock_Qty=0 | Shows with "out of stock" note |
| 8 | Send a photo | (send any image) | No reply (correctly ignored) |

### Task 5.3: Check n8n Execution Logs

| Detail | Info |
|--------|------|
| **What to do** | Review the execution history in n8n |
| **Why** | Confirm data flows correctly through all nodes |
| **How** | Click "Executions" in n8n sidebar → click on recent executions → inspect each node's output |
| **Done when** | You can see: webhook received → message extracted → products read → reply formatted → reply sent |

---

## Phase 6: Go Live with v1

### v1 Go-Live Checklist

- [ ] Workflow is ACTIVE in n8n
- [ ] Meta webhook is verified and "messages" subscription is active
- [ ] Products tab has real product data (not just test data)
- [ ] Search_Keywords column is filled for all products
- [ ] Active = "NO" products don't appear in search
- [ ] Bot replies within a few seconds
- [ ] Replies are under 2000 characters
- [ ] Sending a photo/sticker doesn't crash the bot
- [ ] Page Access Token is valid (not expired)
- [ ] Your n8n instance has a stable public URL

### After Go-Live

- **Daily:** Check n8n execution logs for errors
- **Weekly:** Review what customers are searching for (check execution logs)
- **Weekly:** Add Search_Keywords for products that aren't being found
- **As needed:** Add new products to the Products tab

---

## Future Phases (v2-v6)

These will be built in order after v1 is stable:

### v2: Add FAQ Answers (+30 min)
- Add FAQ data to the FAQ tab in Google Sheets
- Add a Message Classifier node (Switch or AI) between "Is Valid Message?" and product search
- Add FAQ Search + Reply branch
- Connect reply to existing Send Messenger Reply pattern

### v3: Add Greeting Replies (+15 min)
- Add greeting detection to the classifier
- Add a simple reply node for hi/thanks/bye
- Optional: look up PSID in Customers tab for personalization

### v4: Add Order Creation (+60-90 min)
- Add order detection to classifier
- Build multi-step conversation (product → size → color → name → email → address → payment)
- Add variant validation against Products tab
- Save to Orders tab + Customers tab
- Send confirmation via Messenger

### v5: Add Order Tracking (+30 min)
- Add tracking detection to classifier
- Look up customer by PSID → get Customer_ID → search Orders/Tracking tabs
- Reply with order status or tracking info

### v6: Add Human Handoff (+30-40 min)
- Add support detection to classifier
- Add automatic escalation after 3 fallbacks
- Create Support_Tickets row with full context
- Reply with ticket number and expected response time

---

## Quick Reference: v1 Architecture

```
[GET Webhook] → [Token Check] → [Challenge/403]

[POST Webhook] → [200 OK] + [Extract Message] → [Valid?] → [Read Products] → [Search & Format] → [Send Reply]
```

**11 nodes total. 2 credentials (Google Sheets + Page Access Token). 1 placeholder to change (verify token).**

---

## Troubleshooting Quick Reference

| Problem | Most Likely Cause | Fix |
|---------|------------------|-----|
| Meta says "webhook verification failed" | Token mismatch or workflow not active | Check token in IF node matches Meta exactly; activate workflow |
| Bot doesn't reply | Page Access Token wrong/expired | Regenerate token in Meta App |
| Bot replies "I'd love to help you find something" for everything | Search terms not matching any products | Add more Search_Keywords to Products tab |
| Bot replies with wrong products | Search_Keywords too broad | Make keywords more specific |
| Error in execution log: "Google Sheets" node | Credential expired or sheet renamed | Re-authorize Google Sheets credential |
| Duplicate replies | Meta retried because 200 wasn't fast enough | Should not happen with parallel 200 response; check n8n load |
| Works in test but not from real Messenger | App in Development mode | Switch Meta App to Live mode |
