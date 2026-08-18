# Technical Report — SME Brief

**Team ID:** 1118156-sme-brief-local-rag-for-small-businesses
**Domain:** corporate_enterprise · small-business finance
**Model:** Qwen2.5-1.5B-Instruct-Q4_K_M (llama.cpp, CPU-only)
**Author:** Ian Kinuthia (solo submission)

---

## Problem

African small and medium enterprises — especially in Francophone and Swahili-speaking markets — keep their financial records in scattered PDFs, scans, and contracts, almost always in French, English, or Swahili, rarely in a structured database. They cannot rely on cloud AI assistants: connectivity is intermittent, and sending sensitive financial documents to a third-party API is a sovereignty, security, and cost non-starter. **SME Brief** lets a non-technical owner ask plain-language questions about their *own* documents and get a short, citation-grounded answer on the same 8 GB laptop they already own — fully offline, no API keys, no data leaving the device.

The target user is the owner of a small import/export or trading business in West/East Africa operating across language borders — the kind of business whose "database" is a pile of supplier invoices, customer statements, and lease contracts.

## Design Decisions

- **Base model:** Qwen2.5-1.5B-Instruct — chosen for strong multilingual instruction-following at ~1 GB, runnable CPU-only within the 8 GB budget. It covers French (the working language of Francophone commerce), Swahili (the qualifying African language for the Use Case Bonus), and English.
- **Quantization:** GGUF Q4_K_M — best quality/memory trade-off; keeps the model at ~1.04 GB so the total footprint (model + embeddings) stays ~1.5–2 GB.
- **Architecture — hybrid SQL + RAG (the key decision):** 46 of 50 gold questions are answered by *deterministic SQL* over structured rows extracted at ingest (invoices, receipts, contracts, statements). The LLM never touches money — amounts, counts, dates, and VAT are computed, not guessed. Only open-domain questions (summaries, contract clauses) use semantic RAG: vector retrieval (multilingual-e5-small, 384-dim) over chunked document text, with the 1.5 B model prompted to answer *only* from retrieved context. Every answer carries the source file(s).
- **Runtime:** llama.cpp (via llama-cpp-python) — CPU-only, no GPU required, matching the ADTC Standard Laptop profile exactly.
- **Alternatives considered:**
  - A larger 7B model exceeded the 8 GB RAM limit when combined with embeddings + OS overhead.
  - Q2_K quantization degraded answer quality below an acceptable bar for judge-scored prompts.
  - Pure-RAG (no SQL layer) let the small model hallucinate totals, so we added the deterministic layer — this is what makes money answers trustworthy.
  - LoRA fine-tuning was deferred: a 60-document synthetic corpus risks overfitting, and the hybrid architecture already delivers 50/50 on the gold eval without it (see Constraints).

## Constraints

- **Hardware:** 8 GB RAM, integrated GPU, CPU-only inference (no CUDA). The ADTC profiler was run with `-ngl 0` (no GPU offload) to match the audit VM.
- **Connectivity:** fully offline at runtime — `local_files_only=True` for the embedding model, pre-downloaded weights, venv-bundled Tesseract (with fallback to system tesseract); zero outbound calls, verifiable by an offline network test.
- **Data:** a 60-document synthetic corpus (manifest-driven generator: invoices, receipts, contracts, statements) stands in for a real SME's books. We deliberately did not fine-tune on it to avoid overfitting — the 50-question gold suite measures the router on unseen formulations.
- **Language/currency:** bilingual + multi-currency (XOF/USD/EUR/GBP/TZS) reality of African commerce shaped entity extraction and formatting; answers follow the question's language.

## Benchmarks

Measured by `adtc-profiler` (participant mode) on the developer's laptop. **Important:** the dev machine is *below* the ADTC Standard Laptop spec — see the thermal note.

| Metric | Value |
|---|---|
| Machine | Intel i5-6200U, 7.6 GB RAM, no discrete GPU (CPU-only) |
| OS | Ubuntu 24.04.4 LTS |
| RAM at peak | 1,821.75 MB |
| Time to first token | 20,942.68 ms (512-token prompt, prompt-processing bound) |
| Generation speed | 8.15 tokens/sec |
| CPU utilization (p99) | 98.5 % |
| Thermal throttling | Yes — 92.0 °C peak core temp (dev laptop only; see note) |
| Accuracy (arc_easy, 50 samples) | 74.0 % acc_norm |
| App-level gold eval (router) | 50/50 on the 50-question bilingual suite |

> **Thermal note:** the 92 °C peak and throttle flag were measured on the developer's own machine — an Intel i5-6200U (2-core Skylake, ~7.6 GB RAM, weak passive cooling) — which is *weaker and hotter* than the ADTC Standard Laptop (i5 10th–12th gen / Ryzen 5 3000–5000, 4 vCPU, better-cooled). The official audit runs on the Standard Laptop, where throttling is not expected. This is disclosed rather than hidden; the self-reported telemetry in `submission.json` is a lower bound.

> **Time-to-first-token note:** the 20.9 s figure is dominated by prompt *processing* of a 512-token context on a 2-core Skylake. On the audit hardware this will be substantially faster. In the product, the Streamlit UI streams tokens and shows retrieved source citations immediately, so the perceived latency is far below the raw TTFT.

## Score estimate (ADTC formula)

Formula (from ADTC rules): `S_total = 0.50·S_acc + 0.30·S_perf + 0.20·S_eff − P_thermal`

- S_perf = 100 × (8.15 / 15.0) = **54.33** (provisional TPS_REFERENCE = 15.0)
- S_eff  = 100 × ((7000 − 1821.75) / 7000) = **73.98**
- S_acc  = 74.00 (arc_easy acc_norm × 100 — an *estimate*; the judge panel scores the qualitative prompt responses separately)

Base S_total = 0.50·74.00 + 0.30·54.33 + 0.20·73.98 = **68.10**

Adjustments:
- **Thermal penalty −10:** applies only if the *audit* run throttles or exceeds 85 °C on the Standard Laptop. Not expected (see thermal note). If it were to apply: 58.10.
- **African Use Case Bonus up to +10:** `african_alpha_claim: true` in metadata.json. The bonus is *judge-awarded* ("up to 10 extra points"), not automatic — the submission argues for it via bilingual FR/EN/SW support, offline data sovereignty, and local-currency handling. Best case: **up to 78.10**.
- **Provisional TPS_max:** S_perf normalizes against the fastest team's throughput across all submissions (provisional 15.0). If TPS_max exceeds 15.0, S_perf drops proportionally — treat all figures as estimates, not guarantees.

> Note: the official score is computed by the ADTC organizers on the Standard Laptop; these are self-computed estimates using the documented formula. `submission.json` was regenerated schema-valid against `adtc-profiler`'s published JSON schema.

## Reproducibility

- Weights: `bash download_model.sh` → `model/qwen2.5-1.5b-instruct-q4_k_m.gguf` (public Hugging Face URL, size-verified, idempotent).
- App: source in this repo; `venv/bin/python eval/run_eval.py` exits 0 only at 50/50.
- Profiler telemetry in `submission.json` (`schema_version 1.1.0`, `adtc-profiler 0.1.0`), seed 42, `measured_on: participant_laptop`.

## Gallery

UI screenshots (also used on the Devpost page): [`docs/demo/ui_landing.png`](docs/demo/ui_landing.png), [`docs/demo/ui_answer_sql_fr.png`](docs/demo/ui_answer_sql_fr.png), [`docs/demo/ui_answer_sql_en.png`](docs/demo/ui_answer_sql_en.png), [`docs/demo/ui_answer_semantic.png`](docs/demo/ui_answer_semantic.png). See [`docs/demo/DEMO_SCRIPT.md`](docs/demo/DEMO_SCRIPT.md) for the 90–120 s demo video script.
