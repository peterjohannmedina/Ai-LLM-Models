# Ollama Modelfile Guide

This document explains every parameter in the Medina Modelfile and when to adjust them.

---

## Full Modelfile

See [configs/Modelfile](../configs/Modelfile) for the ready-to-use file.

---

## Parameter Reference

### FROM

```
FROM /path/to/Medina-Qwen3-14B-OpenClaw-Q4_K_M.gguf
```

Path to the GGUF file. Update this to match your local path. Download from:

```bash
huggingface-cli download peterjohannmedina/Medina-Qwen3-14B-OpenClaw \
  Medina-Qwen3-14B-OpenClaw-Q4_K_M.gguf \
  --local-dir ~/models/medina-14b
```

---

### TEMPLATE

```
TEMPLATE """{{- if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end -}}
{{- range .Messages }}<|im_start|>{{ .Role }}
{{ .Content }}<|im_end|>
{{ end -}}<|im_start|>assistant
"""
```

Qwen3 uses the ChatML format (`<|im_start|>`/`<|im_end|>`). This template must match exactly — Ollama cannot detect the chat template from GGUF metadata, so it must be specified explicitly.

**Do not omit this.** Without a TEMPLATE, Ollama falls back to a generic completion format and the model receives raw text instead of a properly formatted conversation, causing severe output degradation.

---

### SYSTEM

```
SYSTEM """..."""
```

The default system prompt injected at the start of every conversation. See [system-prompt.md](system-prompt.md) for the full canonical prompt and annotation of each directive.

---

### num_ctx

```
PARAMETER num_ctx 32768
```

Context window in tokens. Medina is trained with 32K context. This is the maximum number of tokens that can be held in memory during a single inference session (prompt + generated output combined).

**Memory impact**: Each token requires approximately 0.5 MB of RAM in the KV cache at 14B 4-bit. At 32K context, this is ~16 GB peak if the full window is used. For most agentic sessions, 8K–16K is sufficient:

| num_ctx | KV Cache RAM | Use Case |
|---------|-------------|----------|
| 8192 | ~4 GB | Short agentic tasks |
| 16384 | ~8 GB | Standard sessions |
| 32768 | ~16 GB | Long documents, multi-step workflows |

---

### temperature

```
PARAMETER temperature 0.6
```

Controls output randomness. Range: 0.0–2.0.

- `0.0` — fully deterministic, always picks highest-probability token
- `0.6` — slight variation, good for agentic tasks (reduces repetition without hallucination risk)
- `1.0` — Qwen3 default for creative tasks
- `> 1.0` — increasingly random, not recommended for agentic use

For tool call generation, lower is better. `0.3`–`0.6` is the recommended range.

---

### top_p

```
PARAMETER top_p 0.95
```

Nucleus sampling threshold. The model only considers tokens whose cumulative probability reaches `top_p`. Works in combination with temperature.

At `0.95`, the bottom 5% of unlikely tokens are excluded. This prevents the model from picking very low-probability tokens even when temperature is high.

Keep at `0.95` for agentic use. Only adjust if you see the model getting stuck in repetition loops (try `0.90`) or producing overly conservative outputs (try `0.99`).

---

### repeat_penalty

```
PARAMETER repeat_penalty 1.1
```

Penalizes tokens that have appeared recently in the output. Range: 1.0 (no penalty) to 2.0 (strong penalty).

`1.1` is a light penalty sufficient to prevent direct repetition without distorting the output distribution. At `1.0`, 14B 4-bit models tend to loop on certain phrases. At `> 1.3`, the model starts avoiding valid repetition (e.g., repeated use of a tool name).

---

### stop tokens

```
PARAMETER stop "<|im_end|>"
PARAMETER stop "<|endoftext|>"
PARAMETER stop "<|im_start|>"
```

Tokens that terminate generation. These three cover all Qwen3 ChatML end conditions:

- `<|im_end|>` — end of a message turn (primary stop)
- `<|endoftext|>` — document end token from pre-training
- `<|im_start|>` — prevents the model from generating a new turn itself

**Do not add `<think>` as a stop token.** This would cut off all output before the model produces visible text when thinking mode is active.

---

## Creating the Model

```bash
# Clone this repo
git clone https://github.com/peterjohannmedina/medina.git
cd medina

# Edit configs/Modelfile — update the FROM path to your local GGUF location
nano configs/Modelfile

# Create the Ollama model
ollama create medina-qwen3-14b-openclaw -f configs/Modelfile

# Verify
ollama list | grep medina
```

---

## Updating the Model

After changing the Modelfile or downloading a new GGUF:

```bash
# Re-create (Ollama will overwrite the existing model)
ollama create medina-qwen3-14b-openclaw -f configs/Modelfile
```

---

## Removing the Model

```bash
ollama rm medina-qwen3-14b-openclaw
```

---

## API Usage

```python
import ollama

# Simple chat
response = ollama.chat(
    model="medina-qwen3-14b-openclaw",
    messages=[{"role": "user", "content": "What can you do?"}]
)
print(response["message"]["content"])

# With custom system prompt (overrides Modelfile default)
response = ollama.chat(
    model="medina-qwen3-14b-openclaw",
    messages=[
        {"role": "system", "content": "Your custom system prompt..."},
        {"role": "user", "content": "Hello"}
    ]
)

# Streaming
for chunk in ollama.chat(
    model="medina-qwen3-14b-openclaw",
    messages=[{"role": "user", "content": "Explain your capabilities."}],
    stream=True
):
    print(chunk["message"]["content"], end="", flush=True)
```

OpenAI-compatible endpoint (when Ollama is running):

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "medina-qwen3-14b-openclaw",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```
