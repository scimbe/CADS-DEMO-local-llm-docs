---
title: Local LLM Demo — Docs
description: A local-GPU LLM, exposed through a real tunnel, with real access control.
---

# Local LLM Demo

**CADS-DEMO-local-llm** ([GitHub](https://github.com/scimbe/CADS-DEMO-local-llm)) exposes a
local-GPU model — `qwen3-coder:30b`, served by Ollama on a single NVIDIA TITAN RTX (24GB VRAM) —
to the internet through a real [CADS-Tunnel](https://github.com/scimbe/CADS-Tunnel) agent,
fronted by [LiteLLM](https://github.com/BerriAI/litellm) for per-person API keys and an
OpenAI/Anthropic-compatible surface.

The live endpoint is **[llm-34a13a96.bunsenbrenner.org](https://llm-34a13a96.bunsenbrenner.org)**
— a real, running proxy to real local GPU inference, not a mockup. Every example response in
this documentation is copied from an actual call against it.

## Start here

- [Call the API and point Claude Code at it]({{ '/tutorials/first-call-and-claude-code/' | relative_url }}) —
  get a scoped key, make a real request, then point an actual coding-agent CLI at the same
  endpoint and see how its behavior differs from a plain API call.

## Sections

- **[Tutorials]({{ '/tutorials/' | relative_url }})** — learn by doing, start to finish.
- **[How-to guides]({{ '/how-to/' | relative_url }})** — accomplish a specific task.
- **[Reference]({{ '/reference/' | relative_url }})** — look up an exact fact (models, endpoints, env vars).
- **[Explanation]({{ '/explanation/' | relative_url }})** — understand why this is built this way.
