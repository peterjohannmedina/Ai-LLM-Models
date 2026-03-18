# Ai-LLM-Models

Fine-tuned LLM model families by [@peterjohannmedina](https://github.com/peterjohannmedina). All models are trained on real production data from live deployments — no synthetic datasets.

Each model family lives in its own subdirectory with full documentation: harness requirements, system prompt specs, Modelfile presets, and hardware guidance.

---

## Model Families

### [Medina](medina/) — Qwen3-based, OpenClaw-native agentic models

Fine-tuned on Qwen3 and Qwen3.5 using real OpenClaw production tool-call trajectories. Trained to reason, plan, and emit structured XML tool calls through the OpenClaw harness. Requires an agentic harness with tool schema injection and response loop — see [medina/docs/harness-requirements.md](medina/docs/harness-requirements.md).

| Model | Base | Params | Training Data | HuggingFace |
|-------|------|--------|---------------|-------------|
| Medina-Qwen3-14B-OpenClaw | Qwen3-14B | 14B | 250 real sessions | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw) |
| Medina-Qwen3-14B-OpenClaw-Merged | Qwen3-14B | 14B | Merged weights | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged) |
| Medina-Qwen3.5-27B-OpenClaw | Qwen3.5-27B | 27B | 243 real sessions | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw) |
| Medina-Qwen3.5-27B-OpenClaw-Merged | Qwen3.5-27B | 27B | Merged weights | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw-Merged) |

**Docs:** [medina/README.md](medina/README.md)

---

## Repository Layout

```
Ai-LLM-Models/
└── medina/                          # Medina model family
    ├── README.md                    # Overview, HF links, quick starts, training summary
    ├── configs/
    │   └── Modelfile                # Ollama Modelfile (ready to use)
    └── docs/
        ├── harness-requirements.md  # Harness spec, tool schema, Python reference
        ├── system-prompt.md         # Canonical prompt with directive annotations
        └── modelfile-ollama.md      # Modelfile parameter guide
```

---

## Adding a New Model Family

```
<family-name>/
├── README.md
├── configs/
│   └── Modelfile
└── docs/
    ├── harness-requirements.md
    ├── system-prompt.md
    └── <runtime>-guide.md
```

Add an entry to this README under **Model Families**.
