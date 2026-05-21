# Messenger Hybrid v5 — Import & Setup Instructions

## What's New in v5?

v5 is a **stable upgrade from v3** — preserving the working AI Agent + Chat Model pattern while adding major intelligence features:

| Feature | v3 | v5 |
|---------|----|----|
| Style/occasion routing | ❌ (goes to AI fallback) | ✅ Dedicated branch BEFORE order intent |
| Weak order keyword fix | ❌ ("i want", "chai" trigger order) | ✅ Only strong buy signals trigger order |
| Message deduplication | ❌ (double replies possible) | ✅ message.mid dedup via workflow static data |
| Customer memory | ❌ | ✅ Per-PSID memory in workflow static data |
| Typo/Banglish normalization | ❌ | ✅ 20+ common typos auto-corrected |
| Size guide support | ❌ | ✅ Height/weight → size recommendation |
| Objection handling | ❌ | ✅ "dam beshi", "last price" handled warmly |
| Budget-aware search | ❌ | ✅ Detects "under 2000 BDT" and filters |
| Canonical output contract | ❌ (inconsistent) | ✅ Every branch outputs same shape |
| Entity extraction | ❌ | ✅ Budget, size, color, occasion auto-detected |
| Improved product ranking | Basic keyword match | ✅ Budget filter, featured boost, stock priority |
| AI Agent input fix | Uses $json.messageText (bug) | ✅ Uses customerMessage + context |
| Graph API version | v18.0 | ✅ v25.0 |
| Smart follow-up questions | ❌ | ✅ One question per reply |

---

## What Was NOT Changed from v3

| Preserved | Why |
|-----------|-----|
| AI Agent node type (`@n8n/n8n-nodes-langchain.agent` v1.7) | Working, stable |
| Chat Model node (`@n8n/n8n-nodes-langchain.lmChatOpenAi` v1) | Working, gpt-4o-mini |
| AI Agent + Chat Model connection via `ai_languageModel` | Required by n8n |
| Single "Send Messenger Reply" HTTP node | One final customer send |
| Webhook path `messenger-webhook` | No Meta reconfiguration needed |
| Telegram alert structure | Same pattern, enhanced content |
| httpHeaderAuth for Facebook | Same credential type |
| Language detection (Bangla/Banglish/English) | Enhanced with more words |

---

## Prerequisites

1. **n8n Cloud** (or self-hosted v1.30+) with public HTTPS URL
2. **v1, v2, v3 workflows saved** (do NOT delete them)
3. **Google Sheets** with required tabs (see below)
4. **Facebook Page Access Token** (same as v3)
5. **OpenAI API key** (for AI fallback — same as v3)
6. **Telegram Bot Token + Chat ID** (for alerts — same as v3)

---

## Google Sheets Required Tabs

### Required (workflow will fail without these):

| Tab | Purpose | Minimum Data |
|-----|---------|--------------|
| **FAQ** | FAQ answers | 10+ rows with Keywords + Answer columns |
| **Products** | Product catalog | 5+ rows with Product_ID, Product_Name, Price, Active |

### Optional (workflow handles gracefully if empty/missing):

| Tab | Purpose | v5 Feature |
|-----|---------|-----------|
| **Size_Guide** | Size recommendations | Height/weight → size mapping |
| **Sales_Objections** | Objection responses | "dam beshi", "discount" replies |

### Future/Optional (documented but not required):

| Tab | Purpose | Notes |
|-----|---------|-------|
| Conversation_Memory | Persistent memory | v5 uses workflow static data instead |
| Bot_Logs | Analytics | Planned for v6 |
| Lead_Followups | Warm lead tracking | Planned for v6 |

---

## Step 1: Import v5 as a New Workflow

1. In n8n, click **menu (⋯)** → **"Import from File"**
2. Select: `n8n-messenger-hybrid-ai-v5.json`
3. This creates a **new workflow** — v1/v2/v3 are untouched
4. You should see **25 nodes** on the canvas

---

## Step 2: Connect Google Sheets Credential

Connect to **four** Google Sheets nodes:

| Node | Sheet Tab |
|------|-----------|
| "Read FAQ Sheet" | FAQ |
| "Read Products Sheet" | Products |
| "Read Size Guide" | Size_Guide |
| "Read Sales Objections" | Sales_Objections |

**For each node:**
1. Double-click the node
2. Select your Google Sheets OAuth2 credential
3. Set Document ID (your "Clothing Brand Chatbot Database" spreadsheet)
4. Set Sheet name to the correct tab

**If Size_Guide or Sales_Objections tabs don't exist yet:** Create empty tabs with just header rows. The workflow handles empty sheets gracefully.

---

## Step 3: Connect Facebook Header Auth Credential

1. Double-click **"Send Messenger Reply"** node
2. Select Credential: Header Auth
3. Use your existing credential (same as v3):
   - Header Name: `Authorization`
   - Header Value: `Bearer EAAxxxxx...` (your Page Access Token)

**Graph API upgraded to v25.0** — no token changes needed.

---

## Step 4: Connect AI Chat Model Credential

1. Double-click **"Chat Model"** node
2. Select Credential: OpenAI API
3. Use your existing credential (same as v3)
4. Model is pre-set to `gpt-4o-mini` (cheapest)

**Verify connection:** The Chat Model node must show a line connecting to "AI Agent (Fallback Only)" via the AI input.

---

## Step 5: Set Up Telegram Bot (Same as v3)

1. Double-click **"Send Telegram Alert"** node
2. In the URL field, replace `TELEGRAM_BOT_TOKEN_PLACEHOLDER` with your bot token
3. In the JSON body, replace `TELEGRAM_CHAT_ID_PLACEHOLDER` with your chat ID

**No Telegram yet?** The bot still works — alerts just won't send.

---

## Step 6: Change Verify Token

1. Double-click **"Verify Token Check"** node
2. Change `CHANGE_ME_VERIFY_TOKEN` to your verify token (same as v3)

---

## Step 7: Add Size Guide Data (Optional)

Create a **Size_Guide** tab with these columns:

| Column | Example |
|--------|---------|
| Category | T-Shirts |
| Size | M |
| Min_Height_Feet | 5 |
| Max_Height_Feet | 5 |
| Min_Weight_Kg | 55 |
| Max_Weight_Kg | 70 |
| Chest | 38 |
| Length | 27 |
| Fit_Type | Regular |
| Notes | Good for average build |

---

## Step 8: Add Sales Objections Data (Optional)

Create a **Sales_Objections** tab with these columns:

| Column | Example |
|--------|---------|
| Objection_ID | OBJ-001 |
| Keywords | dam beshi, expensive, price high |
| Response | Amader products premium quality fabric use kora. Budget friendly options o ache — ki range e khujchen? |
| Active | YES |

---

## Step 9: Add Optional Product Columns

v5 supports these **optional** columns in your Products tab:

| Column | Purpose |
|--------|---------|
| Occasion_Tags | eid, party, casual, office, wedding |
| Style_Tags | stylish, premium, minimal, oversized |
| Is_Featured | YES/NO — gets slight ranking boost |
| Stock_Qty | Number — shows "Limited stock!" warning if ≤3 |

These are optional. If missing, the workflow works fine without them.

---

## Step 10: Switch from v3 to v5

1. **Deactivate v3** (toggle OFF)
2. **Activate v5** (toggle ON)
3. Same webhook path (`messenger-webhook`) — no Meta changes needed

---

## How to Test Without Production Activation

1. Keep v3 ACTIVE (production)
2. Open v5 workflow (keep INACTIVE)
3. Click **"Test Workflow"** button
4. In a new tab, use curl/Postman to send a test POST to your v5 webhook URL:

```bash
curl -X POST "https://your-n8n.app.n8n.cloud/webhook-test/messenger-webhook" \
  -H "Content-Type: application/json" \
  -d '{
    "object": "page",
    "entry": [{
      "messaging": [{
        "sender": {"id": "TEST_PSID_123"},
        "recipient": {"id": "PAGE_ID"},
        "timestamp": 1234567890,
        "message": {
          "mid": "test_mid_001",
          "text": "Hi"
        }
      }]
    }]
  }'
```

5. Check execution results in n8n — verify correct routing and reply text

---

## How to Confirm AI Agent + Chat Model Are Working

1. Send a test message that should trigger AI fallback:
   - "What should I wear to a beach wedding in summer?"
   - (This shouldn't match any FAQ, product, or rule)
2. Check execution logs:
   - "AI Needed?" node should take the TRUE path
   - "AI Agent (Fallback Only)" should execute (green)
   - "Chat Model" should show as executed
   - "Prepare Final Reply" should have a non-empty `replyText`

---

## How to Confirm Deduplication Works

1. Send the same test payload with the same `message.mid` twice rapidly
2. First execution: processes normally, sends reply
3. Second execution: "Extract + Dedup" returns `skipSend: true`
4. "Is Valid + Not Duplicate?" takes FALSE path — no reply sent

---

## Rollback Plan

1. **Deactivate v5** (toggle OFF)
2. **Activate v3** (toggle ON)
3. Messages flow through v3 immediately
4. Debug v5 using execution logs
5. Once fixed, switch back

**Never delete v3.** It's your safety net.

---

## Credential Summary

| Credential | Type | Used By | Placeholder |
|-----------|------|---------|-------------|
| Google Sheets OAuth2 | googleSheetsOAuth2Api | 4 sheet read nodes | GOOGLE_SHEETS_CREDENTIAL_PLACEHOLDER |
| Facebook Page Token | httpHeaderAuth | Send Messenger Reply | FACEBOOK_PAGE_ACCESS_TOKEN_PLACEHOLDER |
| OpenAI API | openAiApi | Chat Model | AI_CHAT_MODEL_CREDENTIAL_PLACEHOLDER |
| Telegram Bot Token | (in URL) | Send Telegram Alert | TELEGRAM_BOT_TOKEN_PLACEHOLDER |
| Telegram Chat ID | (in body) | Send Telegram Alert | TELEGRAM_CHAT_ID_PLACEHOLDER |
| Verify Token | (in IF condition) | Verify Token Check | CHANGE_ME_VERIFY_TOKEN |

---

## Cost Summary

| Route | AI Used? | Cost |
|-------|----------|------|
| Greetings/thanks/bye | No | Free |
| Human support detection | No | Free |
| Order tracking | No | Free |
| Style/occasion recommendation (from sheets) | No | Free |
| Size recommendation | No | Free |
| Objection handling | No | Free |
| Strong order intent | No | Free |
| FAQ match | No | Free |
| Product search | No | Free |
| Complex questions (AI fallback) | Yes | ~$0.001-0.01 per msg |
| Telegram alerts | No | Free |

**Expected:** 80-95% of messages handled without AI (improvement from v3's 70-90%).

---

## Architecture Diagram

```
POST Webhook ──→ Respond 200 OK (Meta ACK)
      │
      └──→ Extract + Dedup
                │
          Is Valid + Not Duplicate?
                │ TRUE
          Load Memory + Normalize
                │
          Read FAQ Sheet ──→ Read Products ──→ Read Size Guide ──→ Read Objections
                                                                        │
                                                              Decision Engine v5
                                                                   │
                                                             AI Needed?
                                                            /          \
                                                    TRUE /              \ FALSE
                                              AI Agent                    │
                                                    \                    │
                                                     └──→ Prepare Final Reply
                                                                │
                                                      Update Memory + Log
                                                                │
                                                          Should Send?
                                                         /            \
                                                  TRUE /                \ FALSE (end)
                                          Send Messenger Reply     Needs Telegram Alert?
                                                │                      │ TRUE
                                          Mark Processed         Prepare Telegram Alert
                                                                       │
                                                                Send Telegram Alert
```

---

## v6 Planned Features (NOT in v5)

- AI Vision (image understanding)
- Abandoned cart messages
- Campaign automation
- Google Sheets-based persistent memory (replacing staticData)
- Full analytics dashboard
- Multi-product order in one conversation
- Payment gateway integration
