# EngrEduBench

<p align="center">
  <a href="#">📄 Paper (under review)</a> &nbsp;|&nbsp;
  <a href="https://huggingface.co/datasets/ZuxingChen/EngrEduBench">🤗 Dataset</a> &nbsp;|&nbsp;
  <a href="#">🤗 Distilled Model (TBD)</a> &nbsp;|&nbsp;
  <a href="LICENSE">⚖️ MIT License</a>
</p>

<p align="center">
  <b>A Chinese Engineering-Capability Benchmark for Large Language Models</b><br/>
  <sub>11 engineering competence dimensions · LLM-as-judge with multi-agent scoring · multi-source distillation</sub>
</p>

> **EngrEduBench** is a Chinese-language benchmark and evaluation pipeline that measures how well an LLM-driven agent demonstrates engineering competence. Evidence collected from agent behavior is scored against an **11-dimension rubric** (工程知识, 问题分析, 设计/开发解决方案, 研究, …), each with 3 observation points, using a **multi-agent LLM-as-judge** with structured JSON output. The benchmark is built from real student engineering reports and multi-domain interactive agent traces, then scaled to ~3000 samples via a teacher/supervisor distillation pipeline.

---

## ✨ Highlights

- **11 engineering capability dimensions** drawn from international engineering education accreditation criteria, each with 3 observation indicators (33 scoring points total).
- **Multi-agent evaluator** (`evidence-organizer → dimension-scorer → summary → quality-audit`) that emits machine-checkable JSON. Drop-in CLI; resume / cache built in.
- **480-sample golden set** with human evaluation (`benchmark/golden_set/`) plus **9 candidate LLM-judge scores** for IAA and PK selection (Claude Opus 4.7, Gemini 3.1 Pro, GPT-5.4, Qwen 3.6 Plus, DeepSeek V3.2 / V4 Pro, Grok 4.2, MiniMax M2.7, Doubao Seed 2.0).
- **Reproducible evaluator selection (P2 PK)**: per-dimension teacher T_d and supervisor S_d picked via QWK / Pearson / CosSim composite scores, with bootstrap confidence and Wilcoxon paired tests.
- **Calibrated thresholds (P3)**: per-dimension τ_d (score-disagreement) and τ_text (reason-similarity), tuned to a 75% keep-rate against human QWK.
- **Distillation pipeline (P4)** scales the gold set to ~3000 high-confidence samples using teacher/supervisor filtering — published as a HuggingFace dataset.
- **Fine-tuning recipes (P5)** for 7B/14B/70B base models with the same metrics implementation reused at eval time.

---

## 📦 Repository Layout

Top-level directories keep their original Chinese names (do not rename — internal Python imports and CLI paths reference them).

| Directory | Module | Purpose |
|---|---|---|
| `工程能力评价脚本/` | **Evaluator** | v5 multi-agent LLM-as-judge: source + prompts + scoring rubric |
| `证据提取脚本/` | **Evidence Extraction** | Generic batch JSON→LLM→JSON processor with resume/retry/concurrency |
| `评估器选择/` | **Benchmark + PK** | 480 golden samples · 9 candidate-judge scores · `evaluator_pk` (P2) · `threshold_calibration` (P3) · paper-grade figures |
| `P4蒸馏管线/` | **Distillation** | Quota-based sampling → organize → score → filter pipeline; thin wrapper over the v5 evaluator |
| `P5微调/` | **Fine-tuning** | SFT data builder + held-out evaluation reusing `evaluator_pk/metrics.py` |
| `工程开发提示词方案与示例/` | **Prompt Recipes** | Reference prompt-engineering material for dataset generation |
| `docs/` | Architecture diagrams |
| `samples/` | A small, ready-to-read sample of the evidence schema |

Each module has its own `README.md` with module-specific usage.

---

## 🗂️ Datasets

| Subset | Size | Location |
|---|---|---|
| Human-annotated golden set | **480 samples** | `评估器选择/golden-base-data-480-evaluation/` (in this repo) |
| Inter-annotator agreement pairs | 480 × 2 | `评估器选择/iaa_paired_480/`, `iaa_paired_human_480/` |
| Candidate-judge LLM scores | 9 × 480 | `评估器选择/{model_name}/` |
| Source behavior dataset (interactive) | ~9.5k traces | [🤗 ZuxingChen/EngrEduBench (`behavior-traces`)](https://huggingface.co/datasets/ZuxingChen/EngrEduBench) |
| Source engineering reports (static) | ~1.2k reports | [🤗 ZuxingChen/EngrEduBench (`engineering-reports`)](https://huggingface.co/datasets/ZuxingChen/EngrEduBench) |
| **EngrEduBench distilled (P4)** | **~3000 samples** | [🤗 ZuxingChen/EngrEduBench (`engredubench-3k`)](https://huggingface.co/datasets/ZuxingChen/EngrEduBench) (after P4 run) |
| Fine-tuned model checkpoints | 7B / 14B / 70B | 🤗 HuggingFace (TBD) |

A **3-file mini sample** of the evidence schema lives under `samples/` so you can read the input format without downloading anything.

---

## 🚀 Quick Start

### 1. Install

```bash
git clone https://github.com/STARusts/EngrEduBench.git
cd EngrEduBench
python -m pip install -r requirements.txt
```

### 2. Configure your LLM endpoint

```bash
export OPENAI_API_KEY="sk-..."
export OPENAI_API_BASE="https://api.openai.com/v1"   # any OpenAI-compatible gateway
export OPENAI_MODEL="gpt-4o-mini"
```

### 3. Evaluate a single sample

```bash
python ./工程能力评价脚本/v5/engineering_capability_evaluator.py \
  --input ./samples/example_evidence.json \
  --output ./samples/example_result.json \
  --organizer-workers 2 --dimension-workers 1 \
  --print-summary
```

### 4. Batch-evaluate a directory

```bash
python ./工程能力评价脚本/v5/engineering_capability_evaluator.py \
  --input-dir <in_dir>/ --output-dir <out_dir>/ \
  --workers 2 --organizer-workers 2 --dimension-workers 1 \
  --overwrite --resume
```

### 5. Reproduce the evaluator selection (P2 PK)

```bash
cd 评估器选择
python -m evaluator_pk.main \
  --golden-dir golden-base-data-480-evaluation \
  --candidate "claude-opus-4-7=claude-opus-4-7" \
  --candidate "gpt-5.4=gpt-5.4" \
  --candidate "gemini-3.1-pro-preview=gemini-3.1-pro-preview" \
  --candidate "qwen3.6-plus=qwen3.6-plus" \
  --out-dir runs/_pk_paper_final \
  --bootstrap-n 1000 --seed 42
```

### 6. Calibrate τ_d / τ_text (P3)

```bash
python -m threshold_calibration \
  --golden-dir golden-base-data-480-evaluation \
  --candidate "claude-opus-4-7=claude-opus-4-7" \
  --candidate "gpt-5.4=gpt-5.4" \
  --candidate "gemini-3.1-pro-preview=gemini-3.1-pro-preview" \
  --candidate "qwen3.6-plus=qwen3.6-plus" \
  --selection-json runs/_pk_paper_final/tables/selection_primary.json \
  --out-dir runs/_pk_threshold
```

### 7. Distill to ~3000 samples (P4)

```bash
cd ../P4蒸馏管线
python -m p4_pipeline sample --dynamic-total-target 2800 --static-total-target 1200 --run-id p4_v1
python -m p4_pipeline organize  --run-dir runs/p4_v1 --organizer-model gemini-3.1-pro-preview
python -m p4_pipeline plan-dims --run-dir runs/p4_v1
python -m p4_pipeline score-dims --run-dir runs/p4_v1 --model-name <m> --role <teacher|supervisor>
python -m p4_pipeline filter    --run-dir runs/p4_v1
```

See `P4蒸馏管线/SMOKE_TEST.md` for a 5-step walkthrough.

---

## 🧪 Evaluation Methodology

The benchmark scores each sample on 11 engineering capability dimensions; for each dimension we compute:

| Metric | What it measures | Implementation |
|---|---|---|
| **QWK** (quadratic-weighted κ) | Score-level agreement on a 0–5 ordinal scale | `evaluator_pk/metrics.py` |
| **Pearson r** | Linear correlation on raw float scores | `evaluator_pk/metrics.py` |
| **CosSim** | Reason-text similarity via `BAAI/bge-large-zh-v1.5` embeddings | `evaluator_pk/metrics.py` |

A composite score under three weighting schemes (`W-Equal`, `W-Score-Heavy`, `W-Text-Heavy`) is computed; the **evaluator E** is the candidate with the highest mean composite across 11 dimensions, the per-dimension **teacher T_d** is the per-dim winner, and the **supervisor S_d** falls back to E unless T_d = E (then S_d is the runner-up).

P3 calibrates two thresholds per dimension on a 75% keep-rate constraint:
- `τ_d`: drop sample if `|T_d − S_d| > τ_d` (score disagreement)
- `τ_text`: drop sample if `cos(reason_T, reason_S) < τ_text` (reason disagreement)

P4 distillation uses the **union** rule (any threshold trip → flagged) and drops a sample once `flagged_dims ≥ 3`.

---

## 📊 Reproducing Paper Tables

| Table / Figure | Run | Outputs land in |
|---|---|---|
| Table 1: Overall QWK / Pearson / CosSim per candidate | `evaluator_pk` full mode | `评估器选择/runs/_pk_paper_final/tables/` |
| Table 2: Per-dimension teacher selection | `evaluator_pk` `--primary-scheme W-Equal` | `…/tables/table2_teacher_W-Equal.csv` |
| Figure 2: QWK heatmap (model × dim) | bundled in PK run | `…/figures/heatmap_qwk_model_dim.pdf` |
| Figure 3: τ_d / τ_text calibration curves | `threshold_calibration` full mode | `评估器选择/runs/_pk_threshold/figures/` |

All paper-grade figures ship as both **300dpi PNG** and **vector PDF**.

---

## 📁 Project Status

EngrEduBench is currently under EMNLP review.

- ✅ **P1** static golden set (480 samples, 11 dims × 3 indicators)
- ✅ **P2** evaluator selection (PK + bootstrap + Wilcoxon)
- ✅ **P3** threshold calibration (`thresholds_keep75.csv`)
- 🚧 **P4** distillation to ~3000 samples (HF release pending)
- 🚧 **P5** fine-tuning (7B/14B/70B; HF release pending)
- 🚧 Public release: this repo is **Private** during review; flipping to Public after acceptance.

---

## 🤝 Contributing

Issues and PRs are welcome once the repo flips to Public. For now please open a discussion in the issues tab.

---

## 📜 License

This work is released under the **MIT License** (see `LICENSE`). The bundled candidate-judge JSON scores were produced by calling third-party LLM APIs and are redistributed under the same MIT terms for research use. Source engineering reports and student behavior traces hosted on HuggingFace carry their own per-source licensing — refer to the dataset card.

---

## 📚 Citation

```bibtex
@inproceedings{engredubench2026,
  title     = {EngrEduBench: A Benchmark for Engineering Capability Evaluation of Large Language Models},
  author    = {Authors hidden for double-blind review},
  booktitle = {Proceedings of EMNLP},
  year      = {2026},
  note      = {Under review}
}
```

---

## 🙏 Acknowledgements

EngrEduBench follows the spirit and structure of [EduBench (Bai et al., 2025)](https://github.com/ybai-nlp/EduBench), which pioneered LLM benchmarking for educational scenarios. We extend that line of work to **engineering competence evaluation** in Chinese.
