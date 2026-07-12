# 9Router Model Discovery

When using 9Router as a local AI gateway (http://localhost:20128), you can query available models and combos via the OpenAI-compatible API endpoint.

## Problem

The 9Router dashboard at `http://localhost:20128/dashboard/combos` requires authentication and cannot be accessed via `web_extract` (blocked: private network address). Direct curl attempts may redirect to `/login`.

## Solution

Use the OpenAI-compatible `/v1/models` endpoint to list all available models and combos:

```bash
curl -s http://localhost:20128/v1/models | jq '.'
```

## Listing Models

### Get all model IDs (sorted)
```bash
curl -s http://localhost:20128/v1/models | jq '.data[] | .id' | sort
```

### Count total models
```bash
curl -s http://localhost:20128/v1/models | jq '.data | length'
```

### Filter by provider (e.g., kr/ prefix for Kiro)
```bash
curl -s http://localhost:20128/v1/models | jq '.data[] | select(.id | startswith("kr/")) | .id'
```

## Model Categories

9Router typically exposes:

1. **Combo models** (e.g., `comboku`) - Multi-provider routing with automatic fallback
2. **Individual provider models** - Direct access to specific providers with variants:
   - Base model (e.g., `kr/claude-sonnet-4.5`)
   - Thinking mode (`-thinking` suffix)
   - Agentic mode (`-agentic` suffix)
   - Combined mode (`-thinking-agentic` suffix)
   - Auto mode (`kr/auto` - automatic model selection)

## Common Providers in 9Router

- **Claude** (Anthropic): Sonnet 4.5, Sonnet 4, Haiku 4.5
- **DeepSeek**: 3.2
- **GLM** (Z.AI): 5
- **MiniMax**: M2.5, M2.1
- **Qwen**: Qwen3 Coder Next

## Usage Pattern

When the user mentions using 9router.com or has `comboku` as their active model:

1. Query available models via `/v1/models` endpoint
2. Parse the response to show available options
3. User can switch between combo routing and specific providers
4. Combo models provide 3-tier fallback (Subscription → Cheap → FREE)

## Example Response Structure

```json
{
  "object": "list",
  "data": [
    {
      "id": "comboku",
      "object": "model",
      "owned_by": "combo"
    },
    {
      "id": "kr/claude-sonnet-4.5",
      "object": "model",
      "owned_by": "kr"
    }
  ]
}
```

## Configuration

The user's Hermes config likely uses:
- Provider: `custom:comboku` or similar
- Base URL: `http://localhost:20128/v1`
- Model: `comboku` (or specific model ID)

## Related

- 9Router docs: https://9router.com
- 9Router GitHub: https://github.com/decolua/9router
