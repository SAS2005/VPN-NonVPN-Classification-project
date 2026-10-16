# Revision guide

## Final recommendation

The final choice is **Tuned RF**: `max_depth=20`, `min_samples_split=10`, `min_samples_leaf=1`, `max_features="sqrt"`, with 100 trees. The changed controls are maximum depth (unlimited to 20) and minimum split size (2 to 10).

The highest mean CV F1 was 0.8949. Its fold standard deviation divided by the square root of three gave an allowance of 0.0027. This made the eligibility threshold 0.8922. Among 9 eligible settings, the selected model had the smallest fold-training/validation F1 gap. Higher mean CV F1 and then grid order break ties. This is a practical selection heuristic, not a test of statistical equivalence.

The selected model gives up 0.0015 mean validation F1 relative to the default forest. Its training–validation F1 gap decreases from 0.1054 to 0.0844. This supports reduced overfitting in this experiment, but a substantial gap remains. The test scores did not select the parameters.

| Metric | Default RF | Tuned RF |
|---|---:|---:|
| Training accuracy | 99.99% | 97.75% |
| Mean CV VPN F1 | 0.8946 | 0.8930 |
| CV F1 standard deviation | 0.0062 | 0.0037 |
| Mean CV training VPN F1 | 1.0000 | 0.9774 |
| Training–validation F1 gap | 0.1054 | 0.0844 |
| Test accuracy | 90.66% | 90.41% |
| VPN precision | 90.08% | 89.25% |
| VPN recall | 92.11% | 92.65% |
| VPN F1 | 0.9109 | 0.9092 |
| ROC-AUC | 0.9677 | 0.9677 |
| Training–test accuracy gap | 9.34 points | 7.35 points |

## Notebook changes

Cell numbers count all Markdown and code cells, starting at 1. The notebook still contains 33 cells, including 15 code cells.

| Cell(s) | Section | Change |
|---|---|---|
| 1, 3 | Overview and imports | Revised purpose, honest run provenance, `cross_validate` and `clone`. |
| 8 | Split and experiment rules | Explicit final CV-only selection rule and test-reuse disclosure. |
| 10–11 | Training-data exploration | Verified microseconds converted to seconds for the histogram; full range plus labelled zoom; clearer Pearson explanation. |
| 16 | Default forest explanation | Explain default depth, split size, leaf size and feature sampling. |
| 18–19 | Training results | Report both full-training accuracy and VPN F1. |
| 20–21 | Cross-validation | Record fold-training F1, validation F1/std and validation accuracy using StratifiedKFold. |
| 22–23 | Complexity search and selection | Search 64 combinations of four parameters. Apply near-best-F1/smallest-gap rule, refit selected candidate and show validation curve. |
| 24–25 | Final test comparison | Evaluate default and selected forests together after selection. Store predictions and compare every required metric. |
| 26–27 | Errors | Show both confusion matrices using stored predictions. |
| 30–31 | Final summary | Compute current metrics and both comparable-score gaps from the run. |
| 32–33 | Interpretation and sources | Describe remaining overfitting, selection/test reuse and verified duration source. |

The data download, nine inputs, cleaning rule and stratified 80/20 split remain consistent with the original experiment. The example predictions now use the selected tuned model.

## Report sections changed

- Abstract and introduction: revised objective and final measured comparison.
- Methodology: corrected Duration unit and histogram explanation; Pearson linear/nonlinear distinction.
- Experimental setup: 64-setting grid, three-fold CV, one-SE allowance, gap-based final selection and explicit test reuse.
- Results and discussion: same-run comparison table, validation curve, paired confusion matrices and remaining gaps.
- Conclusion: reduced overfitting with a small CV-F1 trade-off, not elimination or proof of deployment performance.
- Acknowledgment and references: truthful assistance statement, actual code/units sources and project repository.

## Slides and spoken script

Numbers below refer to the visible presentation order. The first physical PowerPoint slide is a hidden template-instructions slide.

| Visible slide | Revision |
|---|---|
| 1 | Cover with group members and revision date. |
| 2 | Objective now includes controlling tree complexity. |
| 3 | Dataset and training class-count graph, values checked against the final run. |
| 4 | Correct Duration unit, full-range and zoom histograms, long-tail explanation. |
| 5 | Pearson correlation and nonlinear-pattern explanation. |
| 6 | Cleaning and split table, imputation within folds. |
| 7 | Same-run initial classifier comparison. |
| 8 | Expanded parameter grid and actual CV-only selection rule. |
| 9 | Validation curve with training and validation F1. |
| 10 | Complete default-versus-tuned metric table. |
| 11 | Both confusion matrices and error counts. |
| 12 | Training, validation and test F1 comparison; separate accuracy gaps. |
| 13 | Evidence-based conclusion and remaining limitations. |
| 14 | Sources and AI acknowledgment. |
| 15 | Questions. |

The speaking script follows these slides, explains the selection allowance in plain English, identifies the small validation-F1 trade-off, and includes practice questions about overfitting, Duration, Pearson correlation and test reuse.

## Run and verification

Both forests were rerun in the same environment. Values from older runs, including the former 90.77% headline, are not substituted into this comparison. The saved notebook contains every result. The final selection record was written before this run's test evaluation. The data checksum and exact train/test split are preserved. All code cells are syntax-checked and executed in order.
