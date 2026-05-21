# Messenger Hybrid v6 — Import & Setup Instructions

## What's New in v6?

v6 is a **stable upgrade from v5** focused on voice support, shorter replies, and better intelligence:

| Feature | v5 | v6 |
|---------|----|----|
| Voice/audio messages | ❌ Ignored | ✅ Transcribed via Whisper, then routed normally |
| AI Agent mode | Tools Agent (v1.7, unstable) | ✅ Conversational Agent (v1.6, stable) |
| Reply style | Sometimes long English | ✅ Always short mixed Bangla+English (1-2 lines) |
| "I want something" | Went to order intent | ✅ Short clarifying question (no AI cost) |
| Sheet fan-out | Could produce 8000+ items | ✅ Sequential reads, single execution each |
| Memory fields | Basic | ✅ lastProduct, lastCategory, lastSize, lastColor, lastBudget, lastOccasion, lastIntent, lastMessage, updatedAt |
| "eta nibo" + memory | Generic order prompt | ✅ Checks memory.lastProduct, asks only missing info |
| Unclear messages | AI fallback (costly) | ✅ Dedicated branch: "কি type item চান?" |
| AI prompt | Long English rules | ✅ Short mixed Bangla+English, MAX 2 lines enforced |

---

## What Was Preserved from v5

| Kept | Why |
|------|-----|
| message.mid deduplication | Prevents double replies |
| Style/occasion before order intent | Correct routing priority |
| Weak order words excluded | "need/want/chai/lagbe" don't trigger order |
| Single "Send Messenger Reply" node | One final customer send |
| Webhook path `messenger-webhook` | No Meta reconfiguration needed |
| Sequential sheet reads (no fan-out) | Prevents 8000+ item explosion |
| Telegram alerts | Same pattern, same placeholders |
| httpHeaderAuth for Facebook | Same credential type |
| try/catch in all Code nodes | No silent crashes |

---

## Prerequisites

1. **n8n Cloud** (or self-hosted v1.30+) with public HTTPS URL
2. **v5 workflow saved** (do NOT delete — rollback target)
3. **Google Sheets** with FAQ + Products tabs (Size_Guide + Sales_Objections optional)
4. **Facebook Page Access Token** (same as v5)
5. **OpenAI API key** (for AI fallback + audio transcription)
6. **Telegram Bot Token + Chat ID** (for alerts — same as v5)

---

## Step 1: Import v6 as a New Workflow

1. In n8n, click **menu (⋯)** → **"Import from File"**
2. Select: `n8n-messenger-hybrid-ai-v6.json`
3. This creates a **new workflow** — v5 is untouched
4. You should see **29 nodes** on the canvas

---

## Step 2: Connect Google Sheets Credential

Connect to **four** Google Sheets nodes:

| Node | Sheet Tab |
|------|-----------|
| "Read FAQ Sheet" | FAQ |
| "Read Products Sheet" | Products |
| "Read Size Guide" | Size_Guide |
| "Read Sales Objections" | Sales_Objections |

For each: double-click → select Google Sheets OAuth2 credential → set Document ID + Sheet name.

**If Size_Guide or Sales_Objections don't exist:** Create empty tabs with header rows. The workflow handles empty sheets gracefully.

---

## Step 3: Connect Facebook Header Auth Credential

1. Double-click **"Send Messenger Reply"** node
2. Select Credential: Header Auth
3. Use your existing credential:
   - Header Name: `Authorization`
   - Header Value: `Bearer YOUR_PAGE_ACCESS_TOKEN`

---

## Step 4: Connect AI Chat Model Credential

1. Double-click **"Chat Model"** node
2. Select Credential: OpenAI API
3. Use your existing OpenAI key (same as v5)
4. Model is pre-set to `gpt-4o-mini`

**Verify:** The Chat Model node must show a line connecting to "AI Agent (Fallback Only)" via the AI input.

---

## Step 5: Set Up Audio Transcription (NEW in v6)

The **"Transcribe Audio"** Code node calls OpenAI Whisper API directly.

1. Open the **"Transcribe Audio"** node
2. Find the line: `'Authorization': 'Bearer OPENAI_TRANSCRIPTION_CREDENTIAL_PLACEHOLDER'`
3. Replace `OPENAI_TRANSCRIPTION_CREDENTIAL_PLACEHOLDER` with your actual OpenAI API key

**Same key as Chat Model?** Yes — you can use the same OpenAI key for both AI Agent and Whisper transcription.

**No voice messages expected?** The workflow still works fine for text-only. Audio branch only activates when a voice note arrives.

---

## Step 6: Set Up Telegram Bot (Same as v5)

1. Double-click **"Send Telegram Alert"** node
2. Replace `TELEGRAM_BOT_TOKEN_PLACEHOLDER` in URL with your bot token
3. Replace `TELEGRAM_CHAT_ID_PLACEHOLDER` in body with your chat ID

---

## Step 7: Change Verify Token

1. Double-click **"Verify Token Check"** node
2. Change `CHANGE_ME_VERIFY_TOKEN` to your verify token (same as v5)

---

## Step 8: Switch from v5 to v6

1. **Deactivate v5** (toggle OFF)
2. **Activate v6** (toggle ON)
3. Same webhook path — no Meta changes needed

---

## How Voice/Audio Works

```
Customer sends voice note in Messenger
        |
        v
Extract + Dedup detects audio attachment
        |
        v
Has Audio? → TRUE → Transcribe Audio (OpenAI Whisper)
        |
        v
Handle Transcription Result
        |
        ├── Failed? → Reply: "Voice ta clear বুঝতে পারিনি। Text করে বলবেন।"
        └── Success? → Text continues normal routing (FAQ/Product/Style/etc.)
```

**Key points:**
- Whisper API supports Bengali/Bangla audio
- Language hint set to `bn` (Bengali) for better accuracy
- If transcription fails, customer gets polite error (no crash)
- Transcribed text goes through same Decision Engine as typed text

---

## How to Test Voice Without Production

1. Keep v5 active (production) and v6 inactive
2. Open v6 → click "Test Workflow"
3. Send test payload with audio attachment:

```json
{
  "object": "page",
  "entry": [{
    "messaging": [{
      "sender": {"id": "TEST_PSID_123"},
      "recipient": {"id": "PAGE_ID"},
      "timestamp": 1234567890,
      "message": {
        "mid": "test_audio_001",
        "attachments": [{
          "type": "audio",
          "payload": {
            "url": "https://example.com/test-audio.mp4"
          }
        }]
      }
    }]
  }]
}
```

4. Check execution: "Has Audio?" should take TRUE path
5. "Transcribe Audio" will fail (test URL not real) → "Handle Transcription Result" returns polite error

**To test real transcription:** Use a real audio URL from a Messenger voice message (check webhook logs for actual payload format).

---

## How to Confirm Replies Stay Short

1. Send these test messages and check reply length:
   - "I want something" → Should reply: "কি type item চান? T-shirt, hoodie, jeans নাকি dress?" (1 line)
   - "hi" → Should reply: "Walaikumussalam! কি খুঁজছেন আজ?" (1 line)
   - "dam beshi" → Should reply from Sales_Objections sheet (1-2 lines)

2. For AI fallback, send a complex question and verify reply is ≤3 lines

3. Check the AI Agent system prompt (open "AI Agent (Fallback Only)" node) — it says "MAX 2 lines"

---

## Credential Summary

| Credential | Type | Used By | Placeholder |
|-----------|------|---------|-------------|
| Google Sheets OAuth2 | googleSheetsOAuth2Api | 4 sheet read nodes | GOOGLE_SHEETS_CREDENTIAL_PLACEHOLDER |
| Facebook Page Token | httpHeaderAuth | Send Messenger Reply | FACEBOOK_PAGE_ACCESS_TOKEN_PLACEHOLDER |
| OpenAI API (Chat) | openAiApi | Chat Model | AI_CHAT_MODEL_CREDENTIAL_PLACEHOLDER |
| OpenAI API (Whisper) | (hardcoded in Code) | Transcribe Audio | OPENAI_TRANSCRIPTION_CREDENTIAL_PLACEHOLDER |
| Telegram Bot Token | (in URL) | Send Telegram Alert | TELEGRAM_BOT_TOKEN_PLACEHOLDER |
| Telegram Chat ID | (in body) | Send Telegram Alert | TELEGRAM_CHAT_ID_PLACEHOLDER |
| Verify Token | (in IF condition) | Verify Token Check | CHANGE_ME_VERIFY_TOKEN |

---

## Rollback Plan

1. **Deactivate v6** (toggle OFF)
2. **Activate v5** (toggle ON)
3. Messages flow through v5 immediately
4. Debug v6 using execution logs

**Never delete v5.** It's your safety net.

---

## Memory Limitation (Documented)

v6 uses **workflow static data** for customer memory. Limitations:

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| Resets on workflow re-import | Memory lost | Re-import only when needed |
| Not shared across workflow copies | Each workflow has own memory | Use only ONE active workflow |
| Expires after 7 days of inactivity | Old conversations forgotten | Acceptable for chat context |
| Max ~500 concurrent customers | Static data size limit | Sufficient for small-medium brands |

**Future v7:** Google Sheets-based persistent memory (survives re-imports).

---

## Architecture Diagram

```
POST Webhook ──→ Respond 200 OK (Meta ACK)
      │
      └──→ Extract + Dedup (text + audio detection)
                │
          Is Valid + Not Duplicate?
                │ TRUE
          Has Audio?
          /         \
    TRUE /           \ FALSE
  Transcribe         │
  Audio              │
    │                │
  Handle Result      │
    │                │
  Skip To Reply?     │
  /        \         │
TRUE        FALSE    │
  │           └──────┤
  │                  │
  │         Load Memory + Normalize
  │                  │
  │         Read FAQ → Products → Size_Guide → Sales_Objections
  │                  │
  │         Decision Engine v6
  │                  │
  │         AI Needed?
  │         /        \
  │     TRUE          FALSE
  │   AI Agent          │
  │        \           │
  └────────→ Prepare Final Reply
                  │
            Update Memory + Log
                  │
            Should Send?
            /          \
      TRUE /            \ FALSE
  Send Reply     Needs Telegram?
      │               │ TRUE
  Mark Processed  Prepare Alert
                      │
                Send Telegram
```
