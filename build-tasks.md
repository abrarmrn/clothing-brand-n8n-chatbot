# Build Tasks — Hybrid AI v2

## Overview

This is your **step-by-step checklist** to get the Hybrid AI v2 chatbot running on Facebook Messenger. The workflow is pre-built — you just need to import it, connect credentials, and test.

**Estimated time:** 1-2 hours for v2 setup (assumes v1 was already working)

> **Fast-Track Option (v5 — Latest):** Import `n8n-messenger-hybrid-ai-v5.json` and follow `messenger-hybrid-v5-import-instructions.md`. v5 is a stable upgrade from v3 with style routing, dedup, memory, typo normalization, size guide, and objection handling. Run tests from `messenger-hybrid-v5-test-plan.md`. You can be live in ~30 minutes.

---

## What v2 Adds Over v1

| Feature | v1 | v2 |
|---------|----|----|
| Greetings | ❌ | ✅ Static replies (free) |
| FAQ | ❌ | ✅ Matched from Google Sheets (free) |
| Products | ✅ | ✅ Matched from Google Sheets (free) |
| Tracking | ❌ | ✅ Detected and replied (free) |
| Support | ❌ | ✅ Detected and replied (free) |
| Order intent | ❌ | ✅ Detected with guidance (free) |
| AI fallback | ❌ | ✅ For complex questions (costs money) |

---

## Phase 1: Prepare Your Data

*Time estimate: 15-20 minutes*

### Task 1.1: Verify Products Tab Has Data

| Detail | Info |
|--------|------|
| **What to do** | Confirm your Products tab has 15+ variant rows with all 15 columns filled |
| **Why** | The product search reads this tab |
| **Done when** | Products tab has data, Search_Keywords filled, some Active=NO for testing |

### Task 1.2: Add FAQ Data

| Detail | Info |
|--------|------|
| **What to do** | Add at least 10 FAQ entries to the FAQ tab |
| **Why** | v2 now matches FAQ — without data, all questions go to AI (costs money) |
| **Done when** | FAQ tab has 10+ rows with Keywords column well-populated |

**Recommended FAQ topics to add:**

| FAQ_ID | Category | Keywords (example) |
|--------|----------|-------------------|
| FAQ-001 | Shipping | shipping, delivery, how long, days, arrive, when |
| FAQ-002 | Returns | return, refund, exchange, send back, money back |
| FAQ-003 | Sizing | size, sizing, fit, measurements, chart, what size |
| FAQ-004 | Payment | payment, pay, credit card, paypal, apple pay |
| FAQ-005 | Shipping | international, worldwide, other countries, global |
| FAQ-006 | Discount | discount, coupon, promo, code, sale, deal, offer |
| FAQ-007 | Care | wash, care, instructions, laundry, shrink |
| FAQ-008 | Shipping | shipping cost, delivery fee, free shipping |
| FAQ-009 | Returns | how long return, return window, 30 days |
| FAQ-010 | General | hours, business hours, contact, email, phone |

**Important:** The more keywords you add, the better the matching. Add variations and misspellings customers might use.

### Task 1.3: Set Active Flags

| Detail | Info |
|--------|------|
| **What to do** | Make sure Active = "YES" for all rows you want the bot to use |
| **Why** | Inactive rows are completely hidden from search |
| **Done when** | All current products have Active=YES, all current FAQs have Active=YES |

---

## Phase 2: Import v2 Workflow

*Time estimate: 10-15 minutes*

### Task 2.1: Import the JSON File

| Detail | Info |
|--------|------|
| **What to do** | Import `n8n-messenger-hybrid-ai-v2.json` into n8n |
| **Why** | This is the complete v2 workflow pre-built |
| **How** | Menu (⋯) → Import from File → select the JSON |
| **Done when** | You see 17 nodes on the canvas |

**Important:** This creates a NEW workflow. Your v1 workflow is untouched.

### Task 2.2: Connect Google Sheets Credential

| Detail | Info |
|--------|------|
| **What to do** | Connect Google Sheets to "Read FAQ Sheet" and "Read Products Sheet" nodes |
| **Why** | n8n needs permission to read your spreadsheet |
| **How** | Double-click each node → Credential → select or create Google Sheets OAuth2 → set Document + Sheet |
| **Done when** | Both nodes show your spreadsheet and correct sheet names |

**Nodes to update:**
- "Read FAQ Sheet" → Document: Clothing Brand Chatbot Database, Sheet: FAQ
- "Read Products Sheet" → Document: Clothing Brand Chatbot Database, Sheet: Products

### Task 2.3: Connect Facebook Page Access Token

| Detail | Info |
|--------|------|
| **What to do** | Connect your Page Access Token to "Send Messenger Reply" node |
| **Why** | n8n needs this to send replies in Messenger |
| **How** | Double-click "Send Messenger Reply" → Credential → select your existing Header Auth (same as v1) |
| **Done when** | The HTTP Request node has a valid credential |

### Task 2.4: Connect AI Chat Model Credential

| Detail | Info |
|--------|------|
| **What to do** | Add your OpenAI API key to the "Chat Model" node |
| **Why** | The AI Agent needs this for fallback responses |
| **How** | Double-click "Chat Model" → Credential → Create New → OpenAI API → paste your API key |
| **Done when** | The Chat Model node shows a connected credential |

**No OpenAI key yet?** The bot still works for greetings, FAQ, products, tracking, and support. Only the AI fallback will fail. You can add it later.

### Task 2.5: Change Verify Token

| Detail | Info |
|--------|------|
| **What to do** | Replace `CHANGE_ME_VERIFY_TOKEN` with your secret |
| **Why** | Must match what Meta has for your webhook |
| **How** | Double-click "Verify Token Check" → change the token value |
| **Done when** | Token matches what you have in Meta Developer Console |

**If using same webhook as v1:** Use the same verify token. No re-verification needed.

---

## Phase 3: Switch from v1 to v2

*Time estimate: 5 minutes*

### Task 3.1: Deactivate v1

| Detail | Info |
|--------|------|
| **What to do** | Turn OFF your v1 workflow |
| **Why** | Two workflows can't listen on the same webhook path simultaneously |
| **How** | Open v1 workflow → toggle OFF (top-right) |
| **Done when** | v1 shows as inactive |

### Task 3.2: Activate v2

| Detail | Info |
|--------|------|
| **What to do** | Turn ON your v2 workflow |
| **Why** | Makes the production webhook URL active |
| **How** | Open v2 workflow → toggle ON (top-right) |
| **Done when** | v2 shows as active, production URL is live |

**Since both v1 and v2 use the same path (`messenger-webhook`), Meta doesn't need any changes.**

---

## Phase 4: Test Each Route

*Time estimate: 20-30 minutes*

### Task 4.1: Test Greetings (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "Hi" | Welcome message | No |
| "Hello" | Welcome message | No |
| "Assalamualaikum" | Welcome message | No |
| "Thanks" | You're welcome message | No |
| "Bye" | Goodbye message | No |

**Verify in n8n:** Check execution logs — "AI Agent (Fallback Only)" should be greyed out (not executed).

### Task 4.2: Test FAQ (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "What is your return policy?" | Return policy from FAQ tab | No |
| "How long does shipping take?" | Shipping FAQ answer | No |
| "Do you accept PayPal?" | Payment FAQ answer | No |
| "How do I wash my clothes?" | Care FAQ answer | No |

**Verify in n8n:** Check "FAQ + Product Search (No AI)" output — should show `route: faq_match` and `aiUsed: false`.

**If FAQ isn't matching:** Add more keywords to the FAQ Keywords column. The score threshold is 3 — if your keywords are too generic, matches may fail.

### Task 4.3: Test Product Lookup (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "Show me t-shirts" | Product list with sizes/colors | No |
| "Black hoodie in large" | Matching products | No |
| "What do you have under $40?" | Filtered products | No |
| "Purple sandals" | "Couldn't find" + categories | No (falls to AI only if no products AND no FAQ) |

**Verify in n8n:** Output shows `route: product_match` and `aiUsed: false`.

### Task 4.4: Test Tracking Detection (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "Where is my order?" | Asks for order number | No |
| "Track ORD-20250120-001" | Acknowledges order ID | No |
| "Has my package shipped?" | Tracking guidance | No |

### Task 4.5: Test Support Detection (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "I want to speak to a human" | Support escalation message | No |
| "THIS IS TERRIBLE SERVICE" | Support escalation message | No |
| "I need a refund" | Support escalation message | No |

### Task 4.6: Test Order Intent (NO AI)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "I want to buy the black t-shirt" | Order guidance (asks for details) | No |
| "Can I purchase a hoodie?" | Order guidance | No |

### Task 4.7: Test AI Fallback (AI IS Used)

| Send in Messenger | Expected | AI Used? |
|-------------------|----------|----------|
| "What should I wear to a wedding?" | Style advice from AI | **Yes** |
| "Is cotton breathable in summer?" | Fabric info from AI | **Yes** |
| "How do I style a hoodie?" | Fashion tips from AI | **Yes** |

**Verify in n8n:** "AI Agent (Fallback Only)" shows as executed (green). "AI Needed?" took the TRUE path.

### Task 4.8: Test Non-Text Events (Ignored)

| Action | Expected | Reply? |
|--------|----------|--------|
| Send a photo | No reply (correctly skipped) | No |
| Send a sticker | No reply | No |
| Send a voice message | No reply | No |

---

## Phase 5: Optimize and Go Live

*Time estimate: 15-20 minutes*

### Task 5.1: Check AI Usage Ratio

| Detail | Info |
|--------|------|
| **What to do** | Review 20+ test executions and count how many hit AI |
| **Why** | If more than 30% hit AI, your FAQ/Product data needs improvement |
| **How** | Go to Executions → filter by v2 → check each execution's path |
| **Done when** | 70%+ of messages are handled without AI |

### Task 5.2: Add Missing FAQ Entries

| Detail | Info |
|--------|------|
| **What to do** | For messages that went to AI but could be FAQ, add FAQ entries |
| **Why** | Each FAQ entry added = fewer AI calls = less cost |
| **How** | Look at AI fallback executions → note the customer questions → create FAQ rows |
| **Done when** | Common questions are covered by FAQ tab |

### Task 5.3: Add Missing Search_Keywords

| Detail | Info |
|--------|------|
| **What to do** | For product searches that failed, add better Search_Keywords |
| **Why** | Better keywords = more product matches = less AI usage |
| **How** | Check "no product match" executions → see what terms were used → add to Products |
| **Done when** | Common product queries find results |

### Task 5.4: Go-Live Checklist

- [ ] v2 workflow is ACTIVE
- [ ] v1 workflow is INACTIVE (kept as backup)
- [ ] All 3 credentials working (Google Sheets, Facebook, OpenAI)
- [ ] FAQ tab has 10+ entries with rich Keywords
- [ ] Products tab has real data with Search_Keywords filled
- [ ] Greetings work (hi, hello, thanks, bye)
- [ ] FAQ matching works for top questions
- [ ] Product search works for main categories
- [ ] AI fallback works for complex questions
- [ ] Non-text messages are ignored (no error)
- [ ] Replies arrive within a few seconds
- [ ] All replies are under 2000 characters

---

## Phase 6: Ongoing Maintenance

### Daily
- Check n8n execution logs for errors (failed nodes = red)
- Monitor if Page Access Token is still valid

### Weekly
- Review AI fallback executions — are there questions you could add to FAQ?
- Check if customers are searching for products with wrong keywords → add Search_Keywords
- Monitor OpenAI billing — if costs are high, add more FAQ/product data

### Monthly
- Add new products to Products tab as inventory changes
- Update FAQ answers if policies change (update Last_Updated date)
- Review which routes get the most traffic (optimize those first)

---

## Rollback Plan (If v2 Has Issues)

1. In n8n: **Deactivate v2** (toggle OFF)
2. In n8n: **Activate v1** (toggle ON)
3. Messages will flow through v1 again immediately (product lookup only)
4. Debug v2 issues using execution logs
5. Once fixed, switch back (deactivate v1, activate v2)

**Never delete v1.** It's your safety net.

---

## Quick Reference

| What | Where |
|------|-------|
| v2 workflow JSON | `n8n-messenger-hybrid-ai-v2.json` |
| v2 setup guide | `messenger-hybrid-v2-import-instructions.md` |
| v1 workflow JSON (backup) | `n8n-messenger-product-lookup-v1.json` |
| v1 setup guide | `messenger-import-instructions.md` |
| Google Sheets structure | `google-sheets-structure.md` |
| Branching logic details | `chatbot-branching-logic.md` |
| Full architecture | `n8n-workflow-plan.md` |

---

## Future: v3 Tasks (Not Yet)

When v2 is stable and you're ready to enhance:

- [ ] Full order creation with variant validation and Google Sheets write
- [ ] Real Tracking tab lookup (currently just detects intent)
- [ ] Support_Tickets tab write (currently just replies)
- [ ] Customer recognition by PSID from Customers tab
- [ ] Conversation memory for multi-turn order flow
- [ ] Messenger quick-reply buttons
