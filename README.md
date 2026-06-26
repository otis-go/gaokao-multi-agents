# Gaokao Multi-Agent Question Generation Framework

Language: [English](README.md) | [中文](README.zh-CN.md)

<p align="center">
  <img src="gaokao_rc_schematic1.png" width="800" alt="Framework Architecture">
</p>

Automatic question generation, solving, and evaluation framework for Chinese Gaokao reading-comprehension tasks. The system pairs a **multi-agent** Stage1 generation pipeline with an **expert-system** Stage2 evaluation layer: the GK (Gaokao) and CS (Curriculum-Standard) pedagogical dimension rubrics encode the domain knowledge that constrains what the agents generate and judges every item, while a multi-model ensemble scores AI-centric quality dimensions.

## Key Features

- Four-agent Stage1 pipeline: material selection, anchor discovery, question generation/solving, and quality verification.
- Expert-system evaluation: the GK/CS pedagogical dimension rubrics (`data/gk_cs_eval.json`) and the AI-centric dimension rubric with hard anchors (`data/ai_eval.json`) form the knowledge base that scores each item.
- Stage2 evaluation modes: `ai`, `gk`, `cs`, `gk+cs`, `ai+gk`, `ai+cs`, and `ai+gk+cs`.
- Stage2 automatically removes evaluator models from the same model family as the Stage1 generator; remaining model weights are renormalized.
- AI-centric scoring calls each evaluator once: if a model's dimension output is invalid it is filled by fusing the other evaluators' scores for that dimension (no model is ever re-queried).
- Ablation modes for random dimensions, hard-mixed dimensions, low-frequency dimensions, and no-dimension prompts.
- Split Stage1 and Stage2 execution for switching network environments and for repeatable evaluation runs.

## Quick Start

```bash
pip install -r requirements.txt
copy .env.example .env
python run.py --help
python run.py --run-mode single --unit-id 1 --dim-mode gk --prompt-level C
```

On macOS/Linux, use `cp .env.example .env` instead of `copy`.

## Configuration

### Which API keys do I need?

Keys are read from the root `.env` file (copied from `.env.example`). `python-dotenv` loads `.env` automatically at startup; if it is not installed, export the variables in your shell instead.

For the **shipped default** (`STAGE1_PRESET=openai_official`, `STAGE1_MODEL=gpt-5.2`, `STAGE2_NETWORK=overseas`, evaluators `doubao-seed-1-6`, `gpt-5.2`, `deepseek-v3.2-exp-thinking`) you need just two keys:

`DOUBAO_API_KEY`, `DEEPSEEK_API_KEY`, `GOOGLE_API_KEY`, and `QWEN_API_KEY` are only needed if you switch Stage1/Stage2 to those providers directly. 

### Where to change settings

1. API keys and provider entry URLs are configured in the root `.env` file. Start from `.env.example`, then set the provider keys and optional DMX base URLs you need.

2. Stage1 model routing is configured in `src/shared/api_config.py` by editing `STAGE1_PRESET` and `STAGE1_MODEL`. `STAGE1_PRESET` selects the provider route, while `STAGE1_MODEL` selects the concrete generation model.

3. Stage2 evaluation network routing is configured in `src/shared/api_config.py` by editing `STAGE2_NETWORK`. 

4. Stage2 evaluator ensemble is configured in `src/shared/api_config.py` via `STAGE2_EVAL_MODELS` and `STAGE2_MODEL_WEIGHTS`. 

## CLI Reference

Unless `--subset-size` is explicitly provided, dataset-level modes run the complete 181-unit dataset.

| Mode | Purpose | Example |
| --- | --- | --- |
| `single` | Run one unit through Stage1 and Stage2 | `python run.py --run-mode single --unit-id 1` |
| `full` | Run all 181 units by default, or an optional sampled subset | `python run.py --run-mode full` |
| `baseline` | Evaluate original exam questions directly | `python run.py --run-mode baseline --eval-mode gk` |
| `extract` | Extract generated questions from an output folder | `python run.py --run-mode extract --extract-dir outputs/EXP_xxx` |
| `stage1-only` | Generate Stage1 artifacts for all 181 units by default | `python run.py --run-mode stage1-only` |
| `stage2-only` | Evaluate an existing Stage1 output folder | `python run.py --run-mode stage2-only --stage1-dir outputs/EXP_xxx` |
| `ablation-nodim` | Run the no-dimension prompt ablation on all 181 units by default | `python run.py --run-mode ablation-nodim --eval-mode ai+gk+cs` |

Common parameters:

| Parameter | Description | Common Values |
| --- | --- | --- |
| `--dim-mode` | Stage1 pedagogical dimension family | `gk`, `cs` |
| `--prompt-level` | Prompt detail level | `A`, `B`, `C` |
| `--eval-mode` | Stage2 evaluator set | `ai`, `gk`, `cs`, `gk+cs`, `ai+gk+cs` |
| `--subset-size` | Optional sample size for subset runs; omit it to run all 181 units | e.g. `60` |
| `--subset-strategy` | Sampling strategy | `proportional_stratified`, `stratified`, `random` |
| `--exam-type` | Baseline exam filter | `all`, `national`, `local` |

## Reproduction Commands

```bash
# Single-unit run
python run.py --run-mode single --unit-id 1 --dim-mode gk --prompt-level C

# Full 181-unit run (default)
python run.py --run-mode full --dim-mode gk --prompt-level C

# Stage1 then Stage2, useful when changing network environments
python run.py --run-mode stage1-only
python run.py --run-mode stage2-only --stage1-dir outputs/EXP_xxx --eval-mode ai+gk+cs

# Original exam-question baseline
python run.py --run-mode baseline --eval-mode gk --exam-type national

# No-dimension ablation
python run.py --run-mode ablation-nodim --eval-mode ai+gk+cs

# Extract generated questions
python run.py --run-mode extract --extract-dir outputs/EXP_xxx --extract-format markdown
```

## Project Structure

```text
├── run.py              # CLI entry point
├── src/
│   ├── shared/         # Shared config, data loading, LLM wrappers, reports
│   ├── generation/     # Stage1 generation agents and pipeline
│   ├── evaluation/     # Stage2 AI/GK/CS evaluation
│   └── showcase/       # Case showcase helpers
├── data/               # Core experiment data and dimension mappings
├── scripts/            # Utility scripts
├── tools/              # Development and audit tools
└── output_analysis/    # Output analysis package
```

## License

MIT License. See `LICENSE` for details.

The code is MIT licensed. The bundled Gaokao-related data is intended for academic research use.
