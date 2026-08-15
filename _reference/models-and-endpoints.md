---
title: Models and endpoints
description: The exact model name, endpoints, and env vars in this deployment.
order: 1
---

# Models and endpoints

## Base URL

```
https://llm-34a13a96.bunsenbrenner.org
```

## Model

| LiteLLM model name | Backing model | Where it runs |
|---|---|---|
| `local-qwen3-coder` | `qwen3-coder:30b` (MoE, ~3.3B active params, Q4_K_M, ~18GB on disk) | Ollama, this origin's NVIDIA TITAN RTX (24GB VRAM) |

Other model names configured on this same LiteLLM instance (`cf-llama-70b`, `cf-llama-free`, and
the `*:cloud` fallback chain) are cloud-routed and **not accessible** to keys scoped to
`local-qwen3-coder` — see [Add an authorized person]({{ '/how-to/add-an-authorized-person/' | relative_url }}).

## Endpoints

| Path | Shape | Notes |
|---|---|---|
| `POST /v1/chat/completions` | OpenAI | Standard chat completions. |
| `POST /v1/messages` | Anthropic | LiteLLM's Anthropic-format passthrough; what Claude Code talks to. |
| `GET /health/liveliness` | — | No auth required; used by the tunnel's own health checks. |

## Auth

`Authorization: Bearer sk-...` (OpenAI-shaped calls) or `x-api-key: sk-...` +
`anthropic-version: 2023-06-01` (Anthropic-shaped calls). Same key works for both.

## Known deployment-specific quirks

- `drop_params: true` is set proxy-wide, because Claude Code sends a `context_management`
  parameter Ollama's backend doesn't understand; without it every Claude-Code-originated request
  fails with a 400.
- The model's context window is advertised as 256K by its card, but this deployment has not raised
  Ollama's default `num_ctx` — treat anything past roughly a few thousand tokens of real context
  with suspicion. See `capability-probe/transcripts/05-long-context-needle.json` in the code repo
  for the exact failure.
