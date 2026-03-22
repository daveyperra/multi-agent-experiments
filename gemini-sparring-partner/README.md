# Gemini Sparring Partner

A Claude skill that uses browser automation to pit Claude against Google Gemini on high-stakes decisions -- investments, career moves, major purchases -- then synthesizes where they agree, where they diverge, and what you should actually do.

## What It Does

When you're about to make a consequential decision, Claude:

1. **Forms its own position** on your question
2. **Asks if you want a second opinion** ("This is a $250k allocation call. Want me to spar it with Gemini?")
3. **Opens Gemini in your browser**, crafts a structured adversarial prompt, and submits it
4. **Reads Gemini's response** and synthesizes a comparison table showing alignment vs. divergence
5. **Optionally debates** -- goes back and forth with Gemini until they converge or reach a genuine impasse

The core insight: when two frontier models with different training data independently agree, your confidence should go up. When they disagree, you get the specific tension points to think through yourself.

## Why This Exists

LLMs are confident. Sometimes they're confidently wrong. A single model can't reliably catch its own blind spots -- it doesn't know what it doesn't know. But a *different* model, trained on different data with different biases, often catches exactly what the first one missed.

This skill automates the workflow of "let me go ask another AI what it thinks" and structures it so the output is actually useful -- not just two walls of text you have to mentally diff yourself.

### What Makes It Different

- **Browser-based, no API keys needed.** Uses your existing Gemini subscription via Claude in Chrome. No setup, no billing, no key management.
- **Structured adversarial prompting.** Doesn't just forward your question -- crafts a prompt that forces Gemini to steelman the counter-argument to Claude's specific position.
- **Epistemic honesty in multi-round debate.** When Gemini brings hard facts that contradict Claude's position, Claude accepts the correction and revises -- no ego-driven doubling down. This is explicitly enforced in the skill instructions.
- **User controls the latency budget.** Claude asks before spending the time. You decide when the second opinion is worth the wait.

## Example: Investment Thesis Stress Test

**User:** "I'm thinking about doubling my position in a BTC mining company."

**Claude forms position:** Cautious yes -- consolidation upside post-halving, but size conservatively.

**Claude opens Gemini, sends adversarial prompt, gets response:**

| Dimension | Claude | Gemini |
|-----------|--------|--------|
| Core call | Increase position modestly | Don't add a dollar |
| Key challenge | Position sizing risk | Miners are structurally inferior to spot BTC -- CapEx treadmill, dilution |
| Factual correction | -- | Company is 7x smaller than claimed vs. competitors |
| Blind spot surfaced | -- | Existing holdings are redundant crypto beta, not diversification |

**After 3 rounds of debate:** Models converge at 9/10 alignment. Both agree: hold current position, use spot BTC ETF for additional exposure, don't trim high-quality tech holdings to fund a miner.

The multi-round debate caught two factual errors in Claude's original analysis and produced a materially different (and better) recommendation than either model alone.

## Installation

### Prerequisites

- [Claude Desktop](https://claude.ai) with Cowork mode
- [Claude in Chrome](https://chromewebstore.google.com/detail/claude-in-chrome/) extension
- A Google Gemini subscription (free tier works, Pro/Ultra recommended)
- Logged into Gemini in your Chrome browser

### Install the Skill

Download `gemini-sparring-partner.skill` and install it in Claude.

## Configuration

The skill triggers automatically on high-stakes decisions in these domains:

- **Investment theses** -- stocks, crypto, portfolio allocation
- **Career / compensation** -- job offers, negotiation, org decisions
- **Major purchases ($5k+)** -- real estate, vehicles, renovations
- **Life strategy** -- relocation, family planning, health decisions

You can also trigger it manually by saying "spar this with Gemini" or "get a second opinion."

### Privacy

The skill automatically strips personal identifiers (name, employer, account numbers) before sending anything to Gemini. It uses role-level descriptions ("senior tech executive") instead.

## Known Limitations

- **Chrome background tab throttling.** If Gemini's tab isn't visible, Chrome may throttle its JavaScript and the response won't complete. The skill includes a Web Worker keep-alive workaround, but using a separate browser window is most reliable.
- **Gemini's contenteditable input.** Standard typing strips newlines. The skill uses `document.execCommand('insertText')` to preserve prompt formatting -- a workaround discovered through testing.
- **Both models can hallucinate.** The skill is designed so that neither model's claims are taken at face value -- but for truly critical decisions, verify surprising factual claims independently.
- **Latency.** Each round of sparring takes ~20-30 seconds (Gemini generation time). Multi-round debates can take 1-2 minutes total.

## How It Works (Technical)

1. Claude crafts an adversarial prompt using a structured template (CONTEXT, DECISION, RECOMMENDED POSITION, KEY ASSUMPTIONS, TASK)
2. Opens Gemini via `tabs_context_mcp` / `navigate` browser tools
3. Injects the prompt into Gemini's contenteditable div via `javascript_tool` + `document.execCommand('insertText')`
4. Waits for generation to complete (polls `get_page_text` for completion signals)
5. Extracts response via `get_page_text`
6. Synthesizes a structured comparison (ALIGNED / DIVERGENT / CONVERGED)
7. In debate mode: repeats with epistemic honesty rules -- accepts factual corrections, only pushes back with counter-evidence of equal specificity

## License

MIT

## Author

Built by [Davey Perra](https://github.com/daveyperra) -- exploring what's possible when frontier models argue with each other instead of just agreeing.
