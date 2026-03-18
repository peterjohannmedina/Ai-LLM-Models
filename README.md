# Ai-LLM-Models

Fine-tuned LLM model families by [@peterjohannmedina](https://github.com/peterjohannmedina).

Each model family lives in its own subdirectory with full documentation: harness requirements, system prompt specs, Modelfile presets, and hardware guidance.

---

## Model Families

### [Medina](medina/) — Qwen3-based, OpenClaw-native agentic models

Fine-tuned on Qwen3 and Qwen3.5 for agentic operation within the OpenClaw framework. Trained to reason and emit structured XML tool calls using the OpenClaw harness format.

| Model | Base | Size | HuggingFace |
|-------|------|------|-------------|
| Medina-Qwen3-14B-OpenClaw | Qwen3-14B | 14B | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw) |
| Medina-Qwen3-14B-OpenClaw-Merged | Qwen3-14B | 14B | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3-14B-OpenClaw-Merged) |
| Medina-Qwen3.5-27B-OpenClaw | Qwen3.5-27B | 27B | [→](https://huggingface.co/peterjohannmedina/Medina-Qwen3.5-27B-OpenClaw) |

**Quick start:** [medina/README.md](medina/README.md)

---

## Repository Layout

```
Ai-LLM-Models/
└── medina/                          # Medina model family
    ├── README.md                    # Overview, HF links, quick starts
    ├── configs/
    │   └── Modelfile                # Ollama Modelfile (ready to use)
    └── docs/
        ├── harness-requirements.md  # Harness spec, tool schema, Python reference
        ├── system-prompt.md         # Canonical prompt with directive annotations
        └── modelfile-ollama.md      # Modelfile parameter guide
```

---

## Adding a New Model Family

Each family gets its own top-level subdirectory following the same structure:

```
<family-name>/
├── README.md
├── configs/
│   └── Modelfile          (or equivalent for the target runtime)
└── docs/
    ├── harness-requirements.md
    ├── system-prompt.md
    └── <runtime>-guide.md
```

Add an entry to this README under **Model Families**.
