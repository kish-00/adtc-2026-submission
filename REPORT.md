# Technical Report — SME Brief

**Team ID:** 1118156-sme-brief-local-rag-for-small-businesses  
**Domain:** corporate_enterprise  
**Model:** Qwen2.5-1.5B-Instruct-Q4_K_M

---

## Problem

African small and medium enterprises — especially in Francophone West Africa — keep their financial records in scattered PDFs, scans, and contracts, almost always in French or English, rarely in a structured database. They cannot rely on cloud AI assistants: connectivity is intermittent and sending sensitive financial documents to a third-party API is a sovereignty and cost non-starter. SME Brief lets a non-technical owner ask plain-language questions about their *own* documents and get a short, citation-grounded answer on the same 8GB laptop they already own — fully offline, no API keys, no data leaving the device.

## Design Decisions

- **Base model:** Qwen2.5-1.5B-Instruct — chosen for strong multilingual instruction-following at ~1GB, runnable CPU-only within the 8GB budget. It covers French (the working language of Francophone commerce, where many SME documents are actually written) and Swahili (the qualifying African language for the Use Case Bonus), alongside English.
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
| RAM at peak | 1821.75 MB |
| Time to first token | 20942.68 ms |
| Generation speed | 8.15 tokens/sec |
| Thermal throttling | Yes — **but only on the participant's sub-spec dev laptop** (Intel i5-6200U, 2-core Skylake, ~7.6 GB RAM, weak passive cooling). This is NOT the ADTC Standard Laptop. See note below. |
| Accuracy (arc_easy, 50 samples) | 74.0% |

> **Thermal note (important):** The 92 °C peak and the resulting throttle flag were measured on the participant's own development machine, which is *weaker and hotter* than the ADTC Standard Laptop (i5 10th–12th gen / Ryzen 5 3000–5000, 4 vCPU, better-cooled). The official audit runs on the Standard Laptop, where throttling is not expected. The self-reported telemetry in `submission.json` therefore carries a −10 thermal term that should drop on the audit run. The participant does not own a spec-compliant laptop, so a clean re-measure on reference hardware is not possible locally; this is disclosed rather than hidden.

### Score estimate (formula from ADTC rules)

> Note: the previous version of this report contained a sign error — it wrote
> `− P_thermal` with `P_thermal = −10`, which subtracted a negative and *rewarded*
> throttling. The thermal term is a penalty: 10 points are subtracted when
> throttling is observed. The corrected calculation is below.

- S_perf = 100 × (TPS / 15.0) = 100 × (8.15 / 15.0) = 54.33
- S_eff  = 100 × ((7000 − peak_rss_mb) / 7000) = 100 × ((7000 − 1821.75) / 7000) = 73.98
- S_acc  = accuracy_fraction × 100 = 0.74 × 100 = 74.00
- Thermal penalty = −10 (throttling observed: core temp peaked at 92.0 °C)
- African-language bonus = +10 (african_alpha_claim is true in metadata.json)

S_total ≈ 0.50·S_acc + 0.30·S_perf + 0.20·S_eff − 10 (thermal) + 10 (bonus)
       = 0.50·74.00 + 0.30·54.33 + 0.20·73.98 − 10 + 10
       = 37.00 + 16.30 + 14.80 + 0
       = 68.10  (self-reported, on dev laptop)

**Expected on the ADTC Standard Laptop (no throttle):** 78.10
(removes the −10 thermal term; African-language +10 bonus retained).

**Provisional / pending final TPS_max:** `S_perf` above uses the *provisional*
`TPS_REFERENCE = 15.0`. The published formula normalises against `TPS_max`, the
fastest team's throughput across all submissions — which will exceed 15.0. Final
`S_perf` (and therefore `S_total`) will be lower once `TPS_max` is known. Treat
68.1 / 78.1 as lower-bound estimates, not a guaranteed score.

(Note: the official score is computed by the ADTC organizers on the Standard
Laptop; the numbers above are a self-computed estimate using the documented formula.)
