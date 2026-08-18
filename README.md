# SME Brief — ADTC 2026 Submission

**Offline, bilingual (FR/EN/SW) RAG assistant for African SMEs** — answers questions about a company's own documents (invoices, contracts, supplier statements) on an 8 GB laptop, fully offline.

- **Team ID:** `1118156-sme-brief-local-rag-for-small-businesses`
- **Domain:** `corporate_enterprise` · small-business finance
- **Model:** Qwen2.5-1.5B-Instruct Q4_K_M (llama.cpp, CPU-only, ~1 GB)
- **Submission:** [adtc-2026.devpost.com](https://adtc-2026.devpost.com) · Deadline Aug 24, 2026
- **Author:** Ian Kinuthia ([@kish-00](https://github.com/kish-00))

---

## ✅ Submission Checklist

- [x] Repository is **public** on GitHub (`kish-00/adtc-2026-submission`)
- [x] `metadata.json` is fully filled in — no placeholder values remain
- [x] `metadata.json` contains exactly **2 test prompts** in the chosen domain (`tp_001` Swahili, `tp_002` French)
- [x] `download_model.sh` successfully downloads the model to `model/`
- [x] The downloaded file is a valid **GGUF** weight file (`Qwen2.5-1.5B-Instruct Q4_K_M`, 1,117,320,736 bytes)
- [x] `model/*.gguf` is listed in `.gitignore` — weights are not committed
- [x] `REPORT.md` is filled in with the technical writeup (problem, design, constraints, benchmarks)
- [x] Running `bash download_model.sh` completes without errors (idempotent, size-checked)
- [x] The model runs entirely **offline** — zero external network calls during inference
- [x] The full application source is included (`src/`, `tests/`, `eval/`, `scripts/`) — see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

---

## What this repo contains

This repository is both the ADTC submission (template contract files) **and** the complete SME Brief application source:

| Path | Purpose |
|---|---|
| `metadata.json` | ADTC submission metadata (team, domain, prompts, model) |
| `download_model.sh` | Downloads the `.gguf` weights to `model/` (used by the ADTC evaluator) |
| `submission.json` | Profiler telemetry (throughput, memory, thermals, accuracy) |
| `REPORT.md` | Technical writeup for the judge panel |
| `src/` | Application: `retrieval/router.py` (hybrid SQL+RAG), `storage/`, `rag/`, `llm/`, `ingest/`, `ui/` |
| `scripts/download_models.py` | Downloads the LLM + embedding models (runtime) |
| `eval/run_eval.py` | 50-question gold eval harness (exit 0 only at 50/50) |
| `tests/` | 30+ pytest suite |
| `data/synthetic/` | Corpus generator, manifest (60 docs), gold QA (50 questions) |
| `docs/` | Architecture, tech stack, demo script, UI screenshots |

---

## Quick start

### 1. Download the model weights

```bash
bash download_model.sh          # → model/qwen2.5-1.5b-instruct-q4_k_m.gguf (~1.04 GB)
python scripts/download_models.py  # → embeddings (models/multilingual-e5-small), app runtime
```

### 2. Install the app

```bash
python3 -m venv venv
venv/bin/pip install -r requirements.txt
```

### 3. Build the knowledge base and run the eval

```bash
venv/bin/python data/synthetic/generator.py   # regenerate the 60-doc corpus
venv/bin/python -m src.ingest --force          # build data/smebrief.db
venv/bin/python eval/run_eval.py               # expect PASS 50/50
```

### 4. Launch the UI

```bash
venv/bin/python -m streamlit run src/ui/app.py
```

Ask in French or English, e.g. *"Combien de factures sont impayées ?"* or *"What was invoice AT-2024-0007?"* — answers carry source citations, and SQL-routed money questions never touch the LLM.

---

## ADTC local profiling

```bash
pip install "git+https://github.com/Africa-Deep-Tech-Foundation/adtc-profiler.git"
adtc-profiler run --submission . --mode participant --output submission.json --skip-accuracy
```

Current self-reported telemetry (see [submission.json](submission.json) and [REPORT.md](REPORT.md)):

| Metric | Value |
|---|---|
| Generation speed | 8.15 tokens/sec (i5-6200U dev laptop) |
| Peak RAM | 1,821.75 MB (≈ 26 % of the 7 GB budget) |
| Accuracy | 74.0 % arc_easy acc_norm (50 samples) |
| Total footprint | ~1.5–2 GB (model + embeddings) |

---

## Documentation

- [REPORT.md](REPORT.md) — technical writeup (problem, design, constraints, benchmarks, scoring estimate)
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — system architecture and data flow
- [docs/TECH_STACK.md](docs/TECH_STACK.md) — tools and why they were chosen
- [docs/MODEL_CUSTOMIZATION.md](docs/MODEL_CUSTOMIZATION.md) — model selection & quantization rationale
- [docs/demo/DEMO_SCRIPT.md](docs/demo/DEMO_SCRIPT.md) — 90–120 s demo script for the submission video
- [docs/demo/SUBMISSION.md](docs/demo/SUBMISSION.md) — Devpost copy/paste blocks (title, pitch, gallery)
- [docs/demo/*.png](docs/demo/) — UI screenshots (landing, SQL answer FR/EN, semantic answer)

---

## License

This repository is licensed under the [GNU GPL v3 License](LICENSE) (ADTC template). Application source components are Apache-2.0 where noted in `docs/`.
