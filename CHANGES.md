# Final frozen model evaluation update

RF-206 remains the selected Tuned Random Forest. This update adds the requested evaluation of the existing test partition, after selection, without changing hyperparameters, features, cleaning, split, seeds, CV results or the default classification rule. DummyClassifier and SGDClassifier remain in the original comparison.

**Reused development-test evaluation — not an independent untouched test.**

The full pipeline fits all 14,383 training rows, including median imputation fitted only there. Predictions cover the existing 3,596 test rows. Original RF is not newly evaluated; its scores and matrix are recovered from the preserved systematic experiment.

| Metric | Original Random Forest | Tuned Random Forest (RF-206) |
| --- | --- | --- |
| Training Accuracy | 99.9930% | 93.7635% |
| Training VPN F1 | 0.999932935 | 0.940615690 |
| Mean 10-fold Validation VPN F1 | 0.901607369 | 0.893824214 |
| Train–Validation F1 Gap | 0.098392631 | 0.046692095 |
| Test Accuracy | 90.6563% | 89.3215% |
| Test VPN precision | 90.0839% | 87.9098% |
| Test VPN recall | 92.1137% | 92.0601% |
| Test VPN F1 | 0.910875332 | 0.899371069 |
| Test ROC-AUC | 0.967685402 | 0.963941807 |

| Model | Actual class | Predicted NonVPN | Predicted VPN |
| --- | --- | --- | --- |
| Original RF (historical) | NonVPN | 1543 | 189 |
| Original RF (historical) | VPN | 147 | 1717 |
| Tuned Random Forest | NonVPN | 1496 | 236 |
| Tuned Random Forest | VPN | 148 | 1716 |

The matrix formulas and scikit-learn scores agree. ROC-AUC uses saved VPN probabilities. The lower accuracy/F1 relative to Original is consistent with the validation ordering. These results cannot establish unbiased independent generalization and cannot change RF-206.

### Verification and files

Only the notebook's historical-test display and final export code cells changed. The other 18 code cells, including data preparation, models, tuning candidates and cross-validation, retain identical source. The existing experiment is reproduced when Run all executes; no additional candidates are introduced.

The report, presentation, speaking script, README and this change record use the same measured test values and the explicit reused-test disclosure. New exports are `reused_test_evaluation.json`, `rf206_reused_test_predictions.csv`, `reused_test_metrics.csv`, `final_train_validation_test_comparison.csv` and the confusion-matrix figure. Exact parameters and prior selection records remain available. All earlier output folders are preserved.

### Protocol

Selection stays frozen at RF-206 before and after prediction. Its probabilities are used only for ROC-AUC and its unmodified class predictions for the remaining metrics. There is no threshold search. Historical full-data conflicting-label cleaning and the previously inspected test partition remain disclosed limitations.

## Preserved advanced optimization record

The earlier advanced analysis preserved prior experiments and changes the recommended practical trade-off through an explicitly post-hoc rule. It does not claim a newly discovered higher-F1 model.

## 1. Were DummyClassifier and SGDClassifier preserved?

Yes. The original majority strategy and default hinge-loss SGD definition remain, with fold-fitted preprocessing. All three algorithms are compared on the same ten folds.

## 2. What are the exact parameters of Original RF?

The full dictionary is printed below and saved in model_parameters.json. It has 100 Gini trees, no depth cap, leaf minimum 1, split minimum 2, sqrt features (3 here), full-size bootstrap draws and seed 42.

## 3. What are the exact parameters of RF-206?

The full dictionary is printed below. It has 200 entropy trees, depth 14, leaf minimum 4, split minimum 10, five features per split, bootstrap fraction 0.70 and seed 42.

## 4. What changes made RF-206 more regularized?

Depth, leaf, split and bootstrap restrictions limit tree fitting. It uses more features per split than Original, not fewer. Criterion and tree count also differ; controlled local sweeps support conditional effects rather than attributing the whole difference to one parameter.

## 5. Was RF-206 reproduced successfully?

Yes. All four reference forests reproduced their historical ten-fold metrics and fold F1 values exactly in the reference environment before any new fits.

## 6. How many additional configurations were tested?

23 local settings were screened, including the center and 22 one-parameter perturbations. Twenty settings were genuinely new relative to retained grids. Eight nonreference settings advanced to ten folds, six of them genuinely new; the final comparison has 12 forests plus Dummy and SGD.

## 7. Was a better Random Forest found?

No new finalist improved the preferred F1/gap compromise. Local-04 improved RF-206 F1 to 0.8959 but increased the gap to 0.0562. Original retained the highest observed F1. RF-206 remains the practical recommendation under exploratory C.

## 8. What was the highest validation VPN F1?

Original RF: 0.901607 mean ten-fold VPN F1. The highest genuinely new finalist was Local-04: 0.895882.

## 9. Which candidate had the best reasonable generalization gap?

RF-206 has the smallest gap within the disclosed 0.010 F1 allowance and stability cap: 0.046692. Local-14 has a smaller unrestricted gap, 0.033183, but its F1 drops to 0.886125 outside that allowance.

## 10. How did the previous rule affect the result?

The unchanged 0.005 rule selects Original. Its 0.896607 cutoff excludes Previous Tuned at 0.896546 and excludes RF-206. This sensitive practical rule is not a significance test.

## 11. Did exploratory analysis change the recommendation?

Yes. Post-hoc criterion C widens the allowance to 0.010 and chooses RF-206. This is explicitly a change in practical preference informed by earlier results, not a new historical prespecification.

## 12. Did the preferred model reduce the observed overfitting gap?

Yes: 0.098393 to 0.046692, a 52.5% relative reduction. Validation F1 also decreases by 0.007783, so this is not proof of better predictive generalization.

## 13. Did any candidate appear to underfit?

The most restrictive settings lower both training and validation F1. For example depth 10 gives 0.9092/0.8729 versus depth 14 at 0.9409/0.8864 in three-fold screening. This supports relative underfitting; no absolute training-accuracy threshold defines it.

## 14. What supports the recommendation?

Identical-fold reproduction, stable 0.008693 fold F1 SD, smaller measured trees, a smaller paired gap, Pareto efficiency and the disclosed tolerance support a practical compromise. Original beats RF-206 in 9 of 10 paired folds, so its observed predictive advantage remains clear descriptively.

## 15. Were the target ranges achieved?

No. Recommended training accuracy is 93.76% and ten-fold F1 is 0.8938. Reused-test accuracy is 89.3215%; independent test accuracy remains unavailable. RF-210 reaches the training range alone but has lower F1; targets never drove selection.

## 16. Were test labels completely excluded from model selection?

Test predictions and scores do not enter the preserved tuning or selection code. The final update uses test labels only to score the already-frozen RF-206. However, it would be inaccurate to claim the complete workflow never depended on held-out labels: preserved conflict cleaning used all source labels before splitting, and the old test had been viewed historically.

## 17. What evaluation limitations remain?

Adaptive selection, global label-dependent cleaning, unknown session grouping, close flows, historical capture artifacts, unverified mirror provenance and no fresh independent test. The 3-by-3 conditional nested result (0.884643) checks only the local selection procedure conditional on the historical center and cannot erase earlier adaptivity.

## 18. Which files were modified?

New advanced notebook, report, presentation, script, README, CHANGES, figures, CSV/JSON tracking and QA records. Older output directories remain byte-identical. GitHub publication is gated on all final artifact checks.

## 19. Did the entire notebook execute successfully?

Yes. All 20 code cells in the 37-cell notebook executed, with actual outputs saved. The complete final evaluation rerun took 964.2 seconds locally.

## 20. Are metrics consistent and reproducible?

All aggregate metrics reconcile with saved fold records. Dataset and split hashes, environment pins, exact settings, historical reproduction checks and execution records are supplied. Final document/presentation consistency is separately checked before publication; other environments may introduce small numerical differences.

## Exact reference parameters

### Original Random Forest

```json
{
  "bootstrap": true,
  "ccp_alpha": 0.0,
  "class_weight": null,
  "criterion": "gini",
  "max_depth": null,
  "max_features": "sqrt",
  "max_leaf_nodes": null,
  "max_samples": null,
  "min_impurity_decrease": 0.0,
  "min_samples_leaf": 1,
  "min_samples_split": 2,
  "min_weight_fraction_leaf": 0.0,
  "monotonic_cst": null,
  "n_estimators": 100,
  "n_jobs": null,
  "oob_score": false,
  "random_state": 42,
  "verbose": 0,
  "warm_start": false
}
```

### RF-206

```json
{
  "bootstrap": true,
  "ccp_alpha": 0.0,
  "class_weight": null,
  "criterion": "entropy",
  "max_depth": 14,
  "max_features": 5,
  "max_leaf_nodes": null,
  "max_samples": 0.7,
  "min_impurity_decrease": 0.0,
  "min_samples_leaf": 4,
  "min_samples_split": 10,
  "min_weight_fraction_leaf": 0.0,
  "monotonic_cst": null,
  "n_estimators": 200,
  "n_jobs": null,
  "oob_score": false,
  "random_state": 42,
  "verbose": 0,
  "warm_start": false
}
```

### RF-210

```json
{
  "bootstrap": true,
  "ccp_alpha": 0.0,
  "class_weight": null,
  "criterion": "entropy",
  "max_depth": 14,
  "max_features": 5,
  "max_leaf_nodes": null,
  "max_samples": 0.7,
  "min_impurity_decrease": 0.0,
  "min_samples_leaf": 4,
  "min_samples_split": 20,
  "min_weight_fraction_leaf": 0.0,
  "monotonic_cst": null,
  "n_estimators": 500,
  "n_jobs": null,
  "oob_score": false,
  "random_state": 42,
  "verbose": 0,
  "warm_start": false
}
```

## Final comparison

| Model | Full-training accuracy | Full-training F1 | 10-fold validation F1 | Fold SD | Paired F1 gap |
| --- | --- | --- | --- | --- | --- |
| Majority baseline (DummyClassifier) | 51.84% | 0.6828 | 0.6828 | 0.0003 | 0.0000 |
| Linear classifier (SGDClassifier) | 58.93% | 0.6420 | 0.6369 | 0.0326 | -0.0008 |
| Original RF | 99.99% | 0.9999 | 0.9016 | 0.0072 | 0.0984 |
| Previous tuned RF | 96.73% | 0.9688 | 0.8965 | 0.0090 | 0.0733 |
| RF-206 | 93.76% | 0.9406 | 0.8938 | 0.0087 | 0.0467 |
| RF-210 | 92.57% | 0.9292 | 0.8904 | 0.0075 | 0.0378 |
| Recommended tuned RF (RF-206) | 93.76% | 0.9406 | 0.8938 | 0.0087 | 0.0467 |

| Model | Validation accuracy | Validation VPN precision | Validation VPN recall | Mean fold-training F1 |
| --- | --- | --- | --- | --- |
| Majority baseline (DummyClassifier) | 51.84% | 51.84% | 100.00% | 0.6828 |
| Linear classifier (SGDClassifier) | 59.22% | 59.07% | 69.39% | 0.6361 |
| Original RF | 89.72% | 89.47% | 90.88% | 1.0000 |
| Previous tuned RF | 89.08% | 88.12% | 91.26% | 0.9699 |
| RF-206 | 88.78% | 87.72% | 91.12% | 0.9405 |
| RF-210 | 88.42% | 87.36% | 90.81% | 0.9282 |
| Recommended tuned RF (RF-206) | 88.78% | 87.72% | 91.12% | 0.9405 |

## Historical preservation

The earlier 64-setting searches, 203-setting five-fold search and 285-setting three-fold search with 20 ten-fold finalists remain retained. The historical 0.005 rule selected Original RF. The advanced protocol was written before the new local fits but after those earlier findings; criterion C therefore remains post-hoc exploratory.

## Revised notebook sections

The notebook has twelve parts: introduction and dataset; preprocessing; EDA; initial algorithms; overfitting; RF tuning; advanced CV; selection sensitivity; learning and validation curves; recommendation; limitations; and conclusion. The advanced revision originally displayed historical test scores only. The final evaluation update above replaces that panel with the frozen RF-206 evaluation and retains Original's historical results.

The report, slides and script explain the same result, include the three original algorithms, and distinguish numerical F1 loss from the practical reduction in the fit gap. Cross-artifact verification is recorded separately from the scientific outputs.
