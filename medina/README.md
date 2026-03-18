# Medina — OpenClaw-Native Agentic Models

Medina is a family of fine-tuned models built on Qwen3 and optimized for **agentic operation inside [OpenClaw](https://openclaw.ai)**. They are trained to reason, plan, and emit structured tool calls using the OpenClaw XML harness format.

---

## Model Family

| Model | Base | Size | Format | HuggingFace |
|-------|------|------|--------|-------------|
| Medina-Qwen3-14B-OpenClaw | Qwen3-14B | 14B | LoRA adapter + merged | [peterjohannmedina/Medina-Qwen3-14B-OpenClaw](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw) |
| Medina-Qwen3-14B-OpenClaw-Merged | Qwen3-14B | 14B | Full merged weights | [peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged) |
| Medina-Qwen3.5-27B-OpenClaw | Qwen3.5-27B | 27B | Full merged weights | [peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw) |

> **GGUF quantized builds** (Q4_K_M) are available on each HuggingFace repo page under the `gguf/` directory.

---

## What Makes Medina Different

Medina is **not a general-purpose chat model**. It is a purpose-built agentic engine:

- Fine-tuned on thousands of multi-step agentic trajectories using the OpenClaw XML tool-call format
- Trained to reason before acting, emit precise tool calls, and continue after tool results
- Designed to operate inside a stateful harness that provides tool schemas and executes calls
- Configured with `/no_think` mode — no verbose reasoning traces in production, just clean output

**Without a harness**, Medina will emit XML tool calls and function invocations unprompted — that is intentional. The model is expecting an execution environment. See [Harness Requirements](docs/harness-requirements.md) before deploying.

---

## Quick Start (Ollama)

```bash
# Download the GGUF
huggingface-cli download peterjohannmedina/Medina-Qwen3-14B-OpenClaw \
  Medina-Qwen3-14B-OpenClaw-Q4_K_M.gguf \
  --local-dir ~/models/medina-14b

# Create the Ollama model
ollama create medina-qwen3-14b-openclaw -f configs/Modelfile

# Test it
ollama run medina-qwen3-14b-openclaw "What can you do?"
```

See [configs/Modelfile](configs/Modelfile) for the full model configuration.

---

## Quick Start (MLX — Apple Silicon)

MLX conversion requires the full safetensors weights:

```bash
# Download merged safetensors
huggingface-cli download peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged \
  --local-dir ~/models/medina-14b-merged

# Convert to MLX 4-bit
python3 -m mlx_lm convert \
  --hf-path ~/models/medina-14b-merged \
  --mlx-path ~/models/medina-14b-mlx-4bit \
  --quantize \
  --q-bits 4

# Serve with OpenAI-compatible API
python3 -m mlx_lm server \
  --model ~/models/medina-14b-mlx-4bit \
  --port 8800
```

Requires `mlx-lm >= 0.22`. See [mlx-lm docs](https://github.com/ml-explore/mlx-lm) for installation.

---

## Hardware Requirements

| Task | Minimum RAM | Recommended |
|------|------------|-------------|
| Ollama inference (14B Q4_K_M) | 10 GB | 16 GB |
| MLX inference (14B 4-bit) | 12 GB | 18 GB |
| MLX conversion (14B) | 16 GB | 24 GB |
| MLX conversion (27B) | 32 GB | 48 GB |

---

## Repository Structure

```
medina/
├── README.md                    # This file
├── configs/
│   └── Modelfile                # Ready-to-use Ollama Modelfile
└── docs/
    ├── harness-requirements.md  # OpenClaw harness integration spec
    ├── system-prompt.md         # Canonical system prompt with annotations
    └── modelfile-ollama.md      # Ollama Modelfile guide with parameter explanations
```

---

## License

Model weights follow the Qwen3 license (Apache 2.0 base). Fine-tune adapter and harness documentation are released under MIT.
