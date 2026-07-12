---
name: hermes-custom-providers
description: "Configure custom OpenAI-compatible API providers in Hermes Agent — local LLM endpoints, API gateways like 9router, and any OpenAI-compatible backend."
version: 1.0.0
author: agent
metadata:
  hermes:
    tags: [hermes, providers, custom, 9router, configuration, model-switching, openai-compatible]
    related_skills: [hermes-agent]
---

# Hermes Custom Provider Setup

Configure Hermes Agent to use any OpenAI-compatible API endpoint as a provider — including local LLM servers, API routers like 9router, or private model hosts.

## When to Use This Skill

- You installed an AI gateway (9router, OpenRouter, etc.) and want Hermes to use it
- You run a local model server (vLLM, llama.cpp, Ollama) and want to connect Hermes
- You need to switch between multiple models from the same custom endpoint
- You're using a combo/aggregator model that routes to multiple backends
- You need to create a domain-specific Hermes profile (see `references/profile-creation-recipe.md`)

## Setup

### 1. Configure the Provider

Set the endpoint URL in Hermes config:

```bash
hermes config set model.base_url "http://127.0.0.1:20128/v1"
hermes config set model.provider custom
hermes config set model.default comboku
```

Using 9router on localhost port 20128 (default port). Change the URL for other providers.

### 2. Define Available Models (Optional but Recommended)

Tell Hermes which models exist so `/model` listing works:

```bash
hermes config set providers.custom.base_url "http://127.0.0.1:20128/v1"
hermes config set providers.custom.model_list '["model-1","model-2","model-3"]'
```

The `model_list` tells Hermes which models are available for switching. Without it, Hermes may not list models properly in `/model` or the model picker.

### 3. Verify Connection

```bash
# List models from the endpoint
curl -s http://127.0.0.1:20128/v1/models | jq '.data[] | .id' | sort

# Test a quick chat
hermes chat -m "<model-id>" -q "Hello"
```

## Model Switching Strategies

### Preferred: In-Session Switching

Switch models within your current profile — no profile hopping, no app restart needed.

```bash
# From terminal — changes take effect on next /reset in Desktop
hermes config set model.default <model-id>
```

In Hermes Desktop, type `/reset` to start a fresh session with the new model.

### Alternative: Profile-Based

Each profile has its own model config. Run `hermes profile create <name> --clone-from default` then set a different model. This launches a **separate** Hermes instance with that model.

**Use profiles when:**
- You want permanently different toolkits, skills, or memory per environment
- You're running multiple Hermes instances simultaneously

**Do NOT use profiles when:**
- You just want to try a different model for the current task (profile-hopping is overkill)
- You want to stay in one chat session and change models on the fly

### One-Shot Query (No Config Change)

Temporary model override for a single query without touching config:

```bash
hermes chat -m "<model-id>" -q "Your question here"
```

## Understanding 9Router Combo Models

9Router (and similar gateways) supports **combo models** — a virtual model that internally routes between multiple real backends with automatic fallback.

### How Combos Work

A combo like `comboku` has a defined list of sub-models and routes requests based on:
1. **Tier 1 — Subscription**: Claude Code, OpenAI Codex, Gemini (paid/OAuth)
2. **Tier 2 — Cheap**: GLM ($0.60/1M), MiniMax ($0.20/1M)
3. **Tier 3 — FREE**: iFlow, Qwen, Kiro, OpenRouter free tier, NVIDIA NIM

When Tier 1 is exhausted (quota or rate limit), it falls back to Tier 2, then Tier 3.

### Combo vs Individual Models

| Aspect | Combo (`comboku`) | Individual (`kr/deepseek-3.2`) |
|--------|-------------------|-------------------------------|
| Routing | Auto-fallback between 13+ models | Direct to one provider |
| Resilience | Survives individual provider outages | Fails if that provider is down |
| Predictability | Unpredictable which model answers | Always the same model |
| Rate limits | Combines quotas across providers | Only that provider's quota |
| Switching | Internal — you cannot pick sub-model | You choose the model |

### Available Models (9Router Example)

Individual models accessible directly:
- `kr/claude-sonnet-4.5` — Claude Sonnet 4.5
- `kr/claude-sonnet-4.5-thinking` — with reasoning
- `kr/claude-sonnet-4.5-agentic` — autonomous mode
- `kr/claude-haiku-4.5` — Claude Haiku 4.5 (fast/cheap)
- `kr/deepseek-3.2` — DeepSeek 3.2 (strong at coding)
- `kr/deepseek-3.2-thinking` — with reasoning
- `kr/glm-5` — GLM 5
- `kr/glm-5-thinking` — with reasoning
- `kr/minimax-m2.5` — MiniMax M2.5
- `kr/minimax-m2.5-thinking` — with reasoning
- `kr/minimax-m2.1` — MiniMax M2.1
- `kr/qwen3-coder-next` — Qwen3 Coder Next
- `kr/qwen3-coder-next-thinking` — with reasoning
- `kr/auto` — Auto-select best available
- `kr/auto-thinking` — Auto-select with reasoning

Full list: `curl -s http://localhost:20128/v1/models | jq '.data[] | .id' | sort`

## Troubleshooting

### `/model` Command Shows No Models

**Cause:** Hermes may not auto-discover models from a `custom` provider endpoint. The `/v1/models` API works but the slash command may not list them.

**Fixes:**
1. Set `providers.custom.model_list` explicitly (see Step 2 above)
2. Type the model ID directly: `/model kr/deepseek-3.2` (not just `/model`)
3. From terminal: `hermes config set model.default <model-id>` then `/reset` in Desktop

### Rate Limits on Individual Models

Models accessed directly (e.g. `kr/claude-sonnet-4.5`) have their own rate limits. If you hit a quota error, switch to another model or use the combo (`comboku`) which auto-fallsback.

```bash
hermes config set model.default comboku
```

### Provider Not Responding

```bash
# Check endpoint is running
curl -s http://localhost:20128/v1/models | head -5

# If 9router dashboard is accessible
curl -s http://localhost:20128/dashboard | head -20

# Restart 9router
pkill -f 9router && 9router
```

## Pitfalls

- **Don't create profiles per model** — it's unnecessary overhead. Use in-session switching instead.
- **`hermes config set` serializes arrays as quoted strings** — ANY list/array value set via `hermes config set key '[a,b,c]'` gets stored as `'[a,b,c]'` (a quoted string), NOT a proper YAML list. This affects `model_list`, `disabled_toolsets`, `platform_toolsets.cli`, and every other array field. Hermes may JSON-parse some of them at runtime, but others break silently. **Fix:** after using `config set` for arrays, always verify with `search_files` and `patch` the config.yaml to convert quoted strings into proper YAML list syntax (`- item` per line). Or skip `config set` entirely for arrays and use `patch` on the raw YAML from the start.
- **Combo models hide sub-models** — you cannot access individual models inside a combo by adding `/model` while using the combo. Switch to the individual model ID directly.
- **Custom provider needs an api_key field** — some providers require `model.api_key` even for free endpoints. 9router doesn't, but set it to `""` (empty string) if a provider complains.
- **Profile delete requires confirmation** — `hermes profile delete <name>` prompts for the profile name as confirmation. Use `yes <name> | hermes profile delete <name>` for non-interactive deletion.
- **Desktop app requires restart** — model changes via `hermes config set` need the Desktop app to be restarted (or use `/reset` in the chat session).
