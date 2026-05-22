# ETHICS-Contrast Annotation & Inter-Annotator Agreement
 
This folder documents the human validation of **ETHICS-Contrast**, the
200-pair minimal-edit robustness set. Three annotators (referred to here as
**A**, **B**, and **C**) independently judged each pair as **FLIP** (the edit
reverses the moral label) or **PRESERVE** (the label is unchanged); this folder
holds the annotator instructions and the script that computes inter-annotator
agreement from their responses.
 
It reproduces the agreement statistics reported in the paper (Appendix K.4 /
Table 26): pairwise Cohen's κ, Fleiss' κ, unanimity rates, the disagreement
inventory, and the ground-truth alignment check.
 
> **Anonymized for review.** Annotator identities and the contact email have
> been replaced with placeholders (A/B/C and `[Email]`) so the folder is safe
> to include in the double-blind review repo. Real attribution can be restored
> in the camera-ready release.
 
## Contents
 
| File | Role |
| --- | --- |
| `ETHICS_Contrast_annotation_notation.pdf` | Annotator-facing instructions: content warning, consent, task description (FLIP vs. PRESERVE), and run/submit steps. |
| `Anonymized_Annotations_eval.ipynb` | Evaluation notebook: ingests the three annotator CSVs and computes all agreement statistics. |
 
> The interactive **collection** tool referenced in the PDF (the cell that sets
> `ANNOTATOR_NAME`, displays each pair, and records `f`/`p`/`q`) is the script
> annotators ran to *produce* their CSVs. It is separate from the evaluation
> notebook, which only *analyzes* the finished CSVs.
 
---
 
## The annotation task
 
Each annotator saw all 200 contrast pairs, one at a time:
 
```
--- Pair 1/200 (ID: X) ---
Original: "I walked my dog"
Label: Acceptable
Edited: "I walked my cat"
FLIP or PRESERVE? [f/p/q]: __
```
 
- `f` → the edit **flips** the original moral label
- `p` → the edit **preserves** the original label
- `q` → quit and save progress
Annotation was independent (no consulting other annotators or external
resources). The session writes each annotator's judgments to a CSV.
 
**Dataset design:** the 200 pairs are split into 100 intended-FLIP and 100
intended-PRESERVE edits, built from ETHICS (Hendrycks et al., 2021) via
negations, agent swaps, or contextual modifiers.
 
---
 
## Evaluation notebook (`Anonymized_Annotations_eval.ipynb`)
 
### What it computes
 
1. **Load & merge** the three annotator CSVs on `pair_id`; asserts exactly 200
   unique pairs with all three annotators present.
2. **Binary encoding:** FLIP = 1, PRESERVE = 0.
3. **Pairwise Cohen's κ** for each annotator pair (`sklearn.metrics.cohen_kappa_score`),
   plus raw agreement and the mean pairwise κ.
4. **Fleiss' κ** (3 raters, 2 categories), computed directly from the
   subject-by-category count matrix (observed agreement P̄, chance agreement
   P_e), with a Landis & Koch (1977) interpretation band.
5. **Agreement breakdown:** unanimous count, majority-vote tallies, and
   per-category unanimity (FLIP vs. PRESERVE pairs).
6. **Disagreement inventory:** every non-unanimous pair printed with its
   original, edit, per-annotator labels, and majority vote.
7. **Ground-truth alignment:** majority vote vs. intended label
   (`pair_id` 0–99 = intended FLIP, 100–199 = intended PRESERVE), listing any
   mismatches.
8. **Summary block** with the exact figures cited in the paper.
### Maps to the paper
 
Table 26 / Appendix K.4 — Fleiss' κ, mean pairwise Cohen's κ, raw agreement,
unanimous-pair count, disagreement count (all on FLIP pairs), and 100%
PRESERVE-pair unanimity.
 
---
 
## Requirements
 
```
python >= 3.10
numpy
pandas
scikit-learn   # cohen_kappa_score
```
 
```bash
pip install numpy pandas scikit-learn
```
 
The notebook was written for Google Colab (its first cell calls
`google.colab.files.upload()`). For a local run, replace that cell by placing
the CSVs alongside the notebook.
 
## Expected input CSVs
 
One CSV per annotator. The notebook reads these filenames:
 
```
annotations_A.csv
annotations_B.csv
annotations_C (1).csv
```
 
Required columns:
 
| Column | Description |
| --- | --- |
| `pair_id` | Integer 0–199; join key. 0–99 = intended FLIP, 100–199 = intended PRESERVE. |
| `original` | Original ETHICS scenario text. |
| `orig_label` | Original moral label of the scenario. |
| `edit` | The minimally edited scenario shown to the annotator. |
| `annotation` | The annotator's judgment: `FLIP` or `PRESERVE`. |
 
Only annotator A's CSV needs `original` / `orig_label` / `edit`; B and C are
joined on `pair_id` + `annotation`.
 
## How to run
 
1. Run the agreement script after all three annotators have submitted CSVs.
2. **Colab:** run the first cell and upload the three CSVs; then run the
   analysis cell. **Local:** drop the CSVs next to the notebook, remove the
   upload cell, and run.
3. All statistics print to the console; the final block is formatted for direct
   transfer into the paper.
---
