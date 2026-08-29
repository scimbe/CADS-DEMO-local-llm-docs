---
title: Add an authorized person
description: Give someone their own scoped API key without touching anyone else's access.
order: 1
---

# Add an authorized person

Access is per-person, via LiteLLM virtual keys — not a shared password, and not the platform's
own SSO login-gate (that gate is cookie/session-based and would block plain API/CLI callers, so
it's deliberately left off for this tunnel). <span class="prov n">not built</span>

## Generate a key

```bash
curl -s http://127.0.0.1:4001/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" -H "Content-Type: application/json" \
  -d '{
    "models": ["local-qwen3-coder"],
    "user_id": "<their email>",
    "key_alias": "llm-demo-<their name>",
    "metadata": {"purpose": "llm.bunsenbrenner.org local model demo", "authorized_email": "<their email>"}
  }'
```

The response includes `"key": "sk-..."` — hand that to them directly (it's shown once; LiteLLM
stores only a hash). The `models` array is the actual enforcement: this key will 401 on every
model on the proxy except `local-qwen3-coder`, including the cloud-routed ones this same LiteLLM
instance also fronts.

<p class="measured"><span class="prov m">measured</span> Enforcement reproduced 2026-08-29 with the example key from the tutorial: it is refused for other models. The refusal now returns HTTP <code>403</code> / <code>key_model_access_denied</code>; earlier captures in these docs show <code>401</code>.</p>

## Revoke one

```bash
curl -s http://127.0.0.1:4001/key/delete \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" -H "Content-Type: application/json" \
  -d '{"keys": ["sk-the-key-to-revoke"]}'
```

## List everyone with access

```bash
curl -s "http://127.0.0.1:4001/key/list?user_id=<their email>" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

These calls only work from the origin host (`127.0.0.1:4001` isn't exposed through the tunnel) —
that's deliberate: key management stays off the public endpoint entirely.
