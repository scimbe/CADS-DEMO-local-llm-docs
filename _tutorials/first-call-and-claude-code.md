---
title: Call the API and point Claude Code at it
description: Get a scoped key, make a real request, then point Claude Code at the same endpoint and watch its behavior change.
order: 1
---

# Call the API and point Claude Code at it

Every command and response below is real — copy-pasted from an actual session against the live
endpoint at `https://llm-34a13a96.bunsenbrenner.org`, not paraphrased.

## 1. Get a scoped key

Ask the operator to generate one (or, if you hold the LiteLLM master key yourself):

```bash
curl -s http://127.0.0.1:4001/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" -H "Content-Type: application/json" \
  -d '{"models": ["local-qwen3-coder"], "user_id": "you@example.com", "key_alias": "llm-demo-you"}'
```

You get back a key like `sk-JHktR99hbpiHhwVVgT5fzQ`, scoped to exactly one model. Try it against
a model it's *not* allowed to use and confirm it's rejected — this is the access-control guarantee
the whole setup rests on:

```bash
curl -s https://llm-34a13a96.bunsenbrenner.org/v1/chat/completions \
  -H "Authorization: Bearer sk-JHktR99hbpiHhwVVgT5fzQ" -H "Content-Type: application/json" \
  -d '{"model":"cf-llama-70b","messages":[{"role":"user","content":"hi"}]}'
```

```json
{"error":{"message":"key not allowed to access model. This key can only access models=['local-qwen3-coder']. Tried to access cf-llama-70b","type":"key_model_access_denied","param":"model","code":"401"}}
```

## 2. Make a real call

```bash
curl -s https://llm-34a13a96.bunsenbrenner.org/v1/chat/completions \
  -H "Authorization: Bearer sk-JHktR99hbpiHhwVVgT5fzQ" -H "Content-Type: application/json" \
  -d '{"model":"local-qwen3-coder","messages":[{"role":"user","content":"Reply with exactly the word: pong"}],"max_tokens":20}'
```

```json
{"id":"chatcmpl-3980139d-136f-404f-a82b-aaf5165425a9","created":1786787743,"model":"local-qwen3-coder","object":"chat.completion","choices":[{"finish_reason":"stop","index":0,"message":{"content":"Hi!","role":"assistant"}}],"usage":{"completion_tokens":3,"prompt_tokens":18,"total_tokens":21}}
```

That's a real request, over the real tunnel, hitting real local GPU inference (`qwen3-coder:30b`
on the origin's TITAN RTX). Round-trip latency for a short reply like this is typically well under
a second — see `capability-probe/transcripts/` in the code repo for a wider sample.

## 3. Point Claude Code at it

The endpoint also answers Anthropic's `/v1/messages` shape (LiteLLM translates it), so Claude Code
itself can talk to it directly:

```bash
ANTHROPIC_BASE_URL="https://llm-34a13a96.bunsenbrenner.org" \
ANTHROPIC_AUTH_TOKEN="sk-JHktR99hbpiHhwVVgT5fzQ" \
CLAUDE_CODE_DISABLE_UNKNOWN_MODEL_WINDOW_ENFORCEMENT=1 \
  claude -p "Reply with exactly the word: pong" --model local-qwen3-coder
```

Two real things happened here that are worth pausing on:

- `CLAUDE_CODE_DISABLE_UNKNOWN_MODEL_WINDOW_ENFORCEMENT=1` is required because Claude Code doesn't
  recognize `local-qwen3-coder` as a known model and refuses to guess its context window without
  it.
- The response wasn't the plain word "pong". Claude Code sends its full system prompt and tool
  definitions along with every request — including whatever this session's own `CLAUDE.md`
  instructions are. Faced with the literal instruction "reply with exactly the word: pong" *plus*
  an elaborate standing instruction set, this model chose to act on the standing instructions
  instead of the direct question — it proposed a tool call rather than answering directly. Ask the
  same literal question over the plain API (step 2) and it just answers. **Same model, different
  harness, different behavior** — see [Why the harness changes the answer]({{ '/explanation/harness-shapes-behavior/' | relative_url }}).

## What you've verified

- A scoped API key that can't touch other models on the proxy.
- A real chat completion from a real local GPU, over a real public tunnel.
- Claude Code itself successfully redirected at that same endpoint — and a first, real
  demonstration of why "does the API work" and "does this behave the way you expect when driven by
  a specific tool" are different questions.

Next: [add another authorized person]({{ '/how-to/add-an-authorized-person/' | relative_url }}) or
read the [model and endpoint reference]({{ '/reference/models-and-endpoints/' | relative_url }}).
