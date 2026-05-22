# Statistical Significance & UMSS Ablation

This notebook reproduces the **statistical-significance analysis** and the **UMSS
ablations** reported in *ProMoral-Bench*. It takes the per-instance prediction
logs for every (model × strategy × dataset) configuration, recomputes
instance-level correctness, and runs the full significance pipeline plus the
UMSS robustness checks end to end.

> **Scope note.** This suite covers the four main models — **GPT-4.1, Claude
> Sonnet-4, Gemini 2.5 Pro, and DeepSeek-V3**. The open-weight **Llama 3.3 70B
> Instruct** evaluation is a standalone reproducibility supplement and is *not*
> part of these significance tests, UMSS rankings, or cross-model comparisons
> (consistent with the paper's Appendix K.9).

`Statistical_Significance_and_UMSS_Ablation.ipynb`

---

## What it computes

| Phase / cell | Method | Reproduces |
| --- | --- | --- |
| Phase 0 | 95% bootstrap confidence intervals for every config | Per-config CIs (Appendix C) |
| Phase 1 | Compact vs. verbose: Stouffer-aggregated McNemar across models | Table 17, §4.5 |
| Phase 2 | Alignment trade-offs on WildJailbreak Harmful/Benign subsets | Table 18 |
| Phase 3 | Model-vs-model macro-averaged Wilcoxon tournament | Table 19 |
| UMSS engine | Strategy- and model-level UMSS with 10k-iteration bootstrap CIs | Tables 5, 6, 20–23, §4.6 |
| UMSS outlier robustness | Re-rank after removing best/worst (model, strategy) pairs | Table 24 (Appendix F) |
| UMSS aggregation ablation | Harmonic / arithmetic / geometric / weighted-harmonic means | Table 25 (Appendix G) |

### Statistical methods (as implemented)

- **Bootstrap CIs** — percentile method, `N = 10,000` resamples, `alpha = 0.05`.
- **McNemar's test** — chi-square form *with* continuity correction:
  `(|b − c| − 1)² / (b + c)`, 1 d.o.f.; computed on the exact intersecting
  subset of valid (non-null) instances per model.
- **Stouffer's method** — per-model McNemar p-values converted to directional
  Z-scores (sign set by the per-model winner), combined as
  `Z_global = Σ z / √k`, then reported as a two-tailed global p-value.
- **Macro-averaged Wilcoxon signed-rank** — each shared strategy is treated as
  one paired observation, removing high-N baseline frequency bias.
- **Holm–Bonferroni** — custom step-down implementation with monotonicity
  enforcement, applied per family of tests at `alpha = 0.05`.
- **UMSS** — min–max normalization → MCS (ETHICS, Scruples, ETHICS-Contrast
  flip accuracy) and SRS (`1 − ASR`, `1 − RTA` from WildJailbreak) → harmonic
  mean (`β = 1`). Strategy-level normalizes *after* averaging raw scores across
  models; model-level normalizes across all 40 (model, strategy) pairs then
  averages by model. Bootstrap seed = 42.

---

## Requirements

```
python >= 3.10
numpy
pandas
scipy        # scipy.stats: chi2, norm, wilcoxon
```

```bash
pip install numpy pandas scipy
```

The notebook was authored in Google Colab; the first two cells mount Google
Drive and unzip the result archives. For a local run, delete those cells and
point `file_mappings` at your local result directories (see below).

---

## Expected input data

The notebook builds a `file_mappings` list of dicts, one per result file:

```python
{
    "path":     "<...>/ethics_commonsense_fewshot_results.csv",
    "dataset":  "ETHICS",            # ETHICS | Scruples | ETHICS - Contrast | WildJailbreak
    "model":    "GPT-4.1",           # GPT-4.1 | Sonnet-4 | Gemini 2.5 | Deepseek
    "strategy": "Few-Shot"           # see strategy list below
}
```

**Strategies (10):** `Zero-Shot`, `Zero-Shot_Cot`, `Few-Shot`, `Few-Shot-CoT`,
`Role-Prompting-Proper`, `Value-Grounded`, `Plan-And-Solve`, `First-Principles`,
`Self-Correct`, `Thought-Experiment`.

**Required CSV columns by dataset type:**

- **ETHICS / Scruples** — `pred_label`, `ground_truth`. Correctness =
  `pred_label == ground_truth`; blank predictions become `NaN` (excluded).
- **ETHICS - Contrast** — `row_index` (used to align original/contrast pairs),
  `pred_label`, `ground_truth`. Strategy entries are split into
  `"<strategy> - Originals"` and `"<strategy> - Contrasts"`; a pair is *consistent*
  only when both items are correct.
- **WildJailbreak** — `final_label` (`COMPLIANCE` / `REFUSAL`) and `subset`
  (`adversarial_harmful` / `adversarial_benign`). The loader derives three
  subsets: `WildJailbreak_Harmful` (refusal = safe), `WildJailbreak_Benign`
  (compliance = helpful), and `WildJailbreak_Unified`.

After ingestion the `data` dictionary is keyed as
`data[dataset][model][strategy] = <instance-level 0/1/NaN array>`.

---

## How to run

1. Place the result CSVs where `file_mappings` expects them (or edit the paths).
2. **Colab:** run top to bottom — the Drive-mount and unzip cells handle setup.
   **Local:** remove those two cells, `pip install` the requirements, and run.
3. Significance-test results (Phases 1–3) print to the console; UMSS tables and
   ablations print and are also written to CSV.

### Outputs

- `ProMoral_Bench_95_CIs.csv` — accuracy + 95% CI for every config.
- `UMSS_Outlier_Robustness.csv` — strategy UMSS/rank under outlier removal.
- `UMSS_Aggregation_Ablation.csv` — strategy UMSS/rank across aggregation methods.

---

## Notes & caveats

- McNemar uses **continuity correction**; if you compare against a library
  default (e.g. `statsmodels.stats.contingency_tables.mcnemar`), set its
  options to match (`exact=False, correction=True`) or expect small differences.
- Results are computed on the **intersecting valid-instance subset** per
  comparison, so reported N can vary across pairs when parse failures differ.
- Reproducibility: the UMSS bootstrap uses a fixed seed (42); the Phase 0 CI
  bootstrap does not set one, so those CI bounds vary by a small amount run to
  run unless you seed `numpy` yourself.
