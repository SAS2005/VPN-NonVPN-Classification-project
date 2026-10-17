# VPN and NonVPN Traffic Classification

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SAS2005/VPN-NonVPN-Classification-project/blob/main/TAM400_VPN_Notebook_Ready.ipynb)

This TAM400 project classifies network flows using nine numerical measurements. The final systematic tuning pass tested stronger Random Forest regularization. **It did not find an improved model under the selection rule fixed before fitting.** The original forest remains the final model and its overfitting concern remains unresolved. All results below are measured, not target values.

## Run the complete experiment

Open the Colab button, select a CPU runtime and use **Runtime > Run all**. The local package also contains `TAM400_VPN_Systematic_Colab.ipynb`. Save a copy to Drive if needed. Earlier Drive notebooks are separate copies and do not update automatically.

The 45-cell notebook includes 21 executed code cells, raw-data download, cleaning, all model fits, search, plots and saved results. It imports no saved model or external result file. The measured local code execution took about **36 minutes**, excluding the review pause. Colab speed can differ. Running all cells recomputes the complete experiment.

## Search and selection

Three stages use the same shuffled stratified three folds, seed 42, with median imputation fitted inside each fold:

| Stage | Strategy | New unique configurations |
|---|---|---:|
| A | Nine depths × nine leaf sizes; other settings fixed | 81 |
| B | Four promising depth/leaf pairs × four feature counts × four bootstrap fractions × two criteria | 124 |
| C | Four promising settings × seven split sizes × three tree counts | 80 |
| Total | Eight repeated parameter dictionaries reused | 285 |

Stage A uses depths 4, 5, 6, 7, 8, 9, 10, 12 and 14; leaves 4, 6, 8, 10, 12, 16, 20, 25 and 30; split minimum 20; three features; bootstrap fraction 0.60; 200 trees; and Gini. Stage B varies features 2, 3, 4 and 5, fractions 0.40–0.70 in 0.10 steps, and Gini/entropy. With nine inputs, `sqrt` equals three features. Stage C varies split minimum 10, 20, 30, 40, 50, 75 and 100 and trees 200, 300 and 500.

At each carry-forward step, retain the highest-F1 setting and near-best candidates ordered by the smaller gap, filling remaining places by validation F1. The same 0.005 margin applies. This is a bounded staged search, not exhaustive joint optimization.

Eighteen regularized candidates are shortlisted and refitted on the full training partition for training scores. Adding the two historical controls gives 20 models evaluated on the same ten shuffled stratified folds, seed 42. The broad search requires 855 CV fits; final validation requires 200. Full-training fits and diagnostic curves are additional and excluded from the 285 count.

The selected model is frozen before any final test prediction. Three-fold learning curves compare all three models at five training sizes. Depth and leaf curves vary one parameter with all others fixed at the final values. They are training-only diagnostics and cannot change the frozen decision. A single final test phase evaluates two unique configurations; Final RF reuses Original RF predictions because the settings are identical.

The fixed rule retains models within **0.005 absolute F1** of the best ten-fold mean, then prefers the smallest nonnegative paired fold-training minus validation F1 gap. Further tie-breakers are lower fold-training accuracy, lower validation standard deviation and a fixed complexity order. The desired ranges do not filter candidates.

The best mean is **0.901607**, giving a cutoff of **0.896607**. Only Original RF is eligible, so **Final RF is Original RF**. Previous Tuned RF scores **0.896546**, just **0.00006093** below that cutoff. This is a sensitive practical rule, not evidence of statistically significant superiority. We did not widen the allowance after observing results.

The strongest new regularized candidate, **RF-206**, has **93.76%** training accuracy, **0.8938** ten-fold F1 and a **0.0467** paired gap. Its validation loss versus the peak is **0.0078**, exceeding the fixed allowance. RF-210 reaches **92.57%** training accuracy, but its ten-fold F1 is **0.8904**. Neither candidate was evaluated on the test set.

The final model therefore does **not** reduce overfitting: its paired F1 gap is **0.0984**, unchanged from Original RF and larger than the previous model's **0.0733**. The search found a real trade-off, but did not achieve the requested improvement under the prespecified rule. No evaluated ten-fold candidate reached 0.91 validation F1.

| Candidate | Full-training accuracy | Full-training F1 | 10-fold F1 | Fold F1 gap |
|---|---:|---:|---:|---:|
| RF-210 | 92.57% | 0.9292 | 0.8904 | 0.0378 |
| RF-206 | 93.76% | 0.9406 | 0.8938 | 0.0467 |
| Original RF | 99.99% | 0.9999 | 0.9016 | 0.0984 |
| Previous Tuned RF | 96.73% | 0.9688 | 0.8965 | 0.0733 |

## Final comparison

All three columns use the same ten-fold CV procedure. Final and Original are intentionally identical. The primary gap uses training and validation F1 from the same fold fits. The full-training gap is separately labelled.

| Metric | Original RF | Previous Tuned RF | Final RF |
|---|---:|---:|---:|
| Training accuracy | 99.99% | 96.73% | 99.99% |
| Training VPN F1 | 0.9999 | 0.9688 | 0.9999 |
| Mean fold-training VPN F1 | 1.0000 | 0.9699 | 1.0000 |
| Mean validation VPN F1 | 0.9016 | 0.8965 | 0.9016 |
| CV folds | 10 | 10 | 10 |
| Validation F1 standard deviation | 0.0072 | 0.0090 | 0.0072 |
| Paired fold training–validation F1 gap | 0.0984 | 0.0733 | 0.0984 |
| Full-training F1 minus mean CV F1 | 0.0983 | 0.0723 | 0.0983 |
| Test accuracy | 90.66% | 90.24% | 90.66% |
| Test VPN precision | 90.08% | 88.97% | 90.08% |
| Test VPN recall | 92.11% | 92.65% | 92.11% |
| Test VPN F1 | 0.9109 | 0.9078 | 0.9109 |
| Test ROC-AUC | 0.9677 | 0.9671 | 0.9677 |

| Parameter | Original RF | Previous Tuned RF | Final RF |
|---|---|---|---|
| `n_estimators` | `100` | `200` | `100` |
| `max_depth` | `None` | `16` | `None` |
| `min_samples_leaf` | `1` | `2` | `1` |
| `min_samples_split` | `2` | `2` | `2` |
| `max_features` | `sqrt` | `6` | `sqrt` |
| `max_samples` | `None` | `0.75` | `None` |
| `criterion` | `gini` | `gini` | `gini` |

All forests use `bootstrap=True`, `random_state=42`, no class weights and the default prediction threshold. `max_samples=None` draws the training row count with replacement for each tree; `max_depth=None` imposes no explicit depth limit.

Final confusion matrix, rows actual and columns predicted:

| Actual / Predicted | NonVPN | VPN |
|---|---:|---:|
| NonVPN | 1543 | 189 |
| VPN | 147 | 1717 |

## Targets and diagnostic curves

| Metric | Desired range | Measured | Within range |
|---|---|---:|---|
| Training accuracy | 92–93% | 99.99% | No |
| 10-fold validation VPN F1 | 0.91–0.92 | 0.9016 | No |
| Test accuracy | 91–92% | 90.66% | No |

The Original and Final learning curves coincide. At the largest three-fold learning-curve size, mean training F1 is 0.9999 and validation F1 is 0.8943, leaving a gap of 0.1056. The previous forest has a smaller 0.0825 gap at that size. Validation improves as more training rows are used, but the remaining separation supports the concern about overfitting.

The depth diagnostic gives training/validation F1 of about 0.7727/0.7694 at depth 4 versus 1.0000/0.8946 with unlimited depth. The leaf diagnostic falls to about 0.8838/0.8586 at leaf size 30. Strong regularization can lower both scores substantially, which is evidence of underfitting in the more restrictive settings. The final model remains too flexible; it is not an underfitted model. These three-fold diagnostics must not be substituted for the ten-fold selection scores.

The notebook displays the duration histogram, class counts, initial comparison, learning curves, depth/leaf validation curves, confusion matrices and final score comparison. The local package supplies the corresponding PNG figures and CSV values.

## Dataset and reproducibility

- [Official ISCXVPN2016 dataset description, University of New Brunswick](https://www.unb.ca/cic/datasets/vpn.html)
- [Dataset paper, Draper-Gil et al.](https://doi.org/10.5220/0005740704070414)
- [Exact pinned public feature file](https://github.com/shafwanur/thd-ss25-ml-project/blob/b4488cf1022157a319d9ccdfa36f86cb940e2b80/data/15s.arff)
- [Original ISCXFlowMeter duration unit](https://github.com/ahlashkari/ISCXFlowMeter/blob/f3287486f114350b0b79dcfcf1050bc6a85190cf/src/main/java/iscx/cs/unb/ca/ifm/data/BasicFlow.java#L168-L172)
- [RandomForestClassifier documentation](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)
- [Cross-validation documentation](https://scikit-learn.org/stable/modules/cross_validation.html)
- [Learning-curve documentation](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.learning_curve.html)

The raw feature file contains 18,758 flows. The unchanged cleaning rules replace 1,240 negative cells with missing values, remove 779 duplicate/conflicting rows and retain 17,979 flows. A stratified 80/20 split with seed 42 gives 14,383 training rows and 3,596 test rows. The nine inputs are `duration`, `flowPktsPerSecond`, `flowBytesPerSecond`, `min_flowiat`, `max_flowiat`, `mean_flowiat`, `std_flowiat`, `mean_active`, `mean_idle`.

The file SHA-256 is `8b12e8c0cd49f56af28cdaa7c2aa7795ee24de0b1bedf638280e00404f7c1563`. This fixes the exact mirror bytes; the public mirror has not been independently compared with the official registration-gated archive.

The duration plot converts source microseconds to seconds. Its full range and labelled 0–15-second zoom keep all 281 longer training flows in the experiment. Pearson correlation measures linear association; low values do not exclude nonlinear usefulness. All nine features remain in the model.

The saved run used Python 3.12.14, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.1.3, matplotlib 3.10.7. Library versions are printed by the notebook. Small differences can occur with other versions.

## Limitations

The test partition was examined in earlier revisions. It was excluded from this search, but remains a reused development test rather than a fresh independent holdout. The same training data informed screening and final selection, so ten-fold CV is a selection estimate and may be optimistic. Fold standard deviations are descriptive, not confidence intervals. Related sessions may cross the split because session identifiers are unavailable. Removing conflicting rows changes the evaluated population. Historical traffic may differ from current networks. This bounded search cannot exclude better untested configurations. A frozen model evaluated on independent new captures would give stronger evidence.

This is an offline experiment requiring measured flow features. It does not inspect a person's current VPN switch.

## Files and assistance

The local package contains the complete notebook, five-page report, PowerPoint, spoken script, figures, CSV tables, `results.json`, and the frozen selection record. `CHANGES.md` answers the requested questions and lists changed cells and document sections. The notebook does not load the result exports. GitHub contains the notebook, README and change summary; the report, presentation and script are supplied in the local package.

OpenAI Codex supported explanations, research, code generation, debugging, local execution, figures and drafts. The students are responsible for reviewing and understanding the work. Complete the individual-contribution fields with the work actually performed.
