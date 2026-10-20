# VPN and NonVPN Traffic Classification

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SAS2005/VPN-NonVPN-Classification-project/blob/main/TAM400_VPN_Notebook_Ready.ipynb)

TAM400 project by **Shucayb AHmed Said and Ayub Ali Barre, Group 17**.

The project compares three classifiers for recorded network flows and investigates Random Forest regularization. **The recommended practical compromise is RF-206**, with 0.8938 mean ten-fold VPN F1 and a 0.0467 paired training–validation F1 gap. Original RF retains the highest observed validation F1, 0.9016, with a larger 0.0984 gap. The new local search did not find a configuration that improves on RF-206 under the disclosed practical rule. This recommendation exchanges some validation performance for simpler trees and a smaller fit gap; it does not establish higher predictive accuracy.

## Run everything in one Colab notebook

Open the Colab button, choose a fresh **CPU runtime**, then **Runtime > Run all**. No separate results notebook, saved model or external result file is needed. The first cell installs the pinned package versions if needed. If Colab requests a restart after installation, restart and run all again. An internet connection is required for installation and the pinned raw dataset download.

The notebook has **37 cells, including 20 code cells**. All 20 executed successfully in the saved reference run on 19 October 2026, taking about **16 minutes locally**. Colab timing will vary. Running all cells recomputes training, screening, validation, curves and exports. All results are saved in `advanced_results/` in the runtime. Download that folder before the runtime expires if you want to keep the CSV and JSON files.

The local download is named `TAM400_VPN_Advanced_Colab.ipynb`; GitHub retains the established filename `TAM400_VPN_Notebook_Ready.ipynb` for the Colab link. Earlier Drive copies do not update automatically.

Reference environment: Python 3.12.14, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.1.3, matplotlib 3.10.7 and SciPy 1.14.1. Model seeds and shuffled stratified split seeds are 42. Estimator dictionaries retain `n_jobs=None`; CV clones use one estimator job and two parallel fold jobs. This affects scheduling, not the model definition.

## Dataset and fixed preprocessing

The source is [ISCXVPN2016 from the Canadian Institute for Cybersecurity at UNB](https://www.unb.ca/cic/datasets/vpn.html), described by [Draper-Gil et al.](https://doi.org/10.5220/0005740704070414). This project uses the same [pinned public ARFF mirror](https://github.com/shafwanur/thd-ss25-ml-project/blob/b4488cf1022157a319d9ccdfa36f86cb940e2b80/data/15s.arff) as earlier revisions. The mirror has not been independently matched against the official archive. Its SHA-256 verifies unchanged input bytes:

`8b12e8c0cd49f56af28cdaa7c2aa7795ee24de0b1bedf638280e00404f7c1563`

NonVPN is 0 and VPN is 1. The nine unchanged inputs are `duration`, `flowPktsPerSecond`, `flowBytesPerSecond`, `min_flowiat`, `max_flowiat`, `mean_flowiat`, `std_flowiat`, `mean_active` and `mean_idle`.

The 18,758 raw rows contain 1,240 negative feature cells, which become missing values. Cleaning removes 725 repeated input/label rows, then 54 conflicting-label rows, leaving **17,979 rows**. A stratified 80/20 split with seed 42 retains **14,383 training rows and 3,596 held-out rows**, with the original indices unchanged. The training partition contains 6,927 NonVPN and 7,456 VPN flows. There are no identical feature vectors across the retained split.

**Cleaning limitation:** conflicting-label removal happens before splitting and uses labels from the full source. Therefore, the dataset construction is label-dependent. Learned median imputation and SGD scaling are fitted exclusively within each CV training fold, with executed assertions checking their fitted parameters. These pipelines do not undo the earlier population filtering.

Duration is stored in microseconds. Plots convert it to seconds using the [original flow-meter calculation](https://github.com/ahlashkari/ISCXFlowMeter/blob/f3287486f114350b0b79dcfcf1050bc6a85190cf/src/main/java/iscx/cs/unb/ca/ifm/data/BasicFlow.java#L168-L172). The full histogram and 0–15 second zoom keep all 281 longer training flows in the analysis. Low Pearson correlations describe weak linear associations, not absence of useful nonlinear information.

## Part A: Three classification algorithms

- **DummyClassifier:** `strategy="most_frequent"`, with median imputation. It always predicts the majority class, VPN. Its recall is 1.0 but precision is only about 0.5184, so recall alone is misleading.
- **SGDClassifier:** the unchanged `SGDClassifier(random_state=42)` definition with default hinge loss, preceded by fold-fitted median imputation and StandardScaler. Its 0.6369 F1 is below the majority baseline in this comparison. This specific linear model is not competitive here; the result does not establish that all linear methods fail.
- **Original Random Forest:** `RandomForestClassifier(random_state=42)` after median imputation. It supplies the nonlinear reference configuration.

Every value in the following comparison uses the same ten shuffled stratified folds. Full-training scores come from separate fits on all 14,383 training rows. The paired gap is the mean of **fold-training VPN F1 minus that fold's validation VPN F1**. It never subtracts accuracy from F1. The recommended row repeats RF-206 intentionally.

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

## Part B: Controlled Random Forest optimization

All four historical reference forests reproduced their saved ten-fold metrics and fold F1 values to numerical tolerance **before any new configuration was fitted**.

| Parameter | Original RF | Previous tuned RF | RF-206 and recommended | RF-210 |
| --- | --- | --- | --- | --- |
| `n_estimators` | `100` | `200` | `200` | `500` |
| `criterion` | `gini` | `gini` | `entropy` | `entropy` |
| `max_depth` | `None` | `16` | `14` | `14` |
| `min_samples_leaf` | `1` | `2` | `4` | `4` |
| `min_samples_split` | `2` | `2` | `10` | `20` |
| `max_features` | `sqrt` | `6` | `5` | `5` |
| `max_samples` | `None` | `0.75` | `0.7` | `0.7` |

Common forest settings are `bootstrap=True`, `random_state=42`, `class_weight=None`, `max_leaf_nodes=None`, `min_impurity_decrease=0.0`, `min_weight_fraction_leaf=0.0`, `ccp_alpha=0.0`, `monotonic_cst=None`, `n_jobs=None`, `oob_score=False`, `verbose=0` and `warm_start=False`. Full parameter dictionaries for every candidate are exported to `model_parameters.json`.

RF-206 restricts depth, minimum leaf size, split size and the bootstrap draw compared with Original RF. It considers **five features per split versus Original's three** under `sqrt` for nine features. Increased feature sampling, entropy and 200 trees cannot simply be called stronger regularization. The local one-parameter experiments measure effects conditional on the other RF-206 settings. RF-210 changes both tree count and split size relative to RF-206, so that comparison alone cannot isolate their effects.

The local search contains **23 unique configurations**, including RF-206 and 22 one-parameter perturbations. **20 configurations are genuinely new** relative to retained historical grids; RF-206 and the split minima 20 and 30 already appeared there. All 23 are recomputed for a self-contained experiment.

| Parameter varied | Values, with all other settings fixed at RF-206 |
| --- | --- |
| `max_depth` | 10, 12, 14, 16, 18 |
| `min_samples_leaf` | 2, 3, 4, 5, 6, 8 |
| `min_samples_split` | 8, 10, 12, 16, 20, 30 |
| `max_features` | 3, 4, 5, 6, 7 |
| `max_samples` | 0.5, 0.6, 0.7, 0.8, 0.9 |

Three-fold screening uses `StratifiedKFold(n_splits=3, shuffle=True, random_state=42)`. The shortlist combines four high-F1 candidates with four near-best candidates with smaller gaps, deduplicating and filling by F1 to retain eight. The near-best allowance is the explicitly exploratory 0.010 rule with fold SD no greater than 0.015. Six shortlisted configurations are genuinely new; two were screened historically.

These eight join Original RF, Previous Tuned RF, RF-206 and RF-210 in **12 forest comparisons**, plus Dummy and SGD, giving **14 ten-fold models**. All use `StratifiedKFold(n_splits=10, shuffle=True, random_state=42)`. The six reference results are reused from their reproduction phase. This search entails 69 screening CV fits and 140 ten-fold CV fits, plus separate full-training fits and diagnostics.

## Selection sensitivity and recommendation

| Criterion | Rule | Chosen model |
| --- | --- | --- |
| A | Highest mean ten-fold VPN F1 | Original RF |
| B | Historical 0.005 F1 allowance, then smaller gap | Original RF |
| C | Post-hoc exploratory 0.010 allowance, SD ≤ 0.015, then smaller gap among eligible Pareto candidates | RF-206 |
| D | Show the F1/gap Pareto frontier and measured complexity | Several trade-offs; no automatic winner |

The earlier 0.005 rule remains unchanged and still selects Original RF. Its cutoff is 0.896607; Previous Tuned RF is just below it at 0.896546. The current 0.010 alternative is **post-hoc and exploratory**, informed by previous experiments. It was recorded before this local pass, but it was not the historical selection rule. The 0.015 SD cap is a practical guard, not a confidence threshold.

Under criterion C, **RF-206** is the smallest-gap eligible stable choice. The unrestricted smallest forest gap belongs to Local-14 (split minimum 30), at 0.0332, but its validation F1 falls to 0.8861 and fails the 0.010 allowance. RF-210 also falls outside that allowance. Thus minimizing the gap alone would choose a less accurate model.

The highest-F1 genuinely new finalist is **Local-04**, which changes depth from 14 to 18. It reaches **0.8959 F1**, **94.93% full-training accuracy** and a **0.0562 gap**. It improves on RF-206's F1 but increases its gap, and does not beat Original's F1. No new finalist dominates RF-206 on both F1 and gap.

RF-206 reduces the paired gap by **52.5%**, from 0.0984 to 0.0467. Mean leaves per tree fall from about **1,147 to 372** in CV fits. Because RF-206 has twice as many trees, total mean forest node count falls by about **35.1%**, not the per-tree percentage. The recommendation is a practical preference for this measured trade-off.

The paired RF-206 minus Original fold F1 difference averages **−0.007783**, with descriptive SD **0.005937**, range **−0.019630 to +0.003266**, and RF-206 higher in **1 of 10 folds**. Original's observed advantage is therefore fairly consistent. Overlapping model SDs do not erase this loss. Folds share training data and are not independent replicate datasets, so these summaries do not establish statistical significance or equivalence.

## Conditional nested check and diagnostic plots

A three-outer-fold by three-inner-fold check evaluates all **26 fixed forest settings** within each outer training subset: the 23 local settings plus Original, Previous Tuned and RF-210. It uses 234 inner CV fits and three outer refits. Selection stays inside each outer training subset, without reusing the global finalist shortlist. Mean outer F1 is **0.884643**, descriptive SD **0.000542**, with scores **0.884478, 0.885374 and 0.884078**.

This checks the local three-fold selection procedure. It is **conditional on the RF-206 center chosen through past experiments**, the fixed family and global cleaning. It does not estimate the complete historical or three-to-ten-fold workflow without bias. It cannot replace a fresh independent test, and it cannot change the frozen recommendation. See [scikit-learn's nested-CV explanation](https://scikit-learn.org/stable/auto_examples/model_selection/plot_nested_cross_validation_iris.html).

Learning curves compare Original RF, RF-206 and Local-04 at five training sizes. RF-206 validation F1 rises from 0.8401 to 0.8873 as training rows per fit increase from 1,917 to 9,588, while its gap remains 0.0538 at the largest three-fold curve point. Original reaches 0.8943 with a 0.1056 gap. These diagnostics support further data collection but do not guarantee its effect on new capture sessions.

Depth, leaf and split validation curves reuse the controlled screening fits. At depth 10, fold training/validation F1 are 0.9092/0.8729, compared with 0.9409/0.8864 at depth 14. Larger leaf or split minima also eventually lower both scores. These are signs of relative underfitting from excessive restriction. Three-fold curve values must not be presented as ten-fold finalist scores.

The notebook also includes class counts, duration and Pearson plots, model and gap comparisons, the F1/gap Pareto frontier, validation versus measured tree complexity, ten-fold impurity importance and held-out-fold permutation importance. Importance error bars show descriptive fold variation, not confidence intervals. Correlated inputs may substitute for one another, so these figures are not causal explanations and do not justify dropping features after selection. See [permutation importance guidance](https://scikit-learn.org/stable/modules/permutation_importance.html).

## Test status and targets

**There are no new test predictions in this revision.** Test metrics never enter candidate screening, ranking or the final recommendation. The historical partition had already been inspected in earlier versions. Its archived Original RF score is retained only for context:

| Model | Evaluation status | Accuracy | VPN precision | VPN recall | VPN F1 | ROC-AUC |
| --- | --- | --- | --- | --- | --- | --- |
| Original RF | Historical, previously inspected partition | 90.66% | 90.08% | 92.11% | 0.9109 | 0.9677 |
| Recommended tuned RF | No fresh independent evaluation | Unavailable | Unavailable | Unavailable | Unavailable | Unavailable |

The desired 92–93% training accuracy and 0.91–0.92 validation F1 were not achieved by the recommendation: its actual values are **93.76% and 0.8938**. No ten-fold forest finalist reached 0.91 F1. Independent test accuracy is unavailable, so the 91–92% test target cannot be assessed. The targets did not filter configurations or force scores.

## Limitations and next steps

Repeated tuning uses the same training data, so ten-fold finalist results are selected comparison estimates. The conditional nested check cannot undo earlier adaptive choices. Global conflicting-label cleaning uses labels before splitting. Capture/session IDs are absent, so related-flow leakage cannot be excluded. After diagnostic median/IQR scaling, 3,872 training rows have a neighbor within distance 0.01, including 73 with a differently labelled nearest neighbor. The arbitrary threshold is a warning for investigation, not proof of shared sessions. No observations were removed for this diagnostic.

Strong skew, extreme values, duration concentrations and zero-heavy activity features may reflect capture conditions. The data are historical, and the public mirror's provenance has not been independently validated against the official archive. The bounded local search cannot rule out better untested configurations. **Overfitting is reduced in the observed gap, not eliminated.**

No independent compatible dataset was obtained and validated. [CICDarknet2020 reuses ISCXVPN2016](https://www.unb.ca/cic/datasets/darknet2020.html), so it cannot supply independent VPN evidence. [MIT VNAT](https://www.ll.mit.edu/r-d/datasets/vpnnonvpn-network-application-traffic-dataset-vnat) is a possible separate source, but its supplied feature file uses a different representation. Reproducing the nine features, checking units and evaluating frozen models on new, session-separated captures are future work.

This is an offline experiment on measured flow summaries, not a detector of a computer's current VPN switch.

## Deliverables and experiment tracking

The complete local package contains the executed notebook, five-page report, PowerPoint, two-presenter speaking script, eleven figures, full CSV tables and JSON records. `CHANGES.md` answers the requested 20 audit questions. The notebook creates all result exports itself and never imports them.

Key exports are `required_comparison.csv`, `finalist_comparison.csv`, `finalist_fold_scores.csv`, `oat_screening.csv`, `oat_screening_fold_scores.csv`, `model_parameters.json`, `selection_criteria.csv`, `paired_fold_differences.csv`, `pareto_candidates.csv`, `nested_outer_results.csv`, `nested_inner_fold_scores.csv`, `final_selection.json` and `results.json`. Earlier experiments remain preserved in the local history folders and repository history. Their original 0.005 selection outcome has not been rewritten.

OpenAI Codex supported explanations, research, code generation, debugging, local execution, figures and drafts. The students are responsible for reviewing and understanding the work, and for describing individual contributions accurately.
