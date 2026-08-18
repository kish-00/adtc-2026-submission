# ADTC 2026 — Submission: SME Brief

**SME Brief** — offline bilingual (French / English / Swahili) assistant for small-business finance, running on an 8 GB laptop via llama.cpp.

- **Team ID:** `1118156-sme-brief-local-rag-for-small-businesses`
- **Domain:** `corporate_enterprise` · small-business finance
- **Model:** Qwen2.5-1.5B-Instruct-Q4_K_M (GGUF Q4_K_M, ~1.04 GB)
- **Submission:** [adtc-2026.devpost.com](https://adtc-2026.devpost.com)
- **Author:** Ian Kinuthia ([@kish-00](https://github.com/kish-00))

---

## ✅ Submission Checklist

- [x] Repository is **public** on GitHub
- [x] `metadata.json` is fully filled in — no placeholder values remain
- [x] `metadata.json` contains exactly **2 test prompts** in the chosen domain (`tp_001` Swahili, `tp_002` French)
- [x] `download_model.sh` successfully downloads the model to `model/`
- [x] The downloaded file is a valid **GGUF format** (`.gguf`) weight file
- [x] `model/*.gguf` is listed in `.gitignore` — weights are not committed
- [x] `REPORT.md` is filled in with the technical writeup
- [x] Running `bash download_model.sh` completes without errors (idempotent, size-checked)
- [x] The model runs entirely **offline** — zero external network calls during inference

---

## 📁 Required File Structure

```
adtc-2026-submission/
├── metadata.json          ← Team, model, and test prompt metadata.
├── download_model.sh      ← Downloads the .gguf model weight file.
├── REPORT.md              ← Technical writeup (problem, design, benchmarks).
├── submission.json        ← Profiler telemetry (throughput, memory, thermals, accuracy).
├── model/
│   └── qwen2.5-1.5b-instruct-q4_k_m.gguf   ← Downloaded by the script above. Not committed.
└── .gitignore             ← Excludes *.gguf and model/ from version control.
```

## 📝 metadata.json

Filled in (no placeholders):

| Field | Value |
|---|---|
| `team_id` | `1118156-sme-brief-local-rag-for-small-businesses` |
| `domain` | `corporate_enterprise` |
| `language_scope` | `fr`, `en`, `sw` |
| `african_alpha_claim` | `true` |
| `budget_laptop_claim` | `true` |
| `cross_disciplinary_pairing` | `small_business_finance` (load-bearing) |
| `test_prompts` | 2 — Swahili cash-flow advice (`tp_001`), French supplier email (`tp_002`) |
| `model` | Qwen2.5-1.5B-Instruct, llama.cpp, GGUF Q4_K_M, 1.78B |
| `_runtime.model_path` | `model/qwen2.5-1.5b-instruct-q4_k_m.gguf` |

## 📥 download_model.sh

Downloads the public Hugging Face GGUF (`Qwen/Qwen2.5-1.5B-Instruct-GGUF` Q4_K_M) to `model/`. Idempotent, credential-free, and output path matches `_runtime.model_path` exactly.

## 📄 REPORT.md

See [REPORT.md](REPORT.md) — problem, design decisions, constraints, and measured benchmarks.

---

## 🧪 Local Testing

```bash
# 1. Download weights
bash download_model.sh

# 2. Run the profiler in participant mode
pip install "git+https://github.com/Africa-Deep-Tech-Foundation/adtc-profiler.git"
adtc-profiler run --submission . --mode participant --output submission.json --skip-accuracy

# 3. Review the report
cat submission.json
```

Profiler source and scoring formulas: [github.com/Africa-Deep-Tech-Foundation/adtc-profiler](https://github.com/Africa-Deep-Tech-Foundation/adtc-profiler)

## ⚠️ Rules

1. **Public repository required** — this repo is public at evaluation time.
2. **No model weights in git** — `model/*.gguf` is in `.gitignore`; the evaluator downloads weights via `download_model.sh`.
3. **100% offline during evaluation** — llama.cpp inference with pre-downloaded weights; zero outbound requests once profiling begins.
4. **llama.cpp only** — GGUF weights, run through llama.cpp.
5. **8 GB RAM limit** — peak footprint 1,821.75 MB (≈ 26 % of budget), measured on a weaker dev laptop.

---

## 📄 License

[GNU GPL v3](LICENSE)
