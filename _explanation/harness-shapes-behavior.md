---
title: Why the harness changes the answer
description: The same model gave three different real answers to the same literal question, depending on what was driving it.
order: 1
---

# Why the harness changes the answer

`qwen3-coder:30b` never changed. Its weights, quantization, and the request's actual content
(`"Reply with exactly the word: pong"`) were identical across three tests in `cli-tools/`. What
changed was the *harness* — the system prompt, tool definitions, and standing instructions each
CLI wraps around the raw model call before it ever reaches the API.

## What actually happened, verified

| Driven by | What it did |
|---|---|
| Plain `curl` to `/v1/chat/completions` | Answered the question directly. |
| Claude Code | Proposed a `Write` tool call, writing "pong" to a file. |
| opencode | Proposed a `bash` tool call, running `echo pong`. |
| Codex CLI | Proposed an `exec_command` tool call. |

Every one of these is a real, captured response — see `cli-tools/results.md` in the code repo.
None of it is invented for effect.

<p class="audited"><span class="prov a">audited</span> These four behaviours are captured transcripts the maintainer recorded (<code>cli-tools/results.md</code> in the code repo), quoted verbatim — not re-run for this page.</p>

## Why this happens

Each of these three CLIs is an *agentic* harness: their system prompts describe an environment
with tools available and instruct the model to use them to accomplish tasks, not just answer
questions in prose. A model that's been fine-tuned to be agent-friendly (as coding-specialized
models like `qwen3-coder` generally are) will lean toward *acting* — reaching for a tool — even
when the literal, simplest reading of the request was a one-word reply. Claude Code's case is the
most pointed example: this very session's own `CLAUDE.md` carries a large, standing instruction
set about spawning agents and orchestrating swarms for "any non-trivial task." Faced with that
context plus a simple direct question, the model followed the standing instructions rather than
the literal question.

This isn't a bug in the model, and it isn't unique to `qwen3-coder` — it's the same phenomenon
[CADS-DEMO-sort](https://github.com/scimbe/CADS-DEMO-sort) is built around from a different angle:
*what wraps the model* shapes the outcome as much as, or more than, the model itself.

## The practical takeaway

Before trusting "the model can't do X" or "the model always does Y", check what's actually calling
it. A plain API call and a request routed through an agentic coding CLI are not the same
experiment, even with identical wording and an identical model underneath.
