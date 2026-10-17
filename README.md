# VPN and NonVPN Traffic Classification

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SAS2005/VPN-NonVPN-Classification-project/blob/main/TAM400_VPN_Notebook_Ready.ipynb)

This TAM400 project compares classifiers using nine numerical network-flow measurements. The latest experiment seeks a Random Forest with strong validation performance and a smaller training–validation gap. All code, training, tuning, plots and saved results are in one notebook. The saved outputs come from one completed local execution.

## Run the project

Click the Colab button, use a CPU runtime and select **Runtime > Run all**. Alternatively, download `TAM400_VPN_Notebook_Ready.ipynb` from this repository and upload it to Colab. Save a copy to Google Drive to keep changes. The earlier Drive notebook is a separate copy and does not sync automatically.

The notebook downloads only the raw feature file. It does not import a saved model or an external results file. It evaluates **203 settings with five-fold stratified CV**, then fits the selected configuration and evaluates the test set once. A separate five-value leaf-size curve is diagnostic and cannot change selection. The local code execution took about 29 minutes, excluding the pause for internal review. Colab runtime depends on the machine.

## Measured comparison

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

The selected forest is chosen using training-data cross-validation only. The highest mean validation VPN F1 is **0.8976**. Dividing that candidate's fold standard deviation by the square root of five gives an allowance of **0.0029**, so candidates scoring at least **0.8947** are eligible. Among the **9** eligible settings, the selected forest has the smallest nonnegative fold-training minus validation F1 gap. Ties use higher validation F1, then fixed candidate order.

This allowance is a practical model-selection heuristic, not a confidence interval or proof of equal performance. The candidate list, random seeds and rule were fixed before fitting. The requested target ranges did not select candidates, and no test metric entered this rule.

Relative to the default in this five-fold run, selected validation F1 changes by **-0.0027**, and the training–validation F1 gap changes from **0.1024** to **0.0764**. Full-training accuracy changes from **99.99%** to **96.73%**. These results must be assessed together; lower training accuracy alone does not establish better generalization.

| Selected parameter | Value |
|---|---|
| `criterion` | `gini` |
| `max_depth` | `16` |
| `max_features` | `6` |
| `max_samples` | `0.75` |
| `min_samples_leaf` | `2` |
| `min_samples_split` | `2` |
| `n_estimators` | `200` |

## Desired ranges and actual outcome

| Metric | Desired range | Observed | Within range |
|---|---|---:|---|
| Full-training accuracy | 92–93% | 96.73% | No |
| Mean validation VPN F1 | 0.91–0.92 | 0.8949 | No |
| Test accuracy | 91–92% | 90.24% | No |

No candidate in this bounded search reached 0.91 mean validation VPN F1, so no searched candidate met all the desired ranges. This does not rule out every possible Random Forest configuration.

The test set is evaluated only after the selection is frozen. Its result does not trigger another search. Lower training accuracy is not an objective by itself. Both accuracy and F1 are recorded so their gaps are compared using the same metric.

## What the search covers

The fixed parameter space contains depth 6, 8, 10, 12, 16, 20 or unlimited; leaf minimum 1, 2, 4, 8 or 16; split minimum 2, 10 or 20; feature sampling sqrt, six or all nine; bootstrap sample fraction 0.2, 0.35, 0.5, 0.75 or the full row count; and Gini or entropy splitting. Two hundred combinations are sampled without replacement using seed 42, each with 200 trees. Three controls retain the original 100-tree default, the previous 100-tree tuned configuration and a 200-tree default. The search uses GridSearchCV on that fixed list.

The same cleaned data, nine inputs and 80/20 stratified split with seed 42 are retained. Five shuffled stratified folds replace the previous three-fold procedure. All models in this comparison use those same five folds. Preprocessing is learned inside each fold. Bootstrap sampling is with replacement within the training fold and does not change the held-out test partition. The prediction threshold remains unchanged.

## Dataset and sources

- [Official ISCXVPN2016 dataset description, UNB/CIC](https://www.unb.ca/cic/datasets/vpn.html)
- [Dataset research paper](https://doi.org/10.5220/0005740704070414)
- [Exact pinned public feature-file copy](https://github.com/shafwanur/thd-ss25-ml-project/blob/b4488cf1022157a319d9ccdfa36f86cb940e2b80/data/15s.arff)
- [Original flow-meter implementation documenting microseconds](https://github.com/ahlashkari/ISCXFlowMeter/blob/f3287486f114350b0b79dcfcf1050bc6a85190cf/src/main/java/iscx/cs/unb/ca/ifm/data/BasicFlow.java#L168-L172)
- [RandomForestClassifier parameter documentation](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)

The data checksum is `8b12e8c0cd49f56af28cdaa7c2aa7795ee24de0b1bedf638280e00404f7c1563`. It fixes the exact mirror bytes; the mirror has not been independently compared with the official registration-gated archive. The raw file has 18,758 flows. Cleaning removes 779 duplicate/conflicting rows, leaving 17,979: 14,383 training and 3,596 test flows.

Duration is converted from microseconds to seconds only for its histogram. The full-range plot and labelled zoom keep longer flows in the experiment. Pearson correlation measures linear association; small values do not exclude useful nonlinear relationships.

## Reproducibility and limitations

The final run used Python 3.12.14, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.1.3, matplotlib 3.10.7. Versions are printed in the notebook. Other runtime versions can produce small differences.

The test partition was examined in earlier revisions. This final evaluation therefore uses a reused development test, not a fresh independent holdout. Re-running does not erase that history. The same CV folds were used to search and select, so selected CV results can be optimistic. Related sessions may cross the split, conflict removal excludes some difficult cases, and the dataset is historical. Independent new captures are needed before claiming performance on new networks.

This is an offline classification experiment requiring nine measured features. It does not inspect a person's current connection or detect their VPN switch directly.

## Files and assistance

The local package contains the complete notebook, five-page report, PowerPoint, spoken script, figures, CSV exports and `results.json`. `CHANGES.md` lists the revised cells and document sections. These result exports are for inspection; the notebook does not load them. GitHub contains the complete notebook and README; the report, slides and script are supplied separately.

OpenAI Codex supported explanations, research, code, debugging, local execution, figures and drafts. The students are responsible for reviewing and understanding the work. Complete the individual-contribution fields with the work actually done by each student.
