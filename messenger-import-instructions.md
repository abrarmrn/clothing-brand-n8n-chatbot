# Messenger Import Instructions

## What Is This File?

This guide walks you through importing the `n8n-messenger-product-lookup-v1.json` workflow into n8n and connecting it to Facebook Messenger. No coding required — just follow the steps.

---

## Prerequisites

Before starting, you need:

1. **n8n running** (Cloud or self-hosted) with a public URL (not localhost)
2. **A Facebook Page** you admin (this is what customers message)
3. **A Meta Developer account** at [developers.facebook.com](https://developers.facebook.com)
4. **Your Google Sheets database** already set up (see `google-sheets-structure.md`)
5. **Products data** in the Products tab (at least 5 products with variants)

---

## Step 1: Import the Workflow into n8n

1. Open your n8n editor
2. Click the **three dots menu (⋯)** in the top-right corner (or go to Workflows)
3. Click **"Import from File"**
4. Select the file: `n8n-messenger-product-lookup-v1.json`
5. The workflow will appear on your canvas with all nodes pre-connected
6. **Rename it** if you want (default name: "Messenger Product Lookup v1")

**What you should see:** 11 nodes arranged in two paths:
- Top path: GET Webhook → Verify Token Check → Respond with Challenge / Respond 403
- Bottom path: POST Webhook → Respond 200 OK + Extract Message → Is Valid Message? → Read Products → Search & Format → Send Messenger Reply

---

## Step 2: Connect Your Google Sheets Credential

1. **Double-click** the node called **"Read Products Sheet"**
2. You will see a red warning: "Credentials not set up"
3. Click the **"Credential"** dropdown
4. Select **"Create new credential"** → choose **"Google Sheets OAuth2"**
5. Follow the Google sign-in prompts to authorize your account
6. Once connected, configure the node:
   - **Document:** Select "Clothing Brand Chatbot Database"
   - **Sheet:** Select "Products"
7. Click **"Save"** on the credential, then **"Save"** on the node

**Test it:** Click "Execute Node" on the Read Products Sheet node. You should see your product data appear in the output panel.

---

## Step 3: Set Your Verify Token

The verify token is a secret password that proves Meta is talking to YOUR webhook (not someone else's).

1. **Double-click** the node called **"Verify Token Check"**
2. Look at the second condition: `hub.verify_token` equals `CHANGE_ME_VERIFY_TOKEN`
3. **Change** `CHANGE_ME_VERIFY_TOKEN` to your own secret string
   - Example: `my_clothing_brand_2025_secret`
   - Use letters, numbers, and underscores only
   - Make it at least 20 characters for security
4. **Remember this token** — you will paste the same value into Meta Developer Console later
5. Click **"Save"**

**Important:** The token in n8n and the token in Meta must match EXACTLY (case-sensitive).

---

## Step 4: Create Facebook Page Access Token Credential

The Page Access Token allows n8n to send messages back to customers on your behalf.

### 4a: Create a Meta App

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Click **"My Apps"** → **"Create App"**
3. Select **"Business"** type (or "Other" if Business isn't available)
4. Name it something like "Clothing Brand Chatbot"
5. Add the **"Messenger"** product to your app (click "Set Up" next to Messenger)

### 4b: Generate Page Access Token

1. In your Meta App dashboard, go to **Messenger** → **Settings**
2. Under **"Access Tokens"**, click **"Add or remove Pages"**
3. Select your Facebook Page and grant permissions
4. Click **"Generate Token"** next to your page name
5. **Copy the token** (it's a long string starting with "EAA...")

### 4c: Add Token to n8n

1. In n8n, **double-click** the node called **"Send Messenger Reply"**
2. Click the **"Credential"** dropdown
3. Select **"Create new credential"** → choose **"Header Auth"**
4. Configure it:
   - **Name:** `Facebook Page Access Token`
   - **Header Name:** `Authorization`
   - **Header Value:** `Bearer EAAxxxxxxx...` (paste your full token with "Bearer " prefix)
5. Click **"Save"**

**Alternative method:** If you prefer, you can use the URL parameter approach instead:
- Change the HTTP Request URL to: `https://graph.facebook.com/v18.0/me/messages?access_token=YOUR_TOKEN_HERE`
- But the Header Auth method is more secure.

---

## Step 5: Copy Your Production Webhook URL

1. In n8n, click on the **"POST Webhook (Messages)"** node
2. Look for the **"Production URL"** at the top of the node panel
   - It looks like: `https://your-n8n-domain.com/webhook/messenger-webhook`
   - If you see "Test URL", click the toggle to switch to "Production URL"
3. **Copy this URL** — you will paste it into Meta Developer Console

**The same base URL** will handle both GET (verification) and POST (messages).

**Important:** The workflow must be **ACTIVE** (toggled on) for the production URL to work. Click the toggle in the top-right of the workflow editor to activate it.

---

## Step 6: Configure Meta Developer Webhooks

1. Go to your Meta App dashboard → **Messenger** → **Settings**
2. Scroll down to **"Webhooks"**
3. Click **"Add Callback URL"** (or "Edit" if one exists)
4. Fill in:
   - **Callback URL:** Paste the production URL from Step 5
     (e.g., `https://your-n8n-domain.com/webhook/messenger-webhook`)
   - **Verify Token:** Paste the SAME token you set in Step 3
     (e.g., `my_clothing_brand_2025_secret`)
5. Click **"Verify and Save"**

**What happens:** Meta sends a GET request to your URL with `hub.mode=subscribe`, `hub.verify_token=your_token`, and `hub.challenge=random_string`. Your n8n workflow checks the token, and if it matches, returns the challenge. Meta confirms the webhook is valid.

6. After verification succeeds, **subscribe to events:**
   - Check the box for **"messages"**
   - Check the box for **"messaging_postbacks"** (optional, for future buttons)
7. Click **"Save"**

---

## Step 7: Test It!

1. Open your Facebook Page
2. Click **"Message"** (or open Messenger and send a message to your Page)
3. Type: **"Show me t-shirts"**
4. Wait a few seconds — the bot should reply with matching products from your Google Sheets!

**If it doesn't work:**
- Check n8n execution history (click "Executions" in the sidebar)
- Make sure the workflow is **ACTIVE** (production toggle ON)
- Verify your Google Sheets has product data in the Products tab
- Check that the Page Access Token is correct

---

## How It Works (The Two Webhook Methods Explained)

### Why GET and POST?

Facebook Messenger uses **two different HTTP methods** for different purposes:

| Method | When It Happens | What It Does |
|--------|----------------|--------------|
| **GET** | Only during initial setup | Meta verifies that YOUR server owns this URL. It sends a challenge, and you must echo it back. This only happens once (or when you re-verify). |
| **POST** | Every time a customer messages | Meta sends the actual message data to your webhook. You respond with 200 OK immediately, then process the message and reply. |

### The GET Verification Flow

```
Meta says: "Hey, is this really your server?"
     |
     v
GET request to your URL with:
  hub.mode = "subscribe"
  hub.verify_token = "your_secret_token"
  hub.challenge = "random_number_12345"
     |
     v
Your n8n workflow checks:
  Is mode "subscribe"? ✓
  Does verify_token match mine? ✓
     |
     v
Responds with: "random_number_12345" (the challenge value)
     |
     v
Meta says: "Great, webhook verified!"
```

### The POST Message Flow

```
Customer sends: "Do you have black t-shirts?"
     |
     v
Meta sends POST to your URL with message data
     |
     v
Your n8n workflow:
  1. Responds "200 OK" immediately (so Meta doesn't retry)
  2. Extracts sender PSID and message text
  3. Searches Google Sheets Products tab
  4. Builds a friendly reply
  5. Sends reply back via Messenger Send API
     |
     v
Customer sees the product list in Messenger!
```

---

## Troubleshooting

### "Webhook verification failed" in Meta Console

| Possible Cause | Fix |
|---------------|-----|
| Verify tokens don't match | Check the token in the IF node matches EXACTLY what you typed in Meta |
| Workflow not active | Toggle the workflow ON (top-right of editor) |
| URL is wrong | Make sure you copied the Production URL, not the Test URL |
| n8n not publicly accessible | If self-hosting, make sure your server has a public HTTPS URL |

### Bot doesn't reply to messages

| Possible Cause | Fix |
|---------------|-----|
| Page Access Token expired/wrong | Regenerate the token in Meta Developer Console |
| Google Sheets not connected | Open "Read Products Sheet" node and re-authorize |
| No product data in sheet | Add products to the Products tab |
| Workflow not active | Toggle the workflow ON |
| App not in Live mode | In Meta App settings, toggle from "Development" to "Live" |

### Bot replies to some messages but not others

| Possible Cause | Fix |
|---------------|-----|
| Message has no text (image, sticker, etc.) | Expected — the workflow only handles text messages currently |
| Search terms too short | Words under 3 characters are filtered out |
| Product Active = "NO" | Only Active = "YES" products appear in results |

---

## What's Next?

This workflow only handles **product lookup**. Future versions will add:

- [ ] FAQ answers (Branch 3)
- [ ] Order creation (Branch 4)
- [ ] Order tracking (Branch 5)
- [ ] Human support handoff (Branch 6)
- [ ] Direct reply for greetings (Branch 1)
- [ ] Message classification to route between branches

For now, every message is treated as a product search. The full branching system is documented in `chatbot-branching-logic.md` and will be built incrementally.

---

## Security Reminders

- **Never commit** your Page Access Token to this repository
- **Never share** your verify token publicly
- **Rotate** your Page Access Token if you suspect it's compromised
- All credentials live **inside n8n only** — not in these planning files
- The placeholder values in the JSON (`CHANGE_ME_VERIFY_TOKEN`, `REPLACE_WITH_YOUR_CREDENTIAL_ID`) are intentionally non-functional
