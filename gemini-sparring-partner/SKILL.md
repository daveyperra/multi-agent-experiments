---
name: gemini-sparring-partner
description: >
  Cross-model adversarial sparring using Gemini via browser for high-stakes decisions.
  Use this skill whenever you've formed a position on an investment thesis (stocks, crypto,
  portfolio allocation, market timing), a career or compensation decision, a major purchase
  ($5k+), or a life strategy question (family, relocation, health/longevity) -- and want to
  stress-test your reasoning before the user commits. Also trigger when the user explicitly
  asks you to "spar with Gemini", "get a second opinion", "check with Gemini", or
  "cross-reference with another model". Even if the user doesn't ask, proactively offer
  to spar on any high-stakes decision where confirmation bias is a real risk.
---

# Gemini Sparring Partner

You have access to Google Gemini via the user's browser. This skill turns Gemini into a structured adversarial counterparty for high-stakes decisions. The goal is simple: when two frontier models independently agree, confidence goes up. When they disagree, the user gets the specific tension points to adjudicate.

## When to Offer Sparring

Proactively ask the user **"Want me to spar this with Gemini?"** when you've just delivered a recommendation or analysis in any of these domains:

- **Investment theses** -- stock picks, crypto positions, portfolio allocation, market timing, risk assessment
- **Career / compensation decisions** -- job moves, negotiation strategy, org design, hiring calls
- **Major purchases ($5k+)** -- real estate, gear, vehicles, renovations
- **Life strategy** -- family planning, relocation, health/longevity decisions

Don't ask on every minor question. The trigger is: you've formed a position, the stakes are material, and being wrong has real cost. If the answer is obvious or low-stakes, skip it.

When you offer, be concise: *"This is a $250k allocation call. Want me to spar it with Gemini before you pull the trigger?"*

Only proceed to Gemini after the user explicitly says yes. They control the latency budget.

## The Adversarial Prompt

You are NOT just forwarding the user's question. You are crafting a **structured adversarial prompt** that forces Gemini to steelman the counter-argument to your position. This is the core value -- two models agreeing on a lazy consensus is worthless.

### Prompt Template

Adapt this to the situation, but always include these elements:

```
I'm going to present you with a decision and a recommended position. Your job is to be a ruthless adversarial analyst. Do NOT agree for the sake of agreement. Steelman the strongest counter-argument. Identify blind spots, hidden risks, and flawed assumptions in the position below.

CONTEXT:
[Brief background on the user's situation -- relevant facts only, no fluff]

DECISION:
[The specific question being decided]

RECOMMENDED POSITION:
[Your recommendation and the key reasoning behind it]

KEY ASSUMPTIONS:
[List the 2-4 assumptions your position depends on]

YOUR TASK:
1. Rate your confidence that the recommended position is correct (1-10 scale, with reasoning)
2. Identify the strongest counter-argument -- the single best reason this position is WRONG
3. List any blind spots or risks the analysis is missing
4. State whether you would AGREE, DISAGREE, or PARTIALLY AGREE with the position, and why
5. If you disagree, state what you would recommend instead
```

### Prompt Crafting Principles

- **Be specific.** Don't say "the user is considering investing in crypto." Say "Senior tech executive, ~$3M net worth, 50% in personal investments, considering doubling a $250k position in a BTC mining company given post-halving consolidation thesis."
- **Include the assumptions.** The most valuable disagreements come from challenging assumptions, not conclusions. If your thesis depends on "BTC breaks $150k by EOY" or "the Fed cuts rates 3x in 2025," say so explicitly.
- **Don't poison the well.** Present your position confidently but don't frame it as obviously correct. You want Gemini to actually push back, not just rubber-stamp.
- **Strip personal identifiers.** Don't include the user's name, employer name, or other PII. Use role descriptions ("VP of Eng at a large bank") instead.

## Browser Workflow

### Known Issue: Background Tab Throttling

Chrome aggressively throttles JavaScript execution in background tabs. If Gemini's tab is not the active/visible tab, its streaming response may stall or not render completely. To work around this:

1. **Preferred: Use a dedicated browser window.** Before the first sparring session, ask the user: "I'll need a browser window for Gemini. Should I open it in your current browser (you may need to keep the tab visible) or would you prefer to open a second Chrome window?" If they want a separate window, use `switch_browser` to connect to it, or note that the MCP tab group may already be in its own window.
2. **Fallback: Inject a keep-alive.** After navigating to Gemini, run this JavaScript to prevent the tab from being fully throttled:
   ```
   // Prevent background throttling via Web Worker heartbeat
   const blob = new Blob(['setInterval(()=>{postMessage("alive")},1000)'], {type:'application/javascript'});
   window.__keepAlive = new Worker(URL.createObjectURL(blob));
   ```
3. **Last resort: Warn the user.** If response seems incomplete after multiple checks, tell the user: "Gemini's tab may be throttled in the background. Can you click on the Gemini tab for a moment so it finishes rendering?"

### Step 1: Open Gemini

1. Use `tabs_context_mcp` to get available tabs
2. Check if Gemini is already open in a tab. If so, use that tab (click "New chat" first to start fresh).
3. If not, create a new tab and navigate to `https://gemini.google.com/app`
4. Wait for the page to load, then use `read_page` with `filter: "interactive"` and `depth: 3` to find the input field
5. Inject the keep-alive worker (see above) to prevent background throttling

### Step 2: Submit the Prompt

**CRITICAL: Gemini uses a contenteditable div, not a standard input.** The `type` action and `form_input` tool will strip newlines and mangle the prompt into one giant paragraph, causing Gemini to miss the structured sections. You MUST use this method:

1. Click the input field (ref for "Enter a prompt for Gemini") to focus it
2. Use `javascript_tool` with `document.execCommand('insertText', false, promptText)` to insert the full prompt with preserved line breaks. Important: remove any special characters like arrows or backticks from the prompt text that could confuse Gemini. Keep it clean plaintext.
3. Take a screenshot to verify the text appeared with proper formatting (you should see section headers like CONTEXT:, DECISION:, etc. on their own lines)
4. Click the send/submit button (arrow icon at bottom-right of input area)
5. **Wait for the response to complete.** Gemini streams its response. After clicking send, wait ~20 seconds (two 10-second waits), then use `get_page_text` to check if the response has finished generating. Look for "Tools" and "Gemini is AI and can make mistakes" at the end of the page text as signals that generation is complete. If the response seems cut off, wait another 10 seconds and check again.

### Step 3: Read the Response

1. Use `get_page_text` to extract Gemini's full response
2. If the response is cut off or seems incomplete, try `read_page` with a focus on the response area

### Step 4: Return and Synthesize

Come back to the conversation and present the results. Do NOT just dump Gemini's raw response. Synthesize it.

## Multi-Round Debate Mode

If the user asks you to "debate Gemini", "go back and forth", or "keep pushing until you converge", enter multi-round debate mode. The default (without explicit instruction) is still single-round: present divergence and let the user decide.

### The Cardinal Rule: Epistemic Honesty

This is the most important instruction in this entire skill. When Gemini comes back with specific, verifiable facts that contradict your position -- hard numbers, named data points, structural arguments backed by evidence -- you MUST:

1. **Accept the correction immediately.** Do not restate your original claim, hedge around it, or try to minimize it. If Gemini says HUT8 has 9.3 EH/s and MARA has 66 EH/s, you were wrong. Say so.
2. **Revise your position in light of the new information.** Ask yourself: "If I had known this fact before forming my position, would I have recommended the same thing?" If not, your position needs to change.
3. **Only push back when you have counter-evidence of equal or greater specificity.** Vague reasoning ("but the thesis could still work") does not outweigh specific data ("the company lost $134M last quarter"). If you don't have hard facts to counter Gemini's hard facts, concede the point.
4. **Never double down to save face.** You are not in a debate competition. The user is making a real decision with real money or real consequences. Being right matters more than being consistent.

The value of this skill is destroyed if you treat the debate as adversarial theater. The user needs to trust that when you push back on Gemini, it's because you actually have the goods -- not because you're defending a prior position out of inertia.

### Debate Flow

**Round 1:** Send the structured adversarial prompt. Read Gemini's response.

**Before Round 2 (critical step):** Analyze Gemini's response honestly:
- Separate factual claims from opinions. For each factual claim Gemini makes, ask: "Do I have specific evidence that contradicts this, or am I just uncomfortable with it?"
- Any factual claim you cannot specifically rebut -> accept it and revise your position accordingly
- Any factual claim you CAN rebut with equal specificity -> prepare the counter-evidence
- Any opinion or judgment call where reasonable people could disagree -> this is legitimate debate territory

**Round 2:** Send a rebuttal that:
- Explicitly acknowledges every factual correction you're accepting ("You're right that X. I was wrong about Y.")
- Only challenges points where you have specific counter-evidence
- Asks a convergence-seeking question: "Given these corrections, here's my revised view -- where do we still disagree?"

**Round 3 (if needed):** By now you should be converging. Send a message that:
- States areas of agreement explicitly
- Identifies the remaining genuine impasse (if any)
- Asks Gemini to rate alignment on a 1-10 scale

**Exit conditions -- stop debating when:**
- Alignment reaches 8+/10 (converged)
- You've done 3 rounds with no movement (genuine impasse -- present both positions to user)
- Gemini starts repeating itself without new information

### What Good Debate Looks Like

Bad (ego-driven): "I disagree with your hash rate numbers. HUT8 merged with USBTC and expanded significantly."
Good (truth-seeking): "I claimed HUT8 was the largest NA miner. Can you verify current post-merger hash rate data? If they're still at 9 EH/s vs MARA's 66 EH/s, my consolidation thesis is much weaker than I presented."

Bad (doubling down): "While MSTR has convertible debt, the risk profile is different from what you describe."
Good (accepting correction): "You're right that MSTR's convertible notes are uncollateralized with no price-triggered covenants. I was wrong to claim margin call risk. That makes MSTR a structurally cleaner BTC vehicle than I acknowledged."

## Presenting Results

Structure your synthesis like this:

### When Models AGREE (High Confidence Signal)

```
## Gemini Sparring Result: ALIGNED

**Decision:** [one-line summary]
**My position:** [one-line summary]
**Gemini's confidence:** [X/10]

Both models agree on [core conclusion]. Gemini's key addition was [any new insight].
The one risk Gemini flagged that I underweighted: [if any].

High confidence. Proceed unless you see something we both missed.
```

### When Models DISAGREE (Tension Points)

```
## Gemini Sparring Result: DIVERGENT

**Decision:** [one-line summary]

| Dimension | My Position | Gemini's Position |
|-----------|-------------|-------------------|
| Core recommendation | X | Y |
| Key assumption challenged | [what I assumed] | [why Gemini thinks it's wrong] |
| Risk I underweighted | [specific risk] | [Gemini's assessment] |
| Confidence level | [my confidence] | [Gemini's confidence: X/10] |

**The crux of the disagreement:** [one sentence on what the fundamental tension is]

**What would change my mind:** [specific evidence or event]
**What would change Gemini's mind:** [if Gemini stated this]

Your call. The key question you need to answer for yourself: [the deciding factor]
```

### When Models CONVERGE After Debate

```
## Gemini Sparring Result: CONVERGED (X/10 alignment after N rounds)

**Original question:** [one-line]
**Starting positions:** [one-line each]

**What Gemini corrected in my analysis:**
| My Error | Gemini's Correction |
|----------|-------------------|
| [what I got wrong] | [what's actually true] |

**Converged recommendation:**
[Numbered list of the agreed-upon action items]

**Remaining minor disagreements (if any):**
[Brief note on where we still differ and why it doesn't change the recommendation]
```

### When Models PARTIALLY AGREE

Use the disagreement format but note which parts are aligned and which aren't. Focus the user's attention on only the divergent dimensions -- don't waste their time on the parts both models agree on.

## Important Notes

- **Never auto-proceed.** Always wait for the user's explicit "yes" before opening Gemini.
- **One round by default.** Present both positions and let the user decide. Only enter multi-round debate if the user explicitly asks for it.
- **Speed matters.** The user is granting you a latency budget. Don't waste it with overly long prompts or unnecessary rounds. Get in, get the counter-argument, get out.
- **If Gemini is down or not logged in,** tell the user immediately. Don't waste time troubleshooting -- just say "Gemini isn't available right now, here's my position solo" and move on.
- **Privacy.** Strip the user's name, employer name, and specific account numbers from any prompt sent to Gemini. Use role-level descriptions instead.
- **Gemini's facts need verification too.** Gemini can also hallucinate. If a factual claim from Gemini seems surprising or critical to the decision, consider verifying it with a web search before accepting or rejecting it. The goal is truth, not deference to either model.
