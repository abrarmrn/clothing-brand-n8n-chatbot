# Messenger Hybrid v3 — Import Instructions

## What's New in v3?

v3 fixes the AI fallback bug from v2 and adds major features:

| Fix/Feature | Details |
|-------------|---------|
| **AI fallback fixed** | Prepare Reply now reads `.output` field (LangChain format) correctly |
| **Language detection** | Auto-detects Bangla, Roman Bangla (Banglish), or English |
| **Reply in customer's language** | Bot responds in the same language style |
| **Telegram human support alert** | Sends notification to your Telegram when human needed |
| **Consistent output** | Every branch outputs `senderPsid` + `replyText` reliably |
| **try/catch error handling** | Code nodes won't crash silently |
| **BD clothing brand context** | AI knows it's a Bangladeshi brand, mentions Eid/occasions |

---

## Prerequisites

1. **n8n running** with public HTTPS URL
2. **v1 and v2 workflows saved** (do NOT delete them)
3. **Google Sheets** with Products + FAQ data populated
4. **Facebook Page Access Token** (same as v1/v2)
5. **OpenAI API key** (for AI fallback)
6. **Telegram Bot Token + Chat ID** (for human support alerts)

---

## Step 1: Import v3 as a New Workflow

1. In n8n, click **menu (⋯)** → **"Import from File"**
2. Select: `n8n-messenger-hybrid-ai-v3.json`
3. This creates a **new workflow** — v1 and v2 are untouched
4. You should see **24 nodes** on the canvas

---

## Step 2: Connect Google Sheets Credential

Connect to **two** nodes:

1. **"Read FAQ Sheet"** → Credential: Google Sheets OAuth2
   - Document: "Clothing Brand Chatbot Database"
   - Sheet: "FAQ"

2. **"Read Products Sheet"** → Credential: same
   - Document: "Clothing Brand Chatbot Database"
   - Sheet: "Products"

**Test:** Click "Execute Node" on each — you should see your data.

---

## Step 3: Connect Facebook Header Auth Credential

1. **"Send Messenger Reply"** → Credential: Header Auth
   - Select your existing "Facebook Page Access Token"
   - Or create: Header Name = `Authorization`, Value = `Bearer EAAxxxxx...`

---

## Step 4: Connect AI Chat Model Credential

1. **"Chat Model"** node → Credential: OpenAI API
   - Select existing or create new with your API key
   - Model is set to `gpt-4o-mini` (cheapest)

---

## Step 5: Set Up Telegram Bot (New in v3)

### Create a Telegram Bot:
1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. Send `/newbot` and follow prompts
3. Copy the **Bot Token** (looks like `123456789:ABCdefGHIjklMNO...`)

### Get your Chat ID:
1. Create a Telegram group for support alerts
2. Add your bot to the group
3. Send a message in the group
4. Visit: `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
5. Find `"chat":{"id":-123456789}` — that number is your Chat ID

### Configure in n8n:
1. **"Send Telegram Alert"** node → edit the URL:
   - Replace `TELEGRAM_BOT_TOKEN_PLACEHOLDER` with your bot token
2. In the JSON body, replace `TELEGRAM_CHAT_ID_PLACEHOLDER` with your chat ID

**No Telegram yet?** The bot still works fine — alerts just won't send. The Messenger reply still goes through.

---

## Step 6: Change Verify Token

1. **"Verify Token Check"** → change `CHANGE_ME_VERIFY_TOKEN`
2. Use the **same token** as v1/v2 if reusing the same webhook

---

## Step 7: Switch from v2 to v3

1. **Deactivate v2** (toggle OFF)
2. **Activate v3** (toggle ON)
3. Same webhook path (`messenger-webhook`) — no Meta changes needed

---

## Testing Each Route

### Test 1: Static Greeting (NO AI)

| Send | Expected | AI? |
|------|----------|-----|
| "Hi" | English welcome | No |
| "Assalamualaikum" | Bangla/Banglish welcome | No |
| "ধন্যবাদ" | Bangla thanks reply | No |
| "Bye" | Goodbye | No |

### Test 2: FAQ Match (NO AI)

| Send | Expected | AI? |
|------|----------|-----|
| "Return policy ki?" | FAQ answer | No |
| "Shipping koto din lage?" | FAQ answer | No |
| "Payment method ki ki?" | FAQ answer | No |

### Test 3: Product Lookup (NO AI)

| Send | Expected | AI? |
|------|----------|-----|
| "T-shirt dekhao" | Product list with ৳ prices | No |
| "Black hoodie" | Matching products | No |
| "2000 er niche ki ache?" | Filtered products | No |

### Test 4: AI Fallback (AI Used)

| Send | Expected | AI? |
|------|----------|-----|
| "Eid er jonno stylish kichu chai under 2000 BDT" | Style suggestion in Banglish | **Yes** |
| "Black jeans er sathe ki porbo?" | Outfit advice | **Yes** |
| "Cotton naki polyester better summer e?" | Fabric info | **Yes** |

**Verify fix:** The reply should be a helpful human-like answer — NOT "I apologize, I could not process..."

### Test 5: Human Support + Telegram Alert

| Send | Expected | Telegram? |
|------|----------|-----------|
| "I want to talk to a human" | Support message | ✅ Alert sent |
| "THIS IS TERRIBLE" | Support message | ✅ Alert sent |
| "Refund chai" | Support message | ✅ Alert sent |

**Check Telegram:** You should receive a notification in your support group.

---

## How v3 Fixes the AI Bug

**The problem in v2:**
- AI Agent (LangChain) outputs its response in a field called `.output`
- Prepare Reply was looking for `.replyText`, `.text`, `.response` — but NOT `.output`
- Result: empty reply → fallback error message

**The fix in v3:**
```javascript
// Priority order for reading reply:
// 1. data.replyText (from rule-based branches)
// 2. data.output (from AI Agent - LangChain format) ← THIS WAS MISSING
// 3. data.text (alternative AI format)
// 4. data.response (another alternative)
// 5. Polite fallback if all empty
```

---

## Keeping v1 and v2 as Backups

- **Never delete** v1 or v2 workflows from n8n
- **Never delete** their JSON files from this repo
- If v3 has issues: deactivate v3, activate v2 (instant rollback)
- v1 is your "nuclear option" — always works for basic product lookup

---

## Language Detection Examples

| Customer Types | Detected As | Bot Replies In |
|---------------|-------------|---------------|
| "আমার একটা t-shirt দরকার" | Bangla | বাংলা |
| "Ami ekta black hoodie chai" | Banglish | Roman Bangla |
| "Show me jeans under 1500" | English | English |
| "Eid er dress ache?" | Banglish | Roman Bangla |
| "কালো জিন্স আছে?" | Bangla | বাংলা |

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| AI still says "I apologize..." | Old Prepare Reply code | Re-import v3 JSON (don't edit v2) |
| Telegram alert not sending | Token/Chat ID wrong | Check URL in Send Telegram Alert node |
| Bot replies in wrong language | Short message misdetected | Language detection uses word lists — add missing Banglish words to router |
| FAQ not matching | Keywords too generic | Add more specific keywords in FAQ sheet |
| Products show ৳ but wrong currency | Currency column empty | Fill Currency column in Products tab (use BDT) |

---

## Cost Summary (Same as v2)

| Route | AI Used? | Cost |
|-------|----------|------|
| Greetings | No | Free |
| FAQ match | No | Free |
| Product search | No | Free |
| Tracking | No | Free |
| Support/handoff | No | Free |
| Order intent | No | Free |
| Complex questions (AI) | Yes | ~$0.001-0.01 per message |
| Telegram alerts | No | Free |
