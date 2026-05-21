# Project Master Context — Clothing Brand Messenger Chatbot

## Identity

**Repo:** `abrarmrn/clothing-brand-n8n-chatbot`
**Platform:** Facebook Messenger chatbot for a Bangladeshi clothing brand
**Stack:** n8n (automation) + Google Sheets (database) + AI Agent (fallback only) + Telegram (human alerts)
**Versions:** v1 (product lookup), v2 (hybrid AI), v3 (fixed AI + language), v4 (full production with memory/vision/dedup)

---

## Core Architecture (v4)

```
Customer (Messenger) → Meta POST → n8n Webhook → 200 ACK parallel
    → Extract Event → Dedup by message.mid → Load Memory → Normalize + Language Detect
    → Hybrid Router (priority-ordered) → Sheet Lookup OR Direct Reply OR AI/Vision
    → Prepare Final Reply → Skip Send check → Send ONE Messenger Reply
    → Update Memory + Save Ticket + Send Telegram Alert (parallel side effects)
    → Mark Processed
```

**Key Rule:** Only ONE `Send Messenger Reply` node sends the customer-facing message. The `Respond 200 OK` is Meta ACK only — never customer reply.

---

## Webhook Structure

| Method | Path | Purpose |
|--------|------|---------|
| GET | `messenger-webhook-v4` | Meta verification (hub.mode + hub.verify_token + hub.challenge) |
| POST | `messenger-webhook-v4` | Receives all Messenger events |

**GET flow:** Verify Token Check → if match: return hub.challenge as text/plain 200; if mismatch: 403.
**POST flow:** Immediately respond 200 "EVENT_RECEIVED" in parallel with processing.

---

## Canonical Output Contract

Every branch MUST output this exact shape:

```json
{
  "senderPsid": "string",
  "messageMid": "string",
  "customerMessage": "string (original)",
  "normalizedMessage": "string (lowercased, typo-fixed)",
  "languageStyle": "bangla|banglish|english|mixed",
  "intent": "string",
  "confidence": "high|medium|low",
  "replyText": "string (the final customer reply)",
  "needsHumanSupport": false,
  "telegramAlertText": "string (empty if no alert)",
  "memoryUpdate": { "Last_Intent": "", "Last_Message": "", "Language_Style": "" },
  "skipSend": false
}
```

---

## Routing Priority (Hybrid Router)

Checked top-to-bottom, first match wins:

| # | Intent | Route | AI? |
|---|--------|-------|-----|
| 1 | Skip (delivery/read/echo/dup) | stop | No |
| 2 | human_support | direct_reply + ticket + telegram | No |
| 3 | greeting/thanks/bye | direct_reply | No |
| 4 | image_message | ai_vision | Yes |
| 5 | order_tracking | check_sheets (Tracking) | No |
| 6 | order_confirm (STRONG phrases only) | order_intent | No |
| 7 | size_recommendation | check_sheets (Size_Guide) | No |
| 8 | objection_handling | check_sheets (Sales_Objections) | No |
| 9 | faq_or_product | check_sheets (FAQ → Products) | No |
| 10 | style_occasion | check_sheets → AI if no match | Maybe |
| 11 | complex_question | ai_fallback | Yes |
| 12 | unclear | direct generic prompt | No |

**Critical Fix (v4):** Style/occasion queries ("stylish for Eid under 2000 BDT") route to style_occasion (#10), NOT order_confirm. Only STRONG order phrases trigger order: `buy now`, `order korte chai`, `eta nibo`, `confirm kore din`, `checkout`.

**Removed from order triggers:** "want", "need", "chai", "lagbe", "something" (too broad).

---

## Language Detection

| Style | Detection |
|-------|-----------|
| bangla | Contains Unicode ০৯৮০-০৯FF |
| banglish | Contains known Banglish words (ki, kemon, ache, chai, korbo, hobe, diben, koto, nibo, etc.) or particles (er, ta, tay, te, gulo, hobe) |
| english | Default when neither above matches |
| mixed | Both Bangla Unicode AND Banglish words |

**Rule:** Reply in same language style as customer. Short messages (≤2 words) inherit from memory.

---

## Typo/Alias Normalization (Pre-AI)

Applied before routing: `delivry→delivery`, `tshart→tshirt`, `hudie→hoodie`, `jins→jeans`, `prize→price`, `availble→available`, `dhaka city→inside dhaka`, `dhakar vitore→inside dhaka`, `dhakar bahire→outside dhaka`.

---

## Google Sheets Structure

**Document:** "Clothing Brand Chatbot Database"

| Tab | Key Columns | Purpose |
|-----|-------------|---------|
| Products | Product_ID, Variant_SKU, Product_Name, Category, Description, Price, Currency, Size, Color, Stock_Qty, In_Stock, Image_URL, Product_URL, Search_Keywords, Active + optional: Discount_Price, Offer_Text, Is_Featured, Urgency_Message, Occasion_Tags, Style_Tags, Gender, Age_Group | One row per variant (size+color) |
| FAQ | FAQ_ID, Category, Question, Keywords, Answer, Active, Sort_Order, Last_Updated | Keyword-matched answers |
| Orders | 25 columns: Order_ID through Notes (includes Variant_SKU, Payment_Status, Order_Status, Channel) | Chatbot-created orders |
| Tracking | Order_ID, Customer_Email, Tracking_Number, Carrier, Shipping_Status, Shipped_Date, Estimated_Delivery, Delivered_Date, Tracking_URL, Last_Updated, Updated_By | Shipping info |
| Customers | Customer_ID, Chat_User_ID, Customer_Name, Email, Phone, Preferred_Channel, First_Contact_Date, Last_Contact_Date, Last_Message_Date, Total_Orders, Total_Spent, Notes | Customer profiles |
| Support_Tickets | 18 columns: Ticket_ID through Resolution_Notes (includes Bot_Summary, Handoff_Reason, Priority) | Human escalation |
| Conversation_Memory | Memory_ID, Sender_PSID, Customer_Name, Language_Style, Last_Intent, Last_Product_Interest, Last_Size, Last_Color, Last_Budget, Last_Message, Short_Summary, Updated_At | Per-customer context |
| Processed_Messages | Message_ID, Sender_PSID, Processed_At, Intent, Short_Summary | Dedup tracking |
| Size_Guide | Category, Size, Chest, Length, Shoulder, Waist, Recommended_Height, Recommended_Weight, Fit_Type, Notes | Size recommendations |
| Sales_Objections | Objection_ID, Objection_Type, Keywords, Best_Response_Bangla, Best_Response_Banglish, Best_Response_English, Follow_Up_Question, Active | Handle price/quality concerns |
| Lead_Followups | Lead_ID, Sender_PSID, Customer_Name, Last_Product_Interest, Last_Size, Last_Color, Last_Budget, Followup_Status, Last_Message_Time, Next_Followup_Time, Notes | Warm lead tracking |

---

## AI Rules

**When AI runs:** Only when rules + sheets cannot answer confidently (style questions, complex queries, image understanding).

**AI System Prompt constraints:**
- Reply in customer's language style
- Keep under 100 words
- Ask ONE follow-up question
- NEVER invent prices, stock, delivery dates, order status
- For order tracking: tell customer to type "track ORD-XXXXX"
- For complaints: tell customer to type "human"
- For products: suggest "show me [category]"
- Use BDT currency, mention Eid/occasions when relevant

**AI Output Fix:** Prepare Final Reply reads: `data.replyText` → `data.output` (LangChain) → `data.text` → `data.response` → `data.message` → safe fallback.

---

## Deduplication

- Extract `message.mid` from Meta payload
- Check against `$getWorkflowStaticData('global').processedMids` array
- If duplicate: set `skipSend: true`, stop before reply
- After successful send: push mid to array (cap at 500)
- Optional: also write to Processed_Messages sheet for persistence

---

## Messenger Send API

**Single node:** "Send Messenger Reply"
- Method: POST
- URL: `https://graph.facebook.com/v25.0/me/messages`
- Auth: Header Auth → `Authorization: Bearer FACEBOOK_PAGE_ACCESS_TOKEN_PLACEHOLDER`
- Body: `{ recipient: { id: senderPsid }, messaging_type: "RESPONSE", message: { text: replyText } }`
- Max reply: 2000 chars (truncate at 1900 for safety)

---

## Telegram Alerts

**Trigger:** `needsHumanSupport === true` OR `telegramAlertText` not empty
**URL:** `https://api.telegram.org/botTELEGRAM_BOT_TOKEN_PLACEHOLDER/sendMessage`
**Body:** `{ chat_id: "TELEGRAM_CHAT_ID_PLACEHOLDER", text: alertText, disable_web_page_preview: true }`
**Content:** PSID, message, language, intent, handoff reason, last product interest, timestamp

---

## Placeholder Credentials (Never Real Values)

| Placeholder | Purpose |
|-------------|---------|
| `CHANGE_ME_VERIFY_TOKEN` | Meta webhook verification |
| `FACEBOOK_PAGE_ACCESS_TOKEN_PLACEHOLDER` | Messenger Send API |
| `TELEGRAM_BOT_TOKEN_PLACEHOLDER` | Telegram Bot API |
| `TELEGRAM_CHAT_ID_PLACEHOLDER` | Telegram chat/group |
| `AI_CHAT_MODEL_CREDENTIAL_PLACEHOLDER` | OpenAI/Anthropic for AI Agent |
| `AI_VISION_MODEL_CREDENTIAL_PLACEHOLDER` | Vision model (gpt-4o) |

---

## Cost-Saving Priority

| Free (no AI) | Paid (AI) |
|--------------|-----------|
| Greetings/thanks/bye | Style/occasion recommendation |
| FAQ strong keyword match | Complex shopping advice |
| Product lookup from sheets | Image understanding |
| Tracking lookup | Unclear/typo when rules fail |
| Size guide lookup | General conversational fallback |
| Objection response from sheets | |
| Draft order with clear data | |
| Human support + Telegram | |

**Target:** 70-90% handled without AI.

---

## Sales Psychology Rules

- Short (1-4 lines), warm, confident, human-like
- Ask ONE follow-up question
- Mention benefits naturally, no pressure
- Limited stock only when Stock_Qty ≤ 3 (real data)
- Discounts only when Discount_Price exists in sheet
- Never fake urgency or invent discounts
- Answer ONLY what was asked (inside Dhaka ≠ full delivery policy)

---

## Version History

| Version | Nodes | Key Features | Status |
|---------|-------|-------------|--------|
| v1 | 11 | Product lookup only | Backup |
| v2 | 21 | Hybrid routing, FAQ, 6 branches | Backup |
| v3 | 24 | Fixed AI .output, language detect, Telegram | Backup |
| v4 | 35+ | Dedup, memory, vision, size guide, objections, fuzzy match, strong-order-only | Latest |

**Rule:** Never delete previous version JSONs. Rollback = deactivate current, activate previous.

---

## Critical n8n Design Rules

1. Never reference `$('Node Name')` unless that node is guaranteed to execute before current
2. Use try/catch in every Code node
3. Pass all required data forward via JSON (don't rely on lateral node access)
4. One final Messenger reply only
5. Respond 200 OK is for Meta only, never customer reply
6. If error occurs: generate safe replyText + set needsHumanSupport=true
7. Validate JSON before commit (use Python generator for large workflows)
8. Keep node names beginner-friendly
