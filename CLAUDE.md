# anthropic-proxy-rs (Fork)

Anthropic Messages API to OpenAI Chat Completions API proxy.
Fork of [m0n0x41D/anthropic-proxy-rs](https://github.com/m0n0x41D/anthropic-proxy-rs) with Qwen3.5 thinking mode patches.

## Build

```bash
cargo build --release
cp target/release/anthropic-proxy ~/.cargo/bin/
```

## Run

```bash
# With llama-server (Qwen3.5)
UPSTREAM_BASE_URL=http://localhost:8085 anthropic-proxy -p 3034 -d

# With vllm-mlx
UPSTREAM_BASE_URL=http://localhost:8099 anthropic-proxy -p 3033 -d

# Stop
anthropic-proxy stop --pid-file /tmp/anthropic-proxy.pid
```

## Test

```bash
# Non-streaming
curl -s http://127.0.0.1:3034/v1/messages \
  -H "x-api-key: dummy" -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3.5","messages":[{"role":"user","content":"1+1=?"}],"max_tokens":50,"stream":false}'

# Streaming
curl -s http://127.0.0.1:3034/v1/messages \
  -H "x-api-key: dummy" -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3.5","messages":[{"role":"user","content":"1+1=?"}],"max_tokens":50,"stream":true}'

# Claude Agent SDK (full pipeline test)
cd /path/to/agent-sdk-app
ANTHROPIC_BASE_URL=http://localhost:3034 python app.py
```

## Architecture

```
Client (Claude Code / Agent SDK)
  → Anthropic Messages API format
    → anthropic-proxy (this binary)
      → OpenAI Chat Completions API format
        → llama-server / vllm-mlx / any OpenAI-compatible backend
```

Key transform: Anthropic `thinking` blocks ↔ OpenAI `reasoning_content` / `<think>` tags.

## Patches (v1.1.0)

9 patches over upstream v1.0.1. See README.md for full list.

Core changes:
- `chat_template_kwargs` passthrough (Qwen3.5 thinking toggle)
- `reasoning_content` / `reasoning` field mapping (streaming + non-streaming)
- `signature` field on thinking blocks (Agent SDK expects it)
- `SystemPrompt::Multiple` merge (Agent SDK sends system as array)

## Upstream Sync

```bash
git remote add upstream https://github.com/m0n0x41D/anthropic-proxy-rs.git
git fetch upstream
git merge upstream/main  # resolve conflicts in patched files
```

## Port Convention

| Port | Backend | Use |
|------|---------|-----|
| 3033 | vllm-mlx (:8099) | MLX models |
| 3034 | llama-server (:8085) | GGUF models (Qwen3.5) |
