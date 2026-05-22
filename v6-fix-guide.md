# v6 Chatbot Fix Guide

## Problems Identified

| # | Problem | Root Cause | Fix |
|---|---------|-----------|-----|
| 1 | "ei tshirt" interpreted as "EI category" | System prompt had no Bangla context awareness. AI treated "ei" as English abbreviation. | New prompt explicitly states "ei" = এই = "this" in Bangla. Never a category. |
| 2 | "okay" resets to greeting | No context memory rules in system prompt. Agent treated each message independently. | New prompt has explicit rules for acknowledgment words. Memory window increased. |
| 3 | Long robotic English replies | System prompt was English-oriented with no language preference set. | New prompt enforces Bangla/Banglish, short replies, max 1-2 sentences. |
| 4 | Weak intent triggers order | No distinction between browsing and ordering intent. | New prompt separates weak (browse) vs strong (order) trigger phrases. |

---

## Step-by-Step Fix Instructions

### Step 1: Update AI Agent System Prompt

1. Open your n8n workflow
2. Double-click the **AI Agent** node
3. Make sure mode is set to **Conversational Agent**
4. In the **System Message** field, delete everything
5. Paste the full prompt from `ai-agent-system-prompt.md`
6. Save the node

### Step 2: Fix Memory Configuration

The #1 reason context resets is **insufficient memory**. Fix:

1. Check if you have a **Memory** sub-node connected to the AI Agent
2. If missing, add one: click the AI Agent → Add Sub-Node → **Window Buffer Memory**
3. Configure:
   - **Context Window Length**: `10` (minimum, 15 recommended)
   - **Session ID**: `{{ $json.sender_id }}` or `{{ $('Messenger Trigger').item.json.sender.id }}`
4. This ensures the bot remembers the last 10-15 messages per user

**Critical:** The Session ID MUST be unique per user. If all users share one session, conversations will cross-contaminate.

### Step 3: Verify Single Output Node

1. Make sure there is only ONE node named **"Send Messenger Reply"** at the end
2. All AI Agent outputs should flow to this single node
3. Remove any duplicate Send nodes or parallel paths

### Step 4: Remove Any Intent Classifier (if present)

If you previously added an AI Intent Classifier or Switch node BEFORE the AI Agent:
1. Remove it
2. Connect the Messenger Trigger directly to the AI Agent
3. Let the Conversational Agent handle all routing internally via its system prompt

### Step 5: Verify Messenger Trigger → AI Agent Connection

Your workflow should be:
```
[Messenger Trigger] → [AI Agent (Conversational)] → [Send Messenger Reply]
                              |
                              ├── Memory (Window Buffer)
                              ├── Model (OpenAI/GPT-4o-mini or similar)
                              └── Tools (Google Sheets lookup - if using tools)
```

If you are NOT using Tools mode, your product data should be injected as context in the system prompt or via a Code node before the AI Agent.

---

## Memory Fix Details

### Problem: Why Context Resets

| Cause | Solution |
|-------|----------|
| No memory node connected | Add Window Buffer Memory |
| Session ID not set | Set to `{{ $json.sender_id }}` |
| Context window too small (1-3) | Increase to 10-15 |
| Memory cleared between executions | Use persistent memory (Redis/Postgres) for production |
| Different Session ID format | Make sure same field used across all runs |

### Recommended Memory Settings

```
Type: Window Buffer Memory
Context Window Length: 15
Session ID: {{ $('Messenger Trigger').item.json.sender.id }}
```

For production with many users, consider:
- **Postgres Chat Memory** or **Redis Chat Memory** for persistence
- These survive n8n restarts

---

## Node Configuration Checklist

### AI Agent Node Settings
- [ ] Mode: **Conversational Agent** (NOT Tools Agent)
- [ ] System Message: Updated prompt from `ai-agent-system-prompt.md`
- [ ] Connected Memory: Window Buffer Memory (context window ≥ 10)
- [ ] Connected Model: GPT-4o-mini or GPT-4o (recommended for Bangla understanding)
- [ ] Session ID: Properly set to sender's unique ID

### Messenger Trigger Node
- [ ] Outputs sender ID that can be used as Session ID
- [ ] Connected directly to AI Agent (no classifier in between)

### Send Messenger Reply Node
- [ ] Named exactly: "Send Messenger Reply"
- [ ] Only ONE such node exists
- [ ] Receives output from AI Agent

---

## Product Context Injection (If NOT Using Tools)

If your AI Agent does NOT have Google Sheets as a tool, you need to inject product data before it:

### Option A: Code Node Before AI Agent

Add a **Code** node between Messenger Trigger and AI Agent:

```javascript
// Fetch from previous Google Sheets node or hardcode for now
const products = $('Google Sheets').all();

const productList = products.map(p => 
  `${p.json.Product_Name} - ৳${p.json.Price} (Size: ${p.json.Sizes_Available}, Color: ${p.json.Colors_Available}) [${p.json.In_Stock}]`
).join('\n');

return [{
  json: {
    ...$input.item.json,
    productContext: productList
  }
}];
```

Then in the AI Agent system prompt, add at the end:
```
Available products:
{{ $json.productContext }}
```

### Option B: Include Products in System Prompt (Small Catalog Only)

If you have fewer than 20 products, paste them directly in the system prompt:
```
Available products:
1. Classic Black T-Shirt - ৳1200 (Size: M,L | Color: Black) - In Stock
2. Slim Fit Jeans - ৳2500 (Size: 30,32,34 | Color: Blue, Black) - In Stock
3. Hoodie Pullover - ৳2000 (Size: M,L,XL | Color: Grey, Black) - In Stock
...
```

---

## Testing After Fix

Run these tests in Messenger (or n8n test chat):

| # | Send This | Should NOT Happen | Should Happen |
|---|-----------|-------------------|---------------|
| 1 | "ami t shirt khujtechi" | Long English paragraph | Short Bangla product list |
| 2 | "only ei tshirt available?" | "EI category", "Electronics" | "হ্যাঁ, এখন এই T-shirt টাই available..." |
| 3 | "okay" | "Hello! How can I assist..." | Continue context, ask size/color |
| 4 | "black hoodie M size order korte chai" | Ask size or color again | Ask only phone + address |
| 5 | "hoodie dekhao" | Start order flow | Show hoodie options |
| 6 | "eta nibo" | Show products | Start order (strong intent) |
| 7 | "accha" | Reset/greeting | Continue previous context |

---

## Model Recommendation

| Model | Bangla Quality | Cost | Recommendation |
|-------|---------------|------|----------------|
| GPT-4o | Excellent | Higher | Best for Bangla/Banglish understanding |
| GPT-4o-mini | Good | Low | Good balance of cost and quality |
| GPT-3.5-turbo | Average | Lowest | May struggle with "ei/eta" context |

**Recommendation:** Use **GPT-4o-mini** minimum. If budget allows, **GPT-4o** gives best Bangla understanding.

---

## Troubleshooting

### Bot still resets context
→ Check Session ID is properly set and unique per user
→ Increase context window to 15+
→ Make sure memory node is actually CONNECTED to AI Agent

### Bot still says "EI category"  
→ Make sure you replaced the ENTIRE old system prompt
→ Check no other node is adding instructions that override
→ Verify model is GPT-4o-mini or above

### Bot gives long English replies
→ System prompt not updated properly
→ Check for any "You are a helpful assistant..." default text remaining

### Order triggers too easily
→ Check the strong intent phrase list in the prompt
→ Add more weak words to the "must NOT trigger" list if needed
