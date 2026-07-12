# LM Studio Empty Stream with Reasoning Models

## Symptom
```
API call failed after 3 retries: Provider returned an empty stream with no finish_reason 
(possible upstream error or malformed SSE response).
```

**Context**: LM Studio running, model loaded, API accessible via curl, but Hermes times out.

## Root Cause: Reasoning Mode
Newer models (Qwen3.5, OpenAI o1/o3-mini) use **reasoning mode**:
- Stream sends `reasoning_content` first (10-30+ chunks of internal thinking)
- `content` field is empty or appears much later
- Hermes expects `content` in early chunks, times out waiting

### Response Pattern
**Non-streaming**:
```json
{
  "choices": [{
    "message": {
      "content": "",  // ❌ Empty
      "reasoning_content": "Let me think... The user asked..."  // ✅ Populated
    }
  }]
}
```

**Streaming**:
```
data: {"choices":[{"delta":{"reasoning_content":"Think"}}]}
data: {"choices":[{"delta":{"reasoning_content":"ing..."}}]}
... (10-30+ chunks)
data: {"choices":[{"delta":{"content":"Answer"}}]}  // Finally appears
data: {"choices":[{"delta":{},"finish_reason":"stop"}]}
```

Hermes reads the first chunks, sees no `content`, assumes empty stream, retries 3x, fails.

## Diagnosis

### 1. Check LM Studio is running and accessible
```bash
# List models
curl -s http://127.0.0.1:1234/v1/models | python3 -m json.tool

# Or port 3000 if configured differently
curl -s http://127.0.0.1:3000/v1/models | python3 -m json.tool
```

### 2. Test non-streaming response
```bash
curl -s http://127.0.0.1:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model":"your-model-name",
    "messages":[{"role":"user","content":"Say hi"}],
    "stream":false,
    "max_tokens":20
  }' | python3 -m json.tool
```

**Look for**:
- `"content": ""` (empty) + `"reasoning_content": "..."` (populated) = reasoning mode active
- `"content": "Hi!"` (populated) = normal mode, issue is elsewhere

### 3. Identify reasoning models
Common reasoning models:
- `qwen/qwen3.5-*` (all variants)
- `openai/o1-*`, `openai/o3-mini`
- DeepSeek-R1
- Models with "reasoning" or "r1" in the name

## Solutions

### Option 1: Disable Reasoning in LM Studio (Recommended)
1. Open LM Studio GUI
2. Select the loaded model
3. **Model Settings** → **Advanced**
4. Find "Reasoning Mode" or "Enable Reasoning" toggle
5. **Disable** it
6. Reload the model

### Option 2: Use Non-Reasoning Model
Load a different model without reasoning:
- Llama 3.1 / 3.3 (meta-llama/Llama-3.3-8B-Instruct)
- Mistral / Mixtral (mistralai/Mistral-7B-Instruct)
- DeepSeek-V3 (non-R1 variant)
- Phi-3/4 (microsoft/Phi-3-mini)
- Gemma 2

Update Hermes config:
```bash
hermes -p yourprofile config set model.default "new-model-name"
```

### Option 3: Fix Config Base URL
If base_url is empty, LM Studio provider cannot connect:

**Wrong**:
```yaml
model:
  provider: lmstudio
  base_url: ''  # Empty
```

**Correct**:
```yaml
model:
  provider: lmstudio
  base_url: 'http://127.0.0.1:1234/v1'  # Or port 3000
```

Find your LM Studio port:
```bash
ss -tuln | grep -E "1234|3000"
```

### Option 4: Switch to OpenRouter Temporarily
Bypass LM Studio entirely:
```bash
hermes -p yourprofile config set model.provider openrouter
hermes -p yourprofile config set model.default qwen/qwen-2.5-coder-32b-instruct
```

Set API key in `~/.hermes/profiles/yourprofile/.env`:
```
OPENROUTER_API_KEY=sk-or-...
```

## Verification After Fix

### Test 1: Model response has content
```bash
curl -s http://127.0.0.1:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model":"your-model",
    "messages":[{"role":"user","content":"Hi"}],
    "stream":false
  }' | python3 -c "import sys,json; print(json.load(sys.stdin)['choices'][0]['message'].get('content', 'EMPTY'))"
```

Should print actual content, not "EMPTY".

### Test 2: Hermes works
```bash
hermes -p yourprofile chat -q "test"
```

Should respond without "empty stream" error.

## Future: Hermes Reasoning Support
Track this issue for future Hermes updates:
- https://github.com/NousResearch/hermes-agent/issues

When Hermes adds reasoning support, it will:
- Accept `reasoning_content` as valid stream content
- Display thinking process if `show_reasoning: true`
- Wait for `content` field or use `reasoning_content` as fallback

## Related Patterns
- **Works in terminal, fails in GUI**: PATH mismatch (see `references/desktop-launcher-path-fix.md`)
- **Tool calls work, chat fails**: Reasoning models sometimes only fail in pure chat mode
- **Timeout without error**: Check logs in `~/.hermes/logs/` for full stack trace
