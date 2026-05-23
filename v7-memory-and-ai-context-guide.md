# v7 Memory & AI Context Guide

## Why v7 Uses Sheet-Based Memory

| Problem in v6 | v7 Solution |
|---------------|-------------|
| Window Buffer Memory resets on n8n restart | Google Sheet memory persists forever |
| No way to track what product was last discussed | `lastProductName`, `lastCategory` stored per customer |
| "ei/eta" has no reference point | Memory provides last shown products with numbers |
| "okay" triggers greeting because agent has no context | `lastBotReply` + `lastIntent` loaded every time |
| Multiple customers share session accidentally | `senderPsid` as unique key per customer |

---

## Memory Schema (Conversation_Memory Tab)

| Column | Type | Purpose | Example |
|--------|------|---------|---------|
| senderPsid | string | Unique customer ID from Messenger | 6842913750284 |
| lastProductId | string | Product ID of last discussed item | PROD-001 |
| lastProductName | string | Name of last discussed product | Classic Black T-Shirt |
| lastCategory | string | Last browsed category | T-Shirts |
| lastSize | string | Size customer mentioned | M |
| lastColor | string | Color customer mentioned | Black |
| lastBudget | string | Budget if customer mentioned | 2000 |
| lastOccasion | string | Occasion if mentioned | casual |
| lastIntent | string | Last detected intent | browse |
| lastUserMessage | string | Customer's last message | ami t shirt khujtechi |
| lastBotReply | string | Bot's last reply text | 1. Classic Black T-Shirt... |
| lastShownProducts | string | Numbered product list last shown | 1:PROD-001,2:PROD-003 |
| updatedAt | ISO datetime | When memory was last updated | 2025-01-20T14:30:00Z |

### How Memory Updates Work

1. **Every message** → Load Memory Sheet → find row where `senderPsid` matches
2. **AI generates reply** → includes `memoryUpdate` in JSON output (only changed fields)
3. **Prepare Memory Update node** → merges AI updates with existing memory
4. **Save Memory node** → `appendOrUpdate` with `senderPsid` as matching column
   - If customer exists → updates their row
   - If new customer → appends a new row

---

## AI Context Window Design

The **Prepare AI Context** node builds a compact context package for the AI. This is the key cost-control mechanism.

### What Gets Sent to AI (Per Request)

```
CUSTOMER MEMORY:
Last product: Classic Black T-Shirt
Last category: T-Shirts
Last intent: browse
Last bot reply: 1. Classic Black T-Shirt - ৳1200 (Size: M,L)...
Last user msg: ami t shirt khujtechi
Last shown products: 1:PROD-001,2:PROD-003

AVAILABLE PRODUCTS:
1. Classic Black T-Shirt | Tk1200 | Size: M,L | Color: Black | Stock: YES | ID: PROD-001
2. Premium V-Neck Tee | Tk1500 | Size: S,M,L,XL | Color: White,Navy | Stock: YES | ID: PROD-002

RELEVANT FAQ:
Q: How long does shipping take?
A: Standard shipping takes 5-7 business days.

CUSTOMER MESSAGE: only ei tshirt available?
```

### What Does NOT Get Sent

- Full product catalog (only top 5 relevant)
- All FAQ entries (only top 3 matching)
- Order history
- Other customers' data
- Full conversation history (compressed into memory fields)

### Token Estimate Per Request

| Component | Approx Tokens |
|-----------|--------------|
| System prompt | ~600 |
| Memory context | ~100-150 |
| Products (5 max) | ~200-300 |
| FAQ (3 max) | ~100-200 |
| Customer message | ~20-50 |
| **Total input** | **~1000-1300** |
| Output (JSON reply) | ~100-200 |
| **Total per request** | **~1200-1500** |

### Cost Estimate (gpt-4.1-mini)

| Metric | Value |
|--------|-------|
| Input cost | ~$0.0004 per request |
| Output cost | ~$0.0003 per request |
| Total per message | ~$0.0007 |
| 1000 messages/day | ~$0.70/day |
| 30,000 messages/month | ~$21/month |

---

## Product Relevance Scoring

The **Prepare AI Context** node scores products to find the top 5 most relevant:

```
Score calculation:
- Each keyword match in product fields: +1 point
- Matches lastCategory from memory: +2 bonus
- Matches lastProductName from memory: +3 bonus
```

### Scoring Example

Customer says: "M size er ki available ache?"
Memory has: lastCategory = "T-Shirts", lastProductName = "Classic Black T-Shirt"

| Product | Keyword "size" | Keyword "available" | Memory Category Bonus | Memory Product Bonus | Total |
|---------|---------------|--------------------|-----------------------|---------------------|-------|
| Classic Black T-Shirt (T-Shirts) | +1 | 0 | +2 | +3 | **6** |
| Slim Fit Jeans (Jeans) | +1 | 0 | 0 | 0 | **1** |
| Premium Tee (T-Shirts) | +1 | 0 | +2 | 0 | **3** |

Result: Classic Black T-Shirt scores highest → AI gets it as context → correctly understands "M size" refers to that T-shirt.

### Fallback Logic

If no keywords match any product:
1. First try: show products from `lastCategory` in memory
2. Second try: show first 5 in-stock products
3. This ensures AI always has some product context to work with

---

## Pronoun & Reference Resolution

### How "ei", "eta", "this" Works

| Customer Says | Memory Has | AI Understands |
|---------------|-----------|----------------|
| "ei tshirt available?" | lastProductName: Classic Black T-Shirt | "Is Classic Black T-Shirt available?" |
| "eta nibo" | lastProductName: Hoodie Pullover | "I want to buy Hoodie Pullover" |
| "oi black ta" | lastShownProducts: 1:PROD-001,2:PROD-003 | Refers to the black one from last list |

The AI is instructed: "ei/eta/this/oi = last discussed product from memory."

### How "2 number ta" Works

When bot shows a numbered list:
```
1. Classic Black T-Shirt - ৳1200
2. Premium V-Neck Tee - ৳1500
3. Hoodie Pullover - ৳2000
```

Memory stores: `lastShownProducts: "1:PROD-001,2:PROD-002,3:PROD-003"`

When customer says "2 number ta nibo":
- AI sees `lastShownProducts` in context
- Understands "2 number ta" = product #2 = Premium V-Neck Tee
- Proceeds with order for that product

### How "okay/accha/hmm" Works

| Customer Says | Memory lastBotReply | AI Does |
|---------------|-------------------|---------|
| "okay" | "Size/colour বললে stock check করি" | Asks "কোন size চান? M নাকি L?" |
| "accha" | "Order করতে চাইলে size দিন" | Asks "Size কত দিবো?" |
| "hmm" | "1. T-Shirt 2. Hoodie দেখবেন?" | Asks "কোনটা ভালো লাগছে?" |

Key: AI sees `lastBotReply` and `lastIntent` → continues that conversation naturally.

---

## AI Output JSON Contract

Every AI response MUST be valid JSON:

```json
{
  "replyText": "জি, এই T-shirt এর M size available আছে. Colour কোনটা চান?",
  "intent": "browse",
  "confidence": "high",
  "aiUsed": true,
  "needsHumanSupport": false,
  "telegramAlertText": "",
  "memoryUpdate": {
    "lastProductId": "PROD-001",
    "lastProductName": "Classic Black T-Shirt",
    "lastCategory": "T-Shirts",
    "lastSize": "M",
    "lastColor": "",
    "lastIntent": "browse",
    "lastShownProducts": ""
  },
  "selectedProductIds": ["PROD-001"],
  "lastShownProducts": [{"number": 1, "id": "PROD-001", "name": "Classic Black T-Shirt"}]
}
```

### Field Descriptions

| Field | Required | Description |
|-------|----------|-------------|
| replyText | Yes | The actual message sent to customer (Bangla/Banglish, short) |
| intent | Yes | browse, order, faq, support, greeting, continuation |
| confidence | Yes | high, medium, low |
| aiUsed | Yes | Always true in v7 |
| needsHumanSupport | Yes | true only for refund/complaint/anger |
| telegramAlertText | No | Alert text for owner (orders, support issues) |
| memoryUpdate | Yes | Object with only changed fields filled |
| selectedProductIds | No | Product IDs referenced in this reply |
| lastShownProducts | No | Array of numbered products if showing a list |

### Validation Node Behavior

The **Validate AI Output** node handles:
1. Extracts JSON from markdown code blocks if AI wraps them
2. Falls back to raw text as replyText if JSON parsing fails
3. Truncates replyText to 500 chars max
4. Overrides with voice-failed message if transcription failed
5. Ensures all required fields have defaults

---

## Intent Classification

| Intent | Triggers | Memory Impact |
|--------|----------|---------------|
| browse | Product questions, "dekhao", "show me" | Updates lastCategory, lastProductName |
| order | Strong phrases: "nibo", "order korte chai", "confirm" | Updates lastIntent=order, triggers alert |
| faq | Policy/shipping/return questions | No product memory change |
| support | Refund, complaint, "human lagbe" | Sets needsHumanSupport=true |
| greeting | "hi", "hello", first message | Minimal memory update |
| continuation | "okay", "hmm", "accha", size/color answers | Preserves existing memory context |

### Strong vs Weak Intent Detection

**Weak (Browse only):**
- chai, lagbe, dekhao, want, need, something, show

**Strong (Order flow):**
- eta nibo, order korte chai, confirm order, buy, order now
- checkout, book this, confirm kore din, order confirm koro
- "2 number ta nibo", "first one ta confirm koro"

---

## Cost Control Mechanisms

### 1. Prefiltered Context (Not Full Sheets)

| Without prefilter | With prefilter |
|-------------------|---------------|
| 50 products × ~50 tokens = 2500 tokens | 5 products × ~50 tokens = 250 tokens |
| 30 FAQ × ~80 tokens = 2400 tokens | 3 FAQ × ~80 tokens = 240 tokens |
| Total context: ~5000+ tokens | Total context: ~500 tokens |
| Cost: ~$0.003/request | Cost: ~$0.0007/request |

**Savings: ~75% cost reduction per message**

### 2. Max Token Limit

AI output capped at 500 tokens (system prompt enforces 1-2 sentence replies). In practice, most replies are 50-100 tokens.

### 3. Low Temperature (0.3)

Reduces randomness → more consistent, shorter replies → fewer tokens wasted.

### 4. Single AI Call Per Message

v6 sometimes made 2-3 AI calls (classify + generate + format). v7 does everything in ONE call.

---

## Memory Lifecycle

### New Customer (First Message)

```
1. Load Memory Sheet → no row found → memory = {}
2. Prepare AI Context → no memory context, shows general products
3. AI replies with product suggestions
4. Save Memory → APPENDS new row with senderPsid
```

### Returning Customer

```
1. Load Memory Sheet → finds row by senderPsid → loads full memory
2. Prepare AI Context → injects memory + relevant products
3. AI uses memory for context continuity
4. Save Memory → UPDATES existing row
```

### Memory Decay (Manual)

Memory doesn't auto-expire. To clean stale data:
- Periodically delete rows where `updatedAt` is older than 7 days
- Or add a scheduled workflow that cleans old memory rows

---

## Edge Cases Handled

| Scenario | How v7 Handles It |
|----------|-------------------|
| Customer sends only emoji | Extract Message sets messageText to emoji, AI responds naturally |
| Voice transcription fails | Set Transcription detects empty result, sets voice_failed flag |
| Google Sheets is slow | Nodes wait; timeout handled by n8n's built-in error handling |
| AI returns invalid JSON | Validate AI Output falls back to raw text as reply |
| AI reply too long | Truncated to 500 chars with "..." |
| Customer has no memory row | Prepare AI Context uses empty object, still works |
| Multiple messages rapid-fire | Dedup Check uses timestamp+senderPsid to skip duplicates |
| Same product asked multiple ways | Scoring considers memory context + keywords |

---

## Testing the Memory System

### Manual Test Sequence

1. **Message 1:** "hoodie dekhao"
   - Check Memory Sheet → lastCategory should be "Hoodies"
   
2. **Message 2:** "black ta available?"
   - Check Memory Sheet → lastColor should be "Black"
   - Reply should reference hoodies (not random products)
   
3. **Message 3:** "M size ache?"
   - Check Memory Sheet → lastSize should be "M"
   - Reply should reference the black hoodie specifically
   
4. **Message 4:** "order korte chai"
   - Reply should say "Phone number আর address দিন" (not ask product/size/color again)
   - Check Memory Sheet → lastIntent should be "order"

5. **Message 5:** "okay"
   - Reply should continue order context, not greet
   - Should still ask for phone/address

### Verifying Memory in Google Sheets

After each test message, open the Conversation_Memory tab and check:
- [ ] Correct senderPsid
- [ ] lastProductName updated when product discussed
- [ ] lastCategory updated when category browsed
- [ ] lastSize/lastColor updated when mentioned
- [ ] lastIntent reflects actual intent
- [ ] lastBotReply matches what was sent
- [ ] updatedAt has current timestamp
