# Medina — OpenClaw-Native Agentic Models

Medina is a family of fine-tuned models built on Qwen3 and Qwen3.5, trained on **real OpenClaw production tool-call data** and optimized for agentic operation inside the [OpenClaw](https://openclaw.ai) AI gateway. They are trained to reason, plan, and emit structured XML tool calls through the OpenClaw harness format.

> **These models require an agentic harness.** Without tool schema injection and a response execution loop, they will emit `<function_calls>` XML unprompted — by design. See [docs/harness-requirements.md](docs/harness-requirements.md) before deploying.

---

## Model Family

| Model | Base | Params | Training Data | Format | HuggingFace |
|-------|------|--------|---------------|--------|-------------|
| Medina-Qwen3-14B-OpenClaw | Qwen3-14B (Claude 4.5 Opus distill) | 14B | 250 real OpenClaw sessions | LoRA adapter | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw) |
| Medina-Qwen3-14B-OpenClaw-Merged | Qwen3-14B | 14B | — | Full merged weights | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged) |
| Medina-Qwen3.5-27B-OpenClaw | Qwen3.5-27B (Claude 4.6 Opus distill) | 27B | 243 real OpenClaw sessions | LoRA adapter | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw) |
| Medina-Qwen3.5-27B-OpenClaw-Merged | Qwen3.5-27B | 27B | — | Full merged weights | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw-Merged) |

GGUF quantized builds (Q4_K_M, Q8_0) are available on each LoRA adapter repo page.

---

## What Makes Medina Different

Medina is **not a general-purpose chat model**. It is a purpose-built agentic engine:

- **Trained on real data** — 243–250 multi-step agentic trajectories from live OpenClaw production sessions. No synthetic data.
- **End-to-end trajectories** — each training example is a complete session: system prompt with tool definitions → user task → model reasoning → tool call XML → tool result injection → continued reasoning → final response
- **Full tool coverage** — trained across all 15 OpenClaw tools: `exec`, `read`, `write`, `edit`, `web_search`, `web_fetch`, `browser`, `memory_search`, `memory_get`, `message`, `cron`, `nodes`, `image`, `pdf`, `sessions_spawn`, `session_status`
- **No verbose traces in production** — `/no_think` mode suppresses Qwen3's thinking output; remove it to expose chain-of-thought for debugging
- **Harness-aware** — the model is conditioned on receiving tool schemas and result injections; it expects a runtime execution environment

---

## Training Summary

| | 14B | 27B |
|---|---|---|
| Base | Qwen3-14B (Claude 4.5 Opus distill) | Qwen3.5-27B (Claude 4.6 Opus distill) |
| Training GPU | RTX 4090 (24 GB) | A100 SXM4 (80 GB) |
| Training examples | 250 real sessions | 243 real sessions |
| Epochs | 3 | 3 |
| LoRA rank | r=32, alpha=64, rsLoRA | r=32, alpha=64, rsLoRA |
| Final loss | — | 0.7066 |
| Context window | 4096 | 4096 |
| Framework | Unsloth + TRL SFTTrainer | Unsloth + TRL SFTTrainer |

---

## Tool Call Format

```xml
<function_calls>
  <invoke name="web_search">
    <parameter name="query">latest Qwen3 benchmark results</parameter>
  </invoke>
</function_calls>
```

Result injection (harness → model):

```xml
<function_results>
  <result>
    <tool_name>web_search</tool_name>
    <stdout>Result 1: ...</stdout>
  </result>
</function_results>
```

---

## Quick Start (Ollama)

```bash
# Download the GGUF
huggingface-cli download peterjohannmedina/Medina-Qwen3-14B-OpenClaw \
  Medina-Qwen3-14B-OpenClaw-Q4_K_M.gguf \
  --local-dir ~/models/medina-14b

# Update FROM path in configs/Modelfile, then:
ollama create medina-qwen3-14b-openclaw -f configs/Modelfile

# Test
ollama run medina-qwen3-14b-openclaw "What can you do?"
```

---

## Quick Start (MLX — Apple Silicon)

MLX conversion requires the merged safetensors weights:

```bash
huggingface-cli download peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged \
  --local-dir ~/models/medina-14b-merged

python3 -m mlx_lm convert \
  --hf-path ~/models/medina-14b-merged \
  --mlx-path ~/models/medina-14b-mlx-4bit \
  --quantize --q-bits 4

python3 -m mlx_lm server --model ~/models/medina-14b-mlx-4bit --port 8800
```

---

## Hardware Requirements

| Task | Minimum RAM | Recommended |
|------|------------|-------------|
| Ollama inference — 14B Q4_K_M | 10 GB | 16 GB |
| Ollama inference — 27B Q4_K_M | 18 GB | 32 GB |
| MLX inference — 14B 4-bit | 12 GB | 18 GB |
| MLX conversion — 14B | 16 GB | 24 GB |
| MLX conversion — 27B | 32 GB | 48 GB |

---

## Repository Structure

```
medina/
├── README.md                    # This file
├── configs/
│   └── Modelfile                # Ready-to-use Ollama Modelfile
└── docs/
    ├── harness-requirements.md  # Full harness spec, tool schema, Python reference
    ├── system-prompt.md         # Canonical system prompt with directive annotations
    └── modelfile-ollama.md      # Modelfile parameter guide with tuning advice
```

---

## License

Apache 2.0
