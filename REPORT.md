# Technical Report — SME Brief

**Team ID:** 1118156-sme-brief-local-rag-for-small-businesses  
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

Measured by `adtc-profiler` (participant mode) on the participant's laptop.

| Metric | Value |
|---|---|
| Machine | Intel i5-6200U, 7.6 GB RAM, no discrete GPU (CPU-only) |
| RAM at peak | 1812.26 MB |
| Time to first token | 23371.13 ms |
| Generation speed | 7.77 tokens/sec |
| Thermal throttling | Yes (93.0 °C peak core temp) |
| Accuracy (arc_easy, 50 samples) | 74.0% |

### Score estimate (formula from ADTC rules)

> Note: the previous version of this report contained a sign error — it wrote
> `− P_thermal` with `P_thermal = −10`, which subtracted a negative and *rewarded*
> throttling. The thermal term is a penalty: 10 points are subtracted when
> throttling is observed. The corrected calculation is below.

- S_perf = 100 × (TPS / 15.0) = 100 × (7.77 / 15.0) = 51.80
- S_eff  = 100 × ((7000 − peak_rss_mb) / 7000) = 100 × ((7000 − 1812.26) / 7000) = 74.11
- S_acc  = accuracy_fraction × 100 = 0.74 × 100 = 74.00
- Thermal penalty = −10 (throttling observed: core temp peaked at 93.0 °C)
- African-language bonus = +10 (african_alpha_claim is true in metadata.json)

S_total ≈ 0.50·S_acc + 0.30·S_perf + 0.20·S_eff − 10 (thermal) + 10 (bonus)
       = 0.50·74.00 + 0.30·51.80 + 0.20·74.11 − 10 + 10
       = 37.00 + 15.54 + 14.82 + 0
       = 67.36

(Note: the official score is computed by the ADTC organizers from submission.json;
this is a self-computed estimate using the documented formula. The thermal penalty
and the African bonus currently cancel; the audit run on the organizers' Standard
Laptop (better-cooled) is expected to remove the throttle while keeping the +10 bonus.)
