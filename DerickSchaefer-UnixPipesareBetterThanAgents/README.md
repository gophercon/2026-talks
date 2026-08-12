# Unix Pipes are Better Than Agents

**Derick Schaefer**

## Abstract

The industry is racing to give LLMs more tools, APIs, and agent frameworks. Reserve, a Go CLI for exploring FRED macroeconomic data, takes the opposite approach: enabling the tool to explain itself. Every command can expose context, intent, examples, and pipeline patterns, allowing models to discover functionality, compose deterministic workflows, and explain exactly how a result was produced.

Rather than letting an agent reason over commands, flags, and raw APIs, Reserve pushes collection, transformation, and analysis into deterministic, reproducible Unix pipelines, preserving precious tokens for insight rather than orchestration. The outcome is simpler, more transparent, and surprisingly powerful.

The industry keeps asking how much intelligence can be embedded in upstream orchestration, while the Unix philosophy has spent the last fifty years moving orchestration into composition.

**Talk/Attendee Level:** All Attendees

## Materials

- [Slides (PDF)](gc2026_unix-pipes-agents.pdf)
- [PowerPoint](gc2026_unix-pipes-agents.pptx)
- [Reserve repository](https://github.com/derickschaefer/reserve)
- [Reserve website](https://reservecli.dev/)
- [RaaS architectural paper/repository](https://github.com/derickschaefer/raas)
