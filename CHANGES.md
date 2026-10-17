# Final systematic tuning results and changes

The experiment is complete, but the requested reduction in overfitting was not achieved by the final selected model. The frozen rule retains Original RF. No test score chose the model, no target was forced and no improvement is fabricated.

## Requested results A to O

| Item | Measurement | Result |
|---|---|---|
| A | Unique three-fold search configurations | 285 |
| B | Shortlisted new models | 18 |
| C | Models evaluated with ten-fold CV | 20 = 18 + 2 controls |
| D | Final hyperparameters | See parameter table below |
| E | Full-training accuracy | 99.99% |
| F | Full-training VPN F1 | 0.9999 |
| G | Mean ten-fold validation VPN F1 | 0.9016 |
| H | Ten-fold validation F1 standard deviation | 0.0072 |
| I | Paired fold-training–validation F1 gap | 0.0984 |
| J | Test accuracy | 90.66% |
| K | VPN precision | 90.08% |
| L | VPN recall | 92.11% |
| M | VPN F1 | 0.9109 |
| N | ROC-AUC | 0.9677 |
| O | Confusion matrix, rows actual NonVPN/VPN | [[1543, 189], [147, 1717]] |

Item I uses paired scores from the same CV fits. Full-training F1 minus mean ten-fold F1 is 0.0983, a different comparison.

| Parameter | Original RF | Previous Tuned RF | Final RF |
|---|---|---|---|
| `n_estimators` | `100` | `200` | `100` |
| `max_depth` | `None` | `16` | `None` |
| `min_samples_leaf` | `1` | `2` | `1` |
| `min_samples_split` | `2` | `2` | `2` |
| `max_features` | `sqrt` | `6` | `sqrt` |
| `max_samples` | `None` | `0.75` | `None` |
| `criterion` | `gini` | `gini` | `gini` |

## Same-run comparison

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

## Selection and diagnostic evidence

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

The Original and Final learning curves coincide. At the largest three-fold learning-curve size, mean training F1 is 0.9999 and validation F1 is 0.8943, leaving a gap of 0.1056. The previous forest has a smaller 0.0825 gap at that size. Validation improves as more training rows are used, but the remaining separation supports the concern about overfitting.

The depth diagnostic gives training/validation F1 of about 0.7727/0.7694 at depth 4 versus 1.0000/0.8946 with unlimited depth. The leaf diagnostic falls to about 0.8838/0.8586 at leaf size 30. Strong regularization can lower both scores substantially, which is evidence of underfitting in the more restrictive settings. The final model remains too flexible; it is not an underfitted model. These three-fold diagnostics must not be substituted for the ten-fold selection scores.

| Metric | Desired range | Measured | Within range |
|---|---|---:|---|
| Training accuracy | 92–93% | 99.99% | No |
| 10-fold validation VPN F1 | 0.91–0.92 | 0.9016 | No |
| Test accuracy | 91–92% | 90.66% | No |

## Answers to the thirteen questions

### 1. Did the final tuning reduce overfitting?

No. Final RF is Original RF. Its paired F1 gap remains 0.0984. Some regularized candidates have smaller gaps, but lose more validation F1 than the fixed allowance permits.

### 2. Is the training–validation gap smaller than before?

No for the final choice. It is unchanged from Original RF and larger than Previous Tuned RF (0.0733), using the same ten folds.

### 3. Is the final model closer to 92–93% training accuracy?

No. Its full-training accuracy is 99.99%. RF-210 reaches 92.57% but its validation F1 is 0.8904; training accuracy alone did not determine selection.

### 4. Did validation performance remain strong?

The final ten-fold VPN F1 is 0.9016, highest among the 20 evaluated settings, but below 0.91–0.92. This score is useful for comparison, not a guarantee on new networks.

### 5. Did the model begin to underfit?

The final model does not show underfitting: its training F1 remains near 1 while validation is lower. The more restrictive depth and leaf settings lower both training and validation F1, showing underfitting at the strong-regularization end.

### 6. Why was the final model selected?

Only Original RF clears the fixed 0.896607 ten-fold F1 cutoff. The smaller-gap ranking therefore has no other eligible candidate. Previous Tuned RF misses by only 0.00006093, so the choice is sensitive to the practical margin, not evidence of a significant difference.

### 7. Why three-fold CV for the broad search?

It makes the 285-configuration comparison computationally feasible: 855 CV fits instead of 2,850. Stratification keeps class proportions similar and identical folds support consistent comparisons.

### 8. Why ten-fold CV for final validation?

The 20 shortlisted/control settings receive a common, more detailed comparison, fitting on 90% of the training partition per fold. It is more expensive, so it is limited to the shortlist. It does not guarantee a higher score or independence from selection.

### 9. Why exclude the test set from selection?

Using test scores to choose parameters would make the evaluation part of tuning. The choice was frozen before this final test phase, and predictions were reused. Earlier test exposure remains disclosed; the partition is not a fresh holdout.

### 10. Which notebook cells changed?

Cells 1, 3, 8, 16–17 and 19–21 update the purpose, imports, rules, two fixed controls and three-fold screening. Cells 22–33 replace the old search with three stages, an 18-model shortlist, common ten-fold comparison and freeze. Cells 34–37 add learning and depth/leaf curves. Cells 38–43 update the final test phase, three-model matrices and comparison. Cells 44–45 update interpretation and sources. Data/cleaning/split and corrected EDA cells 4–15 retain their underlying methods. All 21 code cells have saved outputs.

### 11. Which report sections changed?

Abstract and Introduction state the unresolved objective and actual outcome. Experimental Setup explains staged three-fold search, shortlist and ten-fold rule. Results and Discussion add the three-model comparison, parameters, curves and cutoff sensitivity. Conclusion states that overfitting was not reduced in the final choice. Methodology retains corrected duration and Pearson explanations. References and acknowledgment are current. The report remains five pages including references.

### 12. Which PowerPoint slides changed?

Physical slides 8–16 cover updated screening, staged search, selection/parameters, learning curves, depth/leaf curves, three-model results, confusion matrices, targets and the unsuccessful outcome. Physical slides 2–7 retain the project/dataset/EDA foundations with current wording. Slides 17–18 provide acknowledgment/sources and questions. Physical slide 1 remains hidden template guidance. The final deck has 17 visible slides.

### 13. Which script sections changed?

Speaking sections for physical slides 8–16 explain the new procedure and measured outcome, including the nearly missed cutoff and remaining overfitting. Sections 2–7 retain the data/EDA explanation. Sections 17–18 and practice questions match the final deck. Speaker assignments are suggestions, not claims about individual project contributions.

## Scope and limitations

The test partition was examined in earlier revisions. It was excluded from this search, but remains a reused development test rather than a fresh independent holdout. The same training data informed screening and final selection, so ten-fold CV is a selection estimate and may be optimistic. Fold standard deviations are descriptive, not confidence intervals. Related sessions may cross the split because session identifiers are unavailable. Removing conflicting rows changes the evaluated population. Historical traffic may differ from current networks. This bounded search cannot exclude better untested configurations. A frozen model evaluated on independent new captures would give stronger evidence.

Search-stage counts exclude controls, full-training refits and diagnostic curve fits. The complete broad table, shortlist, ten-fold table and curve values are supplied as CSV files for inspection, not as inputs to the notebook. The source protocol and selection audit were checked before releasing the final test phase.
