# v7 Import Instructions - Messenger Hybrid AI Chatbot

## Overview

This guide walks you through importing the v7 workflow into n8n and connecting your existing credentials.

**Time needed:** 15-25 minutes  
**Prerequisite:** Working v6 workflow (we'll reuse your credentials)

---

## Step 1: Create Conversation_Memory Sheet Tab

Before importing, add a new tab to your Google Sheets database:

1. Open your **Clothing Brand Chatbot Database** spreadsheet
2. Click **"+"** at the bottom to add a new tab
3. Name it exactly: `Conversation_Memory`
4. Add these column headers in Row 1:

| A | B | C | D | E | F | G | H | I | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| senderPsid | lastProductId | lastProductName | lastCategory | lastSize | lastColor | lastBudget | lastOccasion | lastIntent | lastUserMessage | lastBotReply | lastShownProducts | updatedAt |

**Important:** Column names must match exactly (case-sensitive).

---

## Step 2: Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** page
3. Click **"..."** menu (top right) → **Import from File**
4. Select `n8n-messenger-hybrid-ai-v7.json`
5. The workflow will open on your canvas with 24 nodes

**Do NOT delete your v6 workflow yet.** Keep it as backup until v7 is tested.

---

## Step 3: Connect Google Sheets Credential

You have **4 Google Sheets nodes** that need credentials:

| Node Name | Sheet Tab |
|-----------|-----------|
| Load Products Sheet | Products |
| Load FAQ Sheet | FAQ |
| Load Memory Sheet | Conversation_Memory |
| Save Memory | Conversation_Memory |

For each:
1. Double-click the node
2. Click **Credential** dropdown
3. Select your existing **Google Sheets OAuth2** credential (same one from v6)
4. Set **Document** → select your "Clothing Brand Chatbot Database" spreadsheet
5. Verify the **Sheet Name** matches the tab name
6. Click **Save**

---

## Step 4: Connect OpenAI Credential

You have **2 OpenAI nodes** that need credentials:

| Node Name | Purpose |
|-----------|---------|
| Transcribe Audio | Whisper speech-to-text |
| AI Generate Reply | GPT-4.1-mini chat completion |

For each:
1. Double-click the node
2. Click **Credential** dropdown
3. Select your existing **OpenAI API** credential (same one from v6)
4. Click **Save**

**Model setting:** The AI Generate Reply node uses `gpt-4.1-mini`. If this model isn't available on your plan, change to `gpt-4o-mini`.

---

## Step 5: Configure Messenger Page Token

The **Send Messenger Reply** node sends replies via Facebook Graph API.

1. Double-click **"Send Messenger Reply"** node
2. Under **Authentication** → **Generic Credential Type** → **Query Auth**
3. Create or select a credential with:
   - **Name:** `access_token`
   - **Value:** Your Facebook Page Access Token

**How to get your Page Access Token:**
- Facebook Developer Portal → Your App → Messenger Settings → Token Generation
- Select your Page → Copy the token

If you already have this configured in v6 (HTTP Header Auth or similar), recreate it as **Query Auth** with parameter name `access_token`.

---

## Step 6: Configure Telegram Alert (Optional)

The **Telegram Alert** node sends notifications for orders and support issues.

1. Double-click **"Telegram Alert"** node
2. In the URL field, replace `YOUR_BOT_TOKEN` with your actual Telegram bot token
3. In the JSON body, replace `YOUR_TELEGRAM_CHAT_ID` with your group/channel chat ID

If you don't use Telegram alerts, you can **disable** this node (right-click → Deactivate) without affecting the main flow.

---

## Step 7: Set Webhook Verify Token

1. Double-click **"Verify Logic"** node
2. In the code, find: `const VERIFY_TOKEN = 'YOUR_VERIFY_TOKEN_HERE';`
3. Replace `YOUR_VERIFY_TOKEN_HERE` with your actual Facebook webhook verify token (same one from v6)

---

## Step 8: Activate and Test

### 8.1 Activate the Workflow
1. Toggle the workflow to **Active** (top right switch)
2. Copy the webhook URL shown on the **Messenger Webhook** node

### 8.2 Update Facebook Webhook URL (if URL changed)
If your n8n URL changed or you're on a different workflow:
1. Go to Facebook Developer Portal → Your App → Webhooks
2. Update the Callback URL to your new webhook URL
3. Keep the same Verify Token

### 8.3 Run Quick Tests

Send these messages from Messenger to your page:

| # | Send | Expected Behavior |
|---|------|-------------------|
| 1 | "hi" | Short Bangla greeting |
| 2 | "ami t shirt khujtechi" | Shows available T-shirts from Products sheet |
| 3 | "only ei tshirt available?" | Refers to previous T-shirt, NOT "EI" category |
| 4 | "okay" | Continues context (asks size/color), no greeting reset |
| 5 | "M size er ki available?" | Checks M size for the previously discussed product |
| 6 | "order korte chai" | Asks only for missing info (phone, address) |
| 7 | Send a voice message | Transcribes and replies in text |

### 8.4 Check Memory Sheet
After testing, open **Conversation_Memory** tab in Google Sheets. You should see a row with your test sender's PSID and the conversation context.

---

## Step 9: Deactivate v6

Once v7 passes all tests:
1. Open your v6 workflow
2. Toggle it to **Inactive**
3. Keep it saved (don't delete) as a backup

---

## Node Flow Diagram

```
[Messenger Webhook POST] ──→ [Respond 200] (immediate)
         │
         ▼
  [Extract Message]
         │
         ▼
   [Has Message?] ──No──→ (stop)
         │ Yes
         ▼
    [Dedup Check] ──duplicate──→ (stop)
         │ new
         ▼
    [Is Audio?]
     /        \
   Yes         No
    │           │
    ▼           ▼
[Download Audio]   [Load Products Sheet]
    │              [Load FAQ Sheet]
    ▼              [Load Memory Sheet]
[Transcribe Audio]      │
    │                   │
    ▼                   │
[Set Transcription]     │
    │                   │
    ▼                   ▼
  [Load Sheets] ──→ [Prepare AI Context]
                          │
                          ▼
                  [AI Generate Reply] (gpt-4.1-mini)
                          │
                          ▼
                  [Validate AI Output]
                          │
                          ▼
                  [Prepare Memory Update]
                     /           \
                    ▼             ▼
          [Prepare Final Reply]  [Save Memory]
                    │
                    ▼
          [Send Messenger Reply]
                    │
                    ▼
             [Needs Alert?]
               /        \
             Yes         No
              │           │
              ▼           ▼
      [Telegram Alert]  (done)
```

---

## Credential Summary

| Credential Type | Used By | How to Get |
|----------------|---------|------------|
| Google Sheets OAuth2 | Load Products, Load FAQ, Load Memory, Save Memory | Reuse from v6 |
| OpenAI API | Transcribe Audio, AI Generate Reply | Reuse from v6 |
| HTTP Query Auth (access_token) | Send Messenger Reply | Facebook Page Token |
| Telegram Bot Token | Telegram Alert | @BotFather on Telegram |

---

## Troubleshooting

### "No data found" from sheets
- Check spreadsheet ID is correct in each Google Sheets node
- Verify tab names match exactly (case-sensitive)
- Make sure Conversation_Memory tab has headers in Row 1

### Bot not responding
- Check workflow is Active
- Verify webhook URL is correctly set in Facebook Developer Portal
- Check n8n execution log for errors (Executions tab)

### Bot gives generic reply
- Check OpenAI credential is working
- Look at AI Generate Reply node output in execution log
- Verify Products sheet has data with In_Stock = "YES"

### Memory not persisting
- Check Save Memory node has correct spreadsheet selected
- Verify "appendOrUpdate" operation is set with matchingColumns = senderPsid
- Check Conversation_Memory tab has correct column headers

### Voice messages not working
- Verify OpenAI credential works for audio transcription
- Check Download Audio node can access the Facebook audio URL
- Facebook audio URLs require the page access token - may need auth header

---

## What Changed from v6 to v7

| Aspect | v6 | v7 |
|--------|----|----|
| Architecture | Conversational Agent + Window Buffer Memory | Code nodes + OpenAI Chat Model + Sheet-based memory |
| Memory | n8n Window Buffer (volatile, lost on restart) | Google Sheet per-customer (persistent) |
| Context handling | Agent tries to remember via prompt | Explicit memory load → AI context injection |
| Product lookup | Agent uses tools or gets full sheet | Code node prefilters top 5 relevant products |
| Pronouns (ei/eta) | Often misunderstood | Memory provides last product, AI instructed properly |
| Reply style | Often long English | Forced short Bangla/Banglish via prompt + validation |
| Output format | Free text | Structured JSON with validation |
| Cost control | Full sheet in context every time | Only top 5 products + 3 FAQ + compressed memory |
| Order flow | Fragile rule-based branches | AI handles with strong/weak intent rules |
| Alerts | Separate path | Integrated: only triggers on support/orders |
