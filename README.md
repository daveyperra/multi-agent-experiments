# Multi-Agent Experiments

Experiments in multi-model AI coordination -- skills, debates, and cross-model verification workflows.

## Why This Repo

Single-model AI has a fundamental limitation: it can't catch its own blind spots. These experiments explore what happens when you make frontier models work *together* -- challenging each other, verifying claims, and synthesizing positions that neither would reach alone.

The focus is on practical, high-stakes use cases where being wrong has real cost: investment decisions, career moves, major purchases, and life strategy.

## Experiments

### [Gemini Sparring Partner](./gemini-sparring-partner/)

A Claude skill that uses browser automation to pit Claude against Google Gemini on consequential decisions. Claude forms a position, crafts a structured adversarial prompt, submits it to Gemini via the browser, and synthesizes a comparison showing where the models agree vs. diverge.

Key features:
- **Browser-based** -- no API keys, uses your existing Gemini subscription via Claude in Chrome
- **Structured adversarial prompting** -- forces genuine pushback, not lazy consensus
- **Multi-round debate with epistemic honesty** -- Claude accepts factual corrections from Gemini instead of doubling down
- **Background tab throttling workaround** -- Web Worker keep-alive for Chrome's tab throttling

[Full documentation and install instructions ->](./gemini-sparring-partner/README.md)

## Author

Built by [Davey Perra](https://github.com/daveyperra)

## License

MIT
