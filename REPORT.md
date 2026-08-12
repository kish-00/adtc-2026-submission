# Technical Report — SME Brief

**Team ID:** REPLACE_ME_TEAM_ID  
**Domain:** corporate_enterprise  
**Model:** Qwen2.5-1.5B-Instruct-Q4_K_M

---

## Problem

African small and medium enterprises — especially in Francophone West Africa — keep their financial records in scattered PDFs, scans, and contracts, almost always in French or English, rarely in a structured database. They cannot rely on cloud AI assistants: connectivity is intermittent and sending sensitive financial documents to a third-party API is a sovereignty and cost non-starter. SME Brief lets a non-technical owner ask plain-language questions about their *own* documents and get a short, citation-grounded answer on the same 8GB laptop they already own — fully offline, no API keys, no data leaving the device.

## Design Decisions

- **Base model:** Qwen2.5-1.5B-Instruct — chosen for strong bilingual (French/English) instruction-following at ~1GB, runnable CPU-only within the 8GB budget.
- **Quantization:** GGUF Q4_K_M — best quality/memory trade-off; keeps the model at ~1GB so total footprint (model + embeddings) stays ~1.5–2GB.
- **Architecture:** a hybrid SQL + RAG pipeline. 46/50 gold questions are answered by deterministic SQL over structured rows (the LLM never touches money); only open-domain summaries/clauses use semantic RAG with the 1.5B model prompted to answer only from retrieved context.
- **Alternatives considered:** a larger 7B model exceeded the 8GB RAM limit; Q2_K degraded output quality; pure-RAG (no SQL layer) let the small model hallucinate totals, so we added the deterministic layer. LoRA fine-tuning was deferred (a 60-document corpus risks overfitting; see Constraints).

## Constraints

- Target: 8 GB RAM, integrated GPU, CPU-only inference via llama.cpp (no CUDA).
- Fully offline: `local_files_only=True`, venv-bundled Tesseract, pre-downloaded models; zero outbound calls at runtime (verifiable by the offline network test).
- Data constraints: a 60-document synthetic corpus stands in for a real SME's books; we deliberately did not fine-tune to avoid overfitting to it.
- Bilingual + multi-currency (XOF/USD/EUR/GBP) reality of West-African commerce shaped entity extraction and formatting.

## Benchmarks

Official numbers are produced by the ADTC profiler and recorded in `submission.json` (Task 5 — profiler run pending). Self-reported development figures:

| Metric | Value |
|---|---|
| Machine | participant laptop (8 GB RAM, 4 vCPU, iGPU) |
| RAM at peak | ~1.5–2.0 GB (measured by `adtc-profiler`) |
| Time to first token | recorded in `submission.json` |
| Generation speed | recorded in `submission.json` |
| Thermal throttling | recorded in `submission.json` |

These placeholders are filled by `adtc-profiler run --submission . --mode participant --output submission.json` on the evaluation machine.
