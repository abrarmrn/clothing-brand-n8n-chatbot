# Messenger Hybrid v2 — Import Instructions

## What Is This?

This guide walks you through importing the **Hybrid AI v2** workflow (`n8n-messenger-hybrid-ai-v2.json`) into n8n. This version is smarter than v1 — it answers most messages from Google Sheets rules **without using AI**, and only calls the AI Agent for complex questions that cannot be answered from your data.

**Cost-saving design:**
- Greetings, FAQ, product lookup, tracking, support → **No AI cost**
- Only unclear/complex questions → **AI Agent runs (costs money)**

---

## Prerequisites

Before starting, you need:

1. **n8n running** with a public HTTPS URL (n8n Cloud or self-hosted)
2. **v1 already working** (keep it as your backup — do NOT delete it)
3. **Google Sheets database** set up with Products and FAQ data populated
4. **Facebook Page Access Token** (same one from v1)
5. **An AI API key** (OpenAI or compatible) — only needed for the fallback AI Agent

---

## Step 1: Import the v2 Workflow

1. In n8n, click the **menu (⋯)** → **"Import from File"**
2. Select: `n8n-messenger-hybrid-ai-v2.json`
3. The workflow appears as a **new, separate workflow** (v1 is untouched)
4. Rename if desired (default: "Messenger Hybrid AI v2")

**What you should see:** 17 nodes in two paths:
- Top path: GET Webhook → Verify → Challenge/403
- Bottom path: POST Webhook → 200 OK + Extract → Valid? → Hybrid Router → Sheet Lookup → FAQ+Product Search → AI Needed? → AI Agent → Prepare Reply → Send Reply
- Side branches: Tracking, Support, Order handlers → Prepare Reply → Send Reply

---

## Step 2: Connect Google Sheets Credential

You need to connect Google Sheets to **three** nodes:

1. **Double-click** "Read FAQ Sheet"
   - Credential → select your existing Google Sheets OAuth2 (or create new)
   - Document: "Clothing Brand Chatbot Database"
   - Sheet: "FAQ"
   - Save

2. **Double-click** "Read Products Sheet"
   - Credential → same Google Sheets credential
   - Document: "Clothing Brand Chatbot Database"
   - Sheet: "Products"
   - Save

3. (Future) If you add Tracking/Orders lookup, connect those sheets too

**Test it:** Click "Execute Node" on each sheet node — you should see your data.

---

## Step 3: Connect Facebook Page Access Token

1. **Double-click** "Send Messenger Reply"
2. Credential → select **"Facebook Page Access Token"** (same Header Auth from v1)
   - If not available: Create New → Header Auth
   - Name: `Facebook Page Access Token`
   - Header Name: `Authorization`
   - Header Value: `Bearer EAAxxxxxx...` (your Page Access Token)
3. Save

---

## Step 4: Connect AI Chat Model Credential

This is the **new credential** in v2 (not needed in v1).

1. **Double-click** "Chat Model" node (connected below AI Agent)
2. Credential → Create New → **"OpenAI API"**
   - Name: `OpenAI account`
   - API Key: paste your OpenAI API key
3. Save

**Model selection:** The default is `gpt-4o-mini` (cheapest). You can change it to:
- `gpt-4o-mini` — cheapest, good for simple support replies
- `gpt-4o` — smarter, more expensive
- `gpt-3.5-turbo` — older but cheap

**No OpenAI key?** The workflow still works for greetings, FAQ, products, tracking, and support — those never call AI. Only truly unrecognized messages will fail without an AI key.

---

## Step 5: Change the Verify Token

1. **Double-click** "Verify Token Check" (IF node)
2. Change `CHANGE_ME_VERIFY_TOKEN` to your own secret
   - Use the **same token** as your v1 if you want to reuse the Meta webhook
   - Or use a different token if you set up a new webhook URL
3. Save

---

## Step 6: Activate and Connect to Meta

### Option A: Replace v1 with v2 (same webhook URL)

1. **Deactivate v1** (toggle it OFF)
2. **Activate v2** (toggle it ON)
3. Since v2 uses the same path (`messenger-webhook`), Meta will automatically send to v2 now
4. No changes needed in Meta Developer Console

### Option B: Run v2 on a different path (keep v1 active for safety)

1. In v2, change the webhook path in both GET and POST nodes to `messenger-webhook-v2`
2. Activate v2
3. In Meta Developer Console, update the Callback URL to the v2 production URL
4. Re-verify the webhook with your token
5. Once v2 is confirmed working, deactivate v1

**Recommended:** Use Option A (simpler). Keep v1 workflow saved but inactive as backup.

---

## Step 7: Test Each Route

### Test 1: Static Direct Reply (NO AI used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "Hi" | "Hello! Welcome to our store..." | No |
| "Thanks" | "You are welcome!..." | No |
| "Bye" | "Goodbye! Thanks for chatting..." | No |
| "Assalamualaikum" | "Hello! Welcome to our store..." | No |

**How to verify:** Check n8n execution logs — the "AI Agent (Fallback Only)" node should show "not executed" (greyed out).

### Test 2: FAQ Match (NO AI used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "What is your return policy?" | Answer from FAQ tab | No |
| "How long does shipping take?" | Answer from FAQ tab | No |
| "Do you accept PayPal?" | Answer from FAQ tab | No |

**How to verify:** In execution logs, check "FAQ + Product Search (No AI)" output shows `aiUsed: false`.

### Test 3: Product Lookup (NO AI used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "Show me t-shirts" | Product list with sizes/colors | No |
| "Black hoodie in large" | Matching products | No |
| "What do you have under $40?" | Filtered products | No |

**How to verify:** Output of "FAQ + Product Search (No AI)" shows `route: 'product_match'` and `aiUsed: false`.

### Test 4: Tracking Intent (NO AI used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "Where is my order?" | Ask for order number | No |
| "Track ORD-20250120-001" | Acknowledge + status | No |

### Test 5: Support Intent (NO AI used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "I want to speak to a human" | Support team response | No |
| "This is terrible service!" | Support team response | No |

### Test 6: AI Fallback (AI IS used)

| Send in Messenger | Expected Reply | AI Used? |
|-------------------|----------------|----------|
| "What should I wear to a wedding?" | General style advice | **Yes** |
| "How do I style a hoodie?" | Fashion tips | **Yes** |
| "Is cotton better than polyester?" | Fabric comparison | **Yes** |

**How to verify:** In execution logs, "AI Agent (Fallback Only)" shows as executed (green), and "AI Needed?" took the TRUE path.

---

## How to Keep v1 as Backup

- **Do NOT delete** `n8n-messenger-product-lookup-v1.json` from this repo
- **Do NOT delete** the v1 workflow from n8n — just deactivate it
- If v2 has issues, you can instantly reactivate v1:
  1. Deactivate v2
  2. Activate v1
  3. Messages will flow through v1 again (product lookup only)

---

## Cost Monitoring

To track how much AI you are using:

1. In n8n, go to **Executions** → filter by this workflow
2. Click on any execution → check if "AI Agent (Fallback Only)" was executed
3. Count how many executions actually hit the AI node vs. how many were handled by rules

**Expected ratio:** 70-90% of messages should be handled WITHOUT AI (greetings, FAQ, products, tracking, support). Only 10-30% should need the AI fallback.

If AI is being used too much:
- Add more FAQ entries with better Keywords
- Add more Search_Keywords to your Products
- Check if common questions are being missed by the FAQ scorer

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| All messages go to AI | FAQ Keywords not matching | Add more keywords to FAQ tab; lower threshold |
| FAQ returns wrong answer | Keywords overlap between entries | Make keywords more specific per FAQ |
| Products not found | Search_Keywords empty | Fill in Search_Keywords column for all products |
| AI replies with "I don't know" | System prompt too restrictive | Adjust the system prompt in AI Agent node |
| AI invents product details | System prompt not followed | Strengthen "DO NOT invent" rules in prompt |
| Bot doesn't reply at all | Credential issue | Check Page Access Token and workflow active status |
| "Greeting" detected for product questions | Short message with greeting word | The router only treats as greeting if message is 3 words or less |

---

## Architecture Summary

```
Message arrives
    |
    v
[Hybrid Router] checks in order:
    |
    ├─ Greeting/Thanks/Bye? → Static reply (FREE)
    ├─ Support keywords? → Support handler (FREE)
    ├─ Tracking keywords? → Tracking handler (FREE)
    ├─ Buy/Order keywords? → Order handler (FREE)
    └─ None of above? → Read FAQ + Products sheets
                              |
                              ├─ FAQ strong match? → Reply with Answer (FREE)
                              ├─ Product match? → Reply with products (FREE)
                              └─ No match? → AI Agent (COSTS MONEY)
```

**Only the last path costs money.** Everything else is free.
