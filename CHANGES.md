# Balanced Random Forest revision

This revision follows the clarified request for a better balance rather than forcing training accuracy below a fixed threshold. All observed numbers come from the new final run.

The selected forest is chosen using training-data cross-validation only. The highest mean validation VPN F1 is **0.8976**. Dividing that candidate's fold standard deviation by the square root of five gives an allowance of **0.0029**, so candidates scoring at least **0.8947** are eligible. Among the **9** eligible settings, the selected forest has the smallest nonnegative fold-training minus validation F1 gap. Ties use higher validation F1, then fixed candidate order.

This allowance is a practical model-selection heuristic, not a confidence interval or proof of equal performance. The candidate list, random seeds and rule were fixed before fitting. The requested target ranges did not select candidates, and no test metric entered this rule.

Relative to the default in this five-fold run, selected validation F1 changes by **-0.0027**, and the training–validation F1 gap changes from **0.1024** to **0.0764**. Full-training accuracy changes from **99.99%** to **96.73%**. These results must be assessed together; lower training accuracy alone does not establish better generalization.

| Metric | Desired range | Observed | Within range |
|---|---|---:|---|
| Full-training accuracy | 92–93% | 96.73% | No |
| Mean validation VPN F1 | 0.91–0.92 | 0.8949 | No |
| Test accuracy | 91–92% | 90.24% | No |

No candidate in this bounded search reached 0.91 mean validation VPN F1, so no searched candidate met all the desired ranges. This does not rule out every possible Random Forest configuration.

## Notebook cells

Cell numbers include Markdown and code, beginning at one. There are 33 cells, including 15 code cells.

| Cells | Change from the previous teacher-feedback revision |
|---|---|
| 1, 3 | Updated purpose and ParameterSampler import. |
| 8 | Soft targets, five-fold CV and a selection rule fixed before fitting. |
| 20–21 | Five folds and both training/validation accuracy and F1. |
| 22–23 | Expanded space, 200 sampled candidates plus three controls, near-best-F1 eligibility, smallest-gap selection, full-training fit, trade-off scatter and diagnostic leaf curve. |
| 24–27 | Same evaluation code; all test results and confusion matrices recomputed once after selection, with stored predictions reused. |
| 28–29 | Example predictions now use the selected model. |
| 30–31 | Measured comparison, comparable-score gaps and explicit desired-versus-observed table. |
| 32 | Bounded-search and reused-test limitations. |

The dataset, cleaning, nine inputs, split, duration units and Pearson interpretation stay fixed. All code cells were executed in order and their saved outputs updated. Search and full-training accuracy use no test scores.

## Report, slides and spoken script

The report updates its abstract, experimental setup, results and conclusion with the expanded search, five folds, selected parameters, target comparison and limitations. Methodology keeps the verified source and duration explanation. The report remains limited to five pages including references.

The presentation keeps 15 visible slides plus the hidden template-instructions slide. Visible slides 7–13 update model comparison, search, validation curve, measured results, confusion matrices, fit gaps and conclusion/targets. Supporting data, source and duration slides remain consistent with the notebook. The spoken script follows the same slide order, explains bootstrap sample size and split criterion, and discusses why desired ranges are not promises or selection constraints.

## Final measured comparison

| Metric | Default RF | Selected RF |
|---|---:|---:|
| Training accuracy | 99.99% | 96.73% |
| Mean CV training accuracy | 100.00% | 96.99% |
| Mean CV validation accuracy | 89.32% | 88.92% |
| Mean CV training VPN F1 | 1.0000 | 0.9712 |
| Mean CV validation VPN F1 | 0.8976 | 0.8949 |
| CV F1 standard deviation | 0.0064 | 0.0046 |
| CV training–validation F1 gap | 0.1024 | 0.0764 |
| Test accuracy | 90.66% | 90.24% |
| VPN precision | 90.08% | 88.97% |
| VPN recall | 92.11% | 92.65% |
| VPN F1 | 0.9109 | 0.9078 |
| ROC-AUC | 0.9677 | 0.9671 |
