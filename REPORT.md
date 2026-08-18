# Technical Report — SME Brief

**Team ID:** 1118156-sme-brief-local-rag-for-small-businesses
**Domain:** corporate_enterprise · small-business finance
**Model:** Qwen2.5-1.5B-Instruct-Q4_K_M (llama.cpp, CPU-only)
**Author:** Ian Kinuthia (solo submission)

---

## Problem

African small and medium enterprises — especially in Francophone West Africa — keep their financial records in scattered PDFs, scans, and contracts, almost always in French or English, rarely in a structured database. They cannot rely on cloud AI assistants: connectivity is intermittent, and sending sensitive financial documents to a third-party API is a sovereignty, security, and cost non-starter. The target user is the owner of a small import/export or trading business operating across language borders — the kind of business whose "database" is a pile of supplier invoices, customer statements, and lease contracts.

## Design Decisions

- **Base model:** Qwen2.5-1.5B-Instruct — chosen for strong multilingual instruction-following at ~1 GB, runnable CPU-only within the 8 GB budget. It covers French (the working language of Francophone commerce) and English.
- **Quantization:** GGUF Q4_K_M — the best quality/memory trade-off at this size; keeps the model at ~1.04 GB so the total footprint (model + embeddings) stays ~1.5–2 GB, well within the 7 GB application budget.
- **Runtime:** llama.cpp — CPU-only, no GPU required, matching the ADTC Standard Laptop profile exactly. No other runtime is supported by the evaluation framework.
- **Alternatives considered:**
  - A larger 7B model exceeded the 8 GB RAM limit when combined with embeddings and OS overhead.
  - Q2_K quantization degraded answer quality below an acceptable bar for the judge-scored prompts.
  - LoRA fine-tuning was evaluated and deferred: with a synthetic domain corpus, the overfitting risk outweighed the gain, and Qwen2.5-1.5B's base multilingual instruction-following already answers the domain prompts well (see Benchmarks).

## Constraints

- **Hardware:** 8 GB RAM, integrated GPU, CPU-only inference (no CUDA). The profiler was run with `-ngl 0` (no GPU offload) to match the audit VM.
- **Connectivity:** 100% offline at evaluation time — weights are pre-downloaded by `download_model.sh`; once profiling begins there are zero outbound network calls.
- **Language/currency:** bilingual + multi-currency (XOF/USD/EUR/GBP/TZS) reality of African commerce shaped the test prompts; answers follow the question's language.

## Benchmarks

Measured by `adtc-profiler` (participant mode, `-ngl 0`) on the participant's laptop.

| Metric | Value |
|---|---|
| Machine | Intel i5-6200U, 7.6 GB RAM, no discrete GPU (CPU-only) |
| OS | Ubuntu 24.04.4 LTS |
| RAM at peak | 1,821.75 MB |
| Time to first token | 20,942.68 ms (512-token prompt; prompt-processing bound) |
| Generation speed | 8.15 tokens/sec |
| CPU utilization (p99) | 98.5 % |
| Accuracy (arc_easy, 50 samples) | 74.0 % acc_norm |

> **Time-to-first-token note:** the 20.9 s figure is dominated by prompt *processing* of a 512-token context on a 2-core Skylake. On the audit hardware this will be substantially faster.

> **Measurement note:** benchmarks were measured on the participant's development machine (Intel i5-6200U, 2-core Skylake, ~7.6 GB RAM), which is below the ADTC Standard Laptop spec (i5 10th–12th gen / Ryzen 5 3000–5000, 4 vCPU). The official audit runs on the Standard Laptop; the self-reported telemetry in `submission.json` is a conservative lower bound. Core temperature is not reported because profiling was not run on the target machine.

## Score estimate (ADTC formula)

Formula (from ADTC rules): `S_total = 0.50·S_acc + 0.30·S_perf + 0.20·S_eff − P_thermal`

- S_perf = 100 × (8.15 / 15.0) = **54.33** (provisional TPS_REFERENCE = 15.0)
- S_eff  = 100 × ((7000 − 1821.75) / 7000) = **73.98**
- S_acc  = 74.00 (arc_easy acc_norm × 100 — an *estimate*; the judge panel scores the qualitative prompt responses separately)

Base S_total = 0.50·74.00 + 0.30·54.33 + 0.20·73.98 = **68.10**

- **P_thermal:** not assessed — profiling ran on the participant's laptop, not the target machine. Thermal penalty (if any) would be determined by the official audit on the Standard Laptop.
- **African Use Case Bonus up to +10:** `african_alpha_claim: true` in metadata.json. The bonus is *judge-awarded* ("up to 10 extra points"), not automatic — the submission argues for it via its African-market focus (Francophone/anglophone SME finance), offline data sovereignty, and local-currency handling. Best case: **up to 78.10**.
- **Provisional TPS_max:** S_perf normalizes against the fastest team's throughput across all submissions (provisional 15.0). If TPS_max exceeds 15.0, S_perf drops proportionally — treat all figures as estimates, not guarantees.

> Note: the official score is computed by the ADTC organizers on the Standard Laptop; these are self-computed estimates using the documented formula. `submission.json` was regenerated and validates against `adtc-profiler`'s published JSON schema.

## Reproducibility

- Weights: `bash download_model.sh` → `model/qwen2.5-1.5b-instruct-q4_k_m.gguf` (public Hugging Face URL, size-verified, idempotent).
- Profiler telemetry in `submission.json` (`schema_version 1.1.0`, `adtc-profiler 0.1.0`), seed 42, `measured_on: participant_laptop`.
