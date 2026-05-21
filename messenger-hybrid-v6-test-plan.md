# Messenger Hybrid v6 — Test Plan

## Overview

Test plan for v6 with 40+ test cases. Focus areas: voice/audio, short replies, unclear messages, memory, and dedup.

**How to test:** Use n8n "Test Workflow" mode with manual POST requests, or activate v6 and message your Facebook Page.

---

## 1. Static Replies (No AI)

| # | Send | Expected Reply | Route | AI? |
|---|------|---------------|-------|-----|
| 1.1 | `hi` | Walaikumussalam! কি খুঁজছেন আজ? | greeting | No |
| 1.2 | `hello` | Hello! What are you looking for? | greeting | No |
| 1.3 | `thanks` | Welcome! আর কিছু লাগলে বলবেন। | thanks | No |
| 1.4 | `bye` | Allah hafez! আবার আসবেন। | bye | No |

**Verify:** Reply is 1 line only. `route` field in execution shows correct value.

---

## 2. FAQ (No AI)

| # | Send | Expected | Route | AI? |
|---|------|----------|-------|-----|
| 2.1 | `inside dhaka delivery koto?` | Inside Dhaka delivery info only | faq_match | No |
| 2.2 | `outside dhaka delivery koto?` | Outside Dhaka info only | faq_match | No |
| 2.3 | `return policy ki?` | Return/exchange policy | faq_match | No |
| 2.4 | `cash on delivery ache?` | Payment/COD info | faq_match | No |

**Verify:** Reply answers ONLY what was asked (no extra info).

---

## 3. Typo Normalization (No AI)

| # | Send | Normalized To | Expected Route |
|---|------|--------------|----------------|
| 3.1 | `delivry charge koto?` | delivery → FAQ match | faq_match |
| 3.2 | `tshart ache?` | t-shirt → Product match | product_match |
| 3.3 | `hudie ache?` | hoodie → Product match | product_match |
| 3.4 | `exchng policy?` | exchange → FAQ match | faq_match |

---

## 4. Product Lookup (No AI)

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 4.1 | `black t-shirt ache?` | Product name + ৳price + sizes | product_match | No |
| 4.2 | `hoodie dekhao` | Hoodie products (max 3) | product_match | No |
| 4.3 | `2000 BDT er moddhe ki ache?` | Products ≤ ৳2000 only | product_match | No |

**Verify:** Max 3 products shown. Ends with "Size/colour বললে exact stock check করে দিতে পারি।"

---

## 5. Style/Occasion (Before Order — No AI if products match)

| # | Send | Expected Route | Should NOT Route To |
|---|------|---------------|---------------------|
| 5.1 | `I need something stylish for Eid under 2000 BDT` | style_recommendation | ❌ NOT order_intent |
| 5.2 | `casual look chai` | style_recommendation | ❌ NOT order_intent |
| 5.3 | `ki porbo party te?` | style_recommendation | ❌ NOT ai_fallback |

**Critical:** Test 5.1 is THE key fix. Must route to style, not order.

---

## 6. Unclear Messages (NEW — No AI)

| # | Send | Expected Reply | Route | AI? |
|---|------|---------------|-------|-----|
| 6.1 | `I want something` | কি type item চান? T-shirt, hoodie, jeans নাকি dress? | unclear_message | No |
| 6.2 | `kichu dekhan` | কি type item চান? T-shirt, hoodie, jeans নাকি dress? | unclear_message | No |
| 6.3 | `suggest koren` | কি type item চান? T-shirt, hoodie, jeans নাকি dress? | unclear_message | No |
| 6.4 | `need something` | কি type item চান? T-shirt, hoodie, jeans নাকি dress? | unclear_message | No |

**Verify:** These do NOT go to AI fallback. Route is `unclear_message`. `aiUsed: false`.

---

## 7. Strong Order Intent (No AI)

| # | Send | Expected Route | Notes |
|---|------|---------------|-------|
| 7.1 | `order korte chai, black hoodie M` | order_intent | Has product context |
| 7.2 | `eta nibo` (after browsing hoodie) | order_intent | Uses memory.lastProduct |
| 7.3 | `I want something nice` | unclear_message | Weak words only → NOT order |

**Verify:** Test 7.3 does NOT route to order. Only strong signals ("order korte chai", "buy", "nibo") with product context trigger order.

---

## 8. Memory Tests

| # | Step | Send | Expected |
|---|------|------|----------|
| 8.1 | First | `black hoodie ache?` | Shows hoodies, stores lastProduct in memory |
| 8.2 | Follow-up | `M size?` | Understands M for hoodie (memory context) |
| 8.3 | Follow-up | `eta nibo` | "ঠিক আছে। Size, colour আর delivery address দিবেন?" |

**Verify:** After 8.1, "Update Memory + Log" stores `lastProduct: "hoodie"` (or actual product name). After 8.3, order branch uses memory context.

---

## 9. Human Support

| # | Send | Expected Reply | Telegram? |
|---|------|---------------|-----------|
| 9.1 | `human er sathe kotha bolte chai` | বুঝতে পারছি। Team 2-4 ঘন্টায় contact করবে। | ✅ |
| 9.2 | `wrong size peyechi` | Support escalation | ✅ |
| 9.3 | `THIS IS TERRIBLE` (all caps) | Support escalation | ✅ |

---

## 10. Voice/Audio (NEW)

| # | Action | Expected | Notes |
|---|--------|----------|-------|
| 10.1 | Send voice note with clear speech | Transcribed → routed normally | Check "Has Audio?" TRUE path |
| 10.2 | Send voice note (transcription fails) | "Voice ta clear বুঝতে পারিনি। Text করে বলবেন।" | Polite fallback |
| 10.3 | Send image (not audio) | Skipped (no reply or "no_text_or_audio") | Extract+Dedup handles |

**How to test 10.1 without real audio:**
Use test payload with audio URL. In test mode, transcription will fail (URL not accessible) → tests 10.2 behavior.

**To test real transcription:**
1. Activate v6
2. Send a voice note from Messenger to your page
3. Check execution logs: "Transcribe Audio" should show transcribed text
4. Reply should be based on what was said in the voice note

---

## 11. Deduplication

| # | Action | Expected |
|---|--------|----------|
| 11.1 | Send same payload with `mid: "dup_test_001"` twice | First: processes + replies. Second: "Is Valid + Not Duplicate?" FALSE path, NO reply |

---

## 12. AI Fallback (Should Be Rare)

| # | Send | Expected Route | AI? | Reply Check |
|---|------|---------------|-----|-------------|
| 12.1 | `cotton naki polyester better summer e?` | ai_fallback | **Yes** | Reply ≤ 2 lines, mixed Bangla+English |

**Verify:**
- `route: ai_fallback`, `aiUsed: true`
- AI Agent (v1.6 Conversational) executes
- Reply is SHORT (max 2-3 lines)
- Reply uses mixed Bangla+English style
- Reply does NOT invent prices or stock

---

## 13. Reply Length Enforcement

| # | Check | How |
|---|-------|-----|
| 13.1 | All non-AI replies | Should be 1-2 lines |
| 13.2 | AI fallback reply | Should be max 2-3 lines |
| 13.3 | Product list | Max 3 products + 1 line follow-up |
| 13.4 | No reply > 1950 chars | Truncation in Prepare Final Reply |

---

## Debug Info (How to Know Which Branch Answered)

In every execution, check the **"Prepare Final Reply"** output:

| Field | What It Tells You |
|-------|-------------------|
| `route` | Which branch answered (greeting, faq_match, product_match, style_recommendation, unclear_message, ai_fallback, etc.) |
| `intent` | Detected customer intent |
| `aiUsed` | true = AI was called (costs money), false = free |
| `confidence` | high/medium/low |
| `memoryUpdate` | What was stored in memory this turn |

---

## Test Payload Template

```json
{
  "object": "page",
  "entry": [{
    "messaging": [{
      "sender": {"id": "TEST_PSID_123"},
      "recipient": {"id": "YOUR_PAGE_ID"},
      "timestamp": 1716300000000,
      "message": {
        "mid": "unique_test_mid_XYZ",
        "text": "YOUR TEST MESSAGE HERE"
      }
    }]
  }]
}
```

**Change `mid` for each test** to avoid dedup blocking.

---

## Audio Test Payload

```json
{
  "object": "page",
  "entry": [{
    "messaging": [{
      "sender": {"id": "TEST_PSID_123"},
      "recipient": {"id": "YOUR_PAGE_ID"},
      "timestamp": 1716300000000,
      "message": {
        "mid": "audio_test_001",
        "attachments": [{
          "type": "audio",
          "payload": {
            "url": "https://scontent.xx.fbcdn.net/v/t39.12345-6/audio.mp4"
          }
        }]
      }
    }]
  }]
}
```

---

## Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| AI replies too long | Old v5 still active | Deactivate v5, activate v6 |
| Voice note ignored | hasAudio not detected | Check attachment.type === 'audio' in payload |
| "I want something" triggers order | Using v5 not v6 | Switch to v6 |
| Sheet read error | Credential not connected | Reconnect Google Sheets on all 4 nodes |
| Double replies | Wrong workflow active | Only ONE workflow should be active |
| Transcription fails | OpenAI key not set in Transcribe Audio code | Replace placeholder in Code node |
