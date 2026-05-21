# Messenger Hybrid v5 — Test Plan

## Overview

This document provides the complete test plan for verifying v5 functionality. Run these tests **before activating v5 in production**.

**How to test:** Use n8n's "Test Workflow" mode with manual POST requests, or temporarily activate v5 and message your Facebook Page from a test account.

---

## Test Categories

| Category | Tests | AI Used? |
|----------|-------|----------|
| Static replies | 3 | No |
| FAQ | 5 | No |
| Typo normalization | 5 | No |
| Product lookup | 4 | No |
| Size recommendation | 3 | No |
| Objection handling | 5 | No |
| Style/occasion | 5 | No (or AI if no sheet match) |
| Memory | 3 | Varies |
| Human support | 3 | No |
| Deduplication | 1 | No |
| AI usage verification | 1 | Yes |
| Order intent (strong only) | 2 | No |

---

## 1. Static Replies (No AI)

| # | Send | Expected Reply Contains | Expected Route | AI? |
|---|------|------------------------|----------------|-----|
| 1.1 | `hi` | "Welcome" or "store" | greeting | No |
| 1.2 | `hello` | "Welcome" or "store" | greeting | No |
| 1.3 | `thanks` | "welcome" or "anything else" | thanks | No |

**Verify:** Execution log shows `route: greeting` or `route: thanks`, `aiUsed: false`.

---

## 2. FAQ (No AI)

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 2.1 | `inside dhaka delivery koto?` | delivery info for Dhaka | faq_match | No |
| 2.2 | `outside dhaka delivery koto?` | delivery info outside Dhaka | faq_match | No |
| 2.3 | `return policy ki?` | return/refund policy info | faq_match | No |
| 2.4 | `exchange kora jabe?` | exchange info | faq_match | No |
| 2.5 | `cash on delivery ache?` | payment/COD info | faq_match | No |

**Verify:** Route is `faq_match`, `aiUsed: false`. Reply matches your FAQ sheet Answer column.

**If test fails:** Add more keywords to your FAQ sheet's Keywords column for that topic.

---

## 3. Typo Normalization (No AI)

| # | Send | Should Be Normalized To | Expected Route |
|---|------|------------------------|----------------|
| 3.1 | `delivry charge koto?` | "delivery charge koto?" → FAQ match | faq_match |
| 3.2 | `tshart ache?` | "t-shirt ache?" → Product match | product_match |
| 3.3 | `exchng policy?` | "exchange policy?" → FAQ match | faq_match |
| 3.4 | `hudie ache?` | "hoodie ache?" → Product match | product_match |
| 3.5 | `prize koto?` | "price koto?" → FAQ or product | faq_match or product_match |

**Verify:** The "Load Memory + Normalize" node output shows `normalizedMessage` with corrected spelling. The downstream routing matches the corrected word.

---

## 4. Product Lookup (No AI)

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 4.1 | `black t-shirt size M ache?` | Product name + ৳ price + sizes | product_match | No |
| 4.2 | `hoodie ache?` | Hoodie products with prices | product_match | No |
| 4.3 | `show me jeans` | Jeans products listed | product_match | No |
| 4.4 | `2000 BDT er moddhe ki ache?` | Products filtered under ৳2000 | product_match | No |

**Verify:** 
- Route is `product_match`
- Products are Active=YES only
- Budget filtering works for test 4.4 (no products above 2000 shown)
- Products show: name, price, sizes, colors
- Limited stock warning (⚠️) shows if Stock_Qty ≤ 3
- Follow-up question asked

---

## 5. Size Recommendation (No AI)

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 5.1 | `amar height 5.6 weight 65, kon size nibo?` | Size recommendation (e.g., "M" or "L") | size_recommendation | No |
| 5.2 | `M hobe?` | Asks for height/weight or confirms M | size_recommendation | No |
| 5.3 | `oversized chai` | Size guidance | style or size | No |

**Verify:**
- If height + weight provided AND Size_Guide sheet has data: specific size recommended
- If height/weight missing: bot asks for the missing info
- `aiUsed: false`

**If test fails:** Ensure Size_Guide tab has rows with Min_Height_Feet, Max_Height_Feet, Min_Weight_Kg, Max_Weight_Kg columns populated.

---

## 6. Objection Handling (No AI)

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 6.1 | `dam beshi` | Value explanation, budget alternative | objection_handling | No |
| 6.2 | `last price?` | Warm reply about value | objection_handling | No |
| 6.3 | `discount hobe?` | Policy about discounts | objection_handling | No |
| 6.4 | `quality kemon?` | Quality assurance | objection_handling | No |
| 6.5 | `fabric kemon?` | Fabric info | objection_handling | No |

**Verify:**
- Route is `objection_handling`
- Reply matches Sales_Objections sheet Response column
- If no sheet match: generic warm reply about quality
- Never invents discounts
- `aiUsed: false`

**If test fails:** Add rows to Sales_Objections with matching Keywords column.

---

## 7. Style/Occasion Recommendation

| # | Send | Expected Reply Contains | Route | AI? |
|---|------|------------------------|-------|-----|
| 7.1 | `I need something stylish for Eid under 2000 BDT` | Product suggestions for Eid ≤ ৳2000 | style_recommendation | No* |
| 7.2 | `Black jeans er sathe ki porbo?` | Outfit pairing suggestion | style_recommendation | No* |
| 7.3 | `Casual look chai` | Casual outfit suggestions | style_recommendation | No* |
| 7.4 | `Gift er jonno suggestion dao` | Gift product suggestions | style_recommendation | No* |
| 7.5 | `Eid er jonno stylish kichu chai` | Eid-appropriate products | style_recommendation | No* |

*No AI if products have Occasion_Tags/Style_Tags that match. Falls to AI if no sheet match.

**Verify:**
- Route is `style_recommendation` (NOT `order_intent`)
- This is the KEY FIX from v3: "I need something stylish for Eid under 2000 BDT" must NOT route to order
- Budget is extracted (2000 in test 7.1)
- Occasion is extracted (eid in test 7.1, 7.5)
- Follow-up question asked
- `aiUsed: false` if products with matching tags found

**Critical test:** Message 7.1 previously routed to order_intent in v3 (because "need" triggered order). In v5 it must route to style_recommendation.

---

## 8. Memory Tests

| # | Step | Send | Expected Behavior |
|---|------|------|-------------------|
| 8.1 | First msg | `black hoodie ache?` | Shows black hoodies, stores product interest in memory |
| 8.2 | Follow-up | `M size ache?` | Understands M size for hoodie (from memory context) |
| 8.3 | Follow-up | `eta nibo` | Should recognize order intent for the hoodie from memory |

**Verify:**
- After 8.1: "Update Memory + Log" node updates memory with product interest
- After 8.2: Memory loaded shows previous interest
- After 8.3: Strong order keyword "nibo" + memory context → order_intent route

**Note:** Memory uses workflow static data. It persists between executions but resets if workflow is re-imported. Memory entries expire after 7 days.

---

## 9. Human Support

| # | Send | Expected Reply Contains | Route | Telegram? |
|---|------|------------------------|-------|-----------|
| 9.1 | `human er sathe kotha bolte chai` | Support confirmation + timeframe | human_support | ✅ |
| 9.2 | `wrong size peyechi` | Support escalation | human_support | ✅ |
| 9.3 | `payment hoyeche but order confirm hoy nai` | Support escalation | human_support | ✅ |

**Verify:**
- Route is `human_support`
- `needsHumanSupport: true`
- `telegramAlertText` is not empty
- Telegram alert sent (check your Telegram group)
- Reply sets expectations (2-4 hours response time)

---

## 10. Deduplication Test

| # | Action | Expected |
|---|--------|----------|
| 10.1 | Send same payload with `message.mid: "dup_test_001"` twice within 5 minutes | First: full processing + reply sent. Second: "Extract + Dedup" returns `skipSend: true`, "Is Valid + Not Duplicate?" takes FALSE path, NO reply sent |

**How to test with curl:**
```bash
# Send first time
curl -X POST "https://your-n8n.app.n8n.cloud/webhook-test/messenger-webhook" \
  -H "Content-Type: application/json" \
  -d '{"object":"page","entry":[{"messaging":[{"sender":{"id":"TEST123"},"recipient":{"id":"PAGE"},"timestamp":1234567890,"message":{"mid":"dup_test_001","text":"hello"}}]}]}'

# Send exact same again immediately
curl -X POST "https://your-n8n.app.n8n.cloud/webhook-test/messenger-webhook" \
  -H "Content-Type: application/json" \
  -d '{"object":"page","entry":[{"messaging":[{"sender":{"id":"TEST123"},"recipient":{"id":"PAGE"},"timestamp":1234567890,"message":{"mid":"dup_test_001","text":"hello"}}]}]}'
```

**Verify:** Only ONE Messenger reply sent. Second execution stops at "Is Valid + Not Duplicate?" FALSE path.

---

## 11. AI Usage Verification

| # | Send | Expected Route | AI Used? |
|---|------|---------------|----------|
| 11.1 | `What fabric is best for summer in Dhaka?` | ai_fallback | **Yes** |

**Verify:**
- "AI Needed?" takes TRUE path
- "AI Agent (Fallback Only)" executes (green in logs)
- "Chat Model" shows as executed
- "Prepare Final Reply" has non-empty `replyText` from AI
- Reply is in same language as customer message
- Reply is SHORT (2-4 lines)
- Reply does NOT invent product names or prices

**How to know AI was used:** Check the canonical output — `aiUsed: true` and `route: ai_fallback`.

---

## 12. Order Intent (Strong Keywords Only)

| # | Send | Expected Route | Should NOT Route To |
|---|------|---------------|---------------------|
| 12.1 | `order korte chai, black hoodie M size` | order_intent | ✅ Correct |
| 12.2 | `I need something nice` | style_recommendation or ai_fallback | ❌ NOT order_intent |

**Verify:** 
- Test 12.1 has strong order keyword ("order korte chai") + product context → routes to order_intent
- Test 12.2 has only weak word ("need") without strong order signal → does NOT route to order_intent
- This confirms weak keywords ("need", "want", "chai", "lagbe") are excluded from order detection

---

## Verification Checklist

After running all tests, confirm:

- [ ] All 3 static replies work (greeting/thanks/bye)
- [ ] FAQ matches work for at least 3 topics
- [ ] Typo normalization corrects at least 3 misspellings
- [ ] Product search returns results with prices
- [ ] Budget filtering works (under XXXX BDT)
- [ ] Style/occasion routes BEFORE order intent
- [ ] "I need something stylish for Eid" does NOT go to order
- [ ] Size recommendation asks for or provides size info
- [ ] Objection handling gives warm value-focused replies
- [ ] Human support triggers Telegram alert
- [ ] Duplicate message.mid does NOT send double reply
- [ ] AI fallback produces helpful short reply
- [ ] Strong order keywords ("order korte chai") trigger order flow
- [ ] Weak words alone ("i want", "chai") do NOT trigger order flow
- [ ] All replies are in customer's detected language style
- [ ] All replies are short (1-4 lines, under 2000 chars)
- [ ] No real credentials in workflow JSON
- [ ] Graph API v25.0 used in Send Messenger Reply URL

---

## Common Issues & Fixes

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Style message goes to order intent | Old v3 still active | Deactivate v3, activate v5 |
| FAQ not matching | Keywords column too sparse | Add more keywords to FAQ sheet |
| Products not found | Active column missing/wrong | Ensure Active = "YES" (uppercase) |
| Size guide not working | Size_Guide tab missing/empty | Create tab with column headers |
| Objection not matching | Sales_Objections tab empty | Add rows with Keywords column |
| AI reply is empty | Chat Model credential missing | Reconnect OpenAI credential |
| Double replies still happening | Old workflow still active | Ensure ONLY v5 is active |
| Memory not working | Workflow re-imported | Memory resets on import — send test messages to rebuild |
| Telegram alert fails | Token/ChatID placeholders | Replace placeholders in Send Telegram Alert node |
| "Something went wrong" reply | Sheet read error | Check Google Sheets credential and document ID |

---

## Test Payload Template

Use this template for manual testing via curl/Postman:

```json
{
  "object": "page",
  "entry": [{
    "messaging": [{
      "sender": {"id": "TEST_PSID_123"},
      "recipient": {"id": "YOUR_PAGE_ID"},
      "timestamp": 1716300000000,
      "message": {
        "mid": "unique_test_mid_001",
        "text": "YOUR TEST MESSAGE HERE"
      }
    }]
  }]
}
```

**Important:** Change `mid` value for each test to avoid dedup blocking subsequent tests.
