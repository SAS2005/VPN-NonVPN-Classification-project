# VPN and NonVPN Traffic Classification

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1YyvYGFFRzNyneBJbhtM6DHIiM0HL6wOe?usp=sharing#scrollTo=vpn-33)

This TAM400 project investigates whether machine learning can distinguish VPN and NonVPN network flows using nine numerical traffic measurements. We compare a simple reference model with SGDClassifier and Random Forest, then evaluate the selected model.

All data preparation, training, model comparisons, tables, graphs and final results are contained in **one notebook**. Random Forest achieved **90.77% test accuracy** and a **VPN F1-score of 0.9120** in the saved results.

## Run in Google Colab

1. Click **Open in Colab** above to open the original project notebook shared by the authors.
2. Use a normal Python CPU runtime. A GPU is not required.
3. Run the cells from top to bottom, or select **Runtime > Run all**.
4. Review the plots and tables below each cell. Section 15 prints the final results from that run.
5. Save a copy to your Google Drive if you want to keep your changes and new outputs.

The notebook downloads the raw feature file automatically. Internet access is needed for that download and for installing any missing libraries. No Google Drive mount, separate notebook, saved model or external results file is required.

The notebook in this repository contains the cells and saved outputs copied from the linked Google Colab project. The Drive notebook and this repository copy do not sync automatically.

You can also download [TAM400_VPN_Notebook_Ready.ipynb](TAM400_VPN_Notebook_Ready.ipynb) and upload it to Colab. GitHub displays the code and saved outputs without running the experiment.

## Dataset and sources

The project uses a public copy of the **ISCXVPN2016** feature file for a **15-second flow timeout**. The original dataset is described by the Canadian Institute for Cybersecurity at the University of New Brunswick, Canada.

- [Official dataset description](https://www.unb.ca/cic/datasets/vpn.html)
- [Research paper: Characterization of Encrypted and VPN Traffic using Time-related Features](https://doi.org/10.5220/0005740704070414), G. Draper-Gil et al., ICISSP 2016, pp. 407–414.
- [Exact version-pinned feature-file copy used by the notebook](https://github.com/shafwanur/thd-ss25-ml-project/blob/b4488cf1022157a319d9ccdfa36f86cb940e2b80/data/15s.arff)

The code checks the downloaded file against this SHA-256 checksum:

```text
8b12e8c0cd49f56af28cdaa7c2aa7795ee24de0b1bedf638280e00404f7c1563
```

The checksum confirms the expected copy is used. This public copy has not been independently compared with the university archive. The official download route requires registration. The 15-second value describes the flow timeout, not the total capture duration. The dataset itself is downloaded at runtime and is not stored in this repository.

## Inputs and preparation

Each row represents a network flow. The target is **NonVPN = 0** or **VPN = 1**, derived from the original application label. That label is excluded from the inputs.

The nine input features are:

```text
duration, flowPktsPerSecond, flowBytesPerSecond,
min_flowiat, max_flowiat, mean_flowiat, std_flowiat,
mean_active, mean_idle
```

| Data step | Count |
|---|---:|
| Original flows | 18,758 |
| Invalid negative values replaced with missing values | 1,240 cells |
| Duplicate or conflicting rows removed | 779 |
| Cleaned flows | 17,979 |
| Training flows | 14,383 |
| Test flows | 3,596 |

We use a stratified 80/20 split with `random_state=42`. Missing values are replaced with medians learned inside each training pipeline. SGDClassifier also uses StandardScaler. Preprocessing is fitted separately within each cross-validation training fold.

The exploratory section contains training-class counts, a flow-duration histogram and a Pearson correlation table. All nine inputs are retained.

## Model comparison

We compare models using three-fold stratified cross-validation on the training set and select using mean VPN F1-score.

| Model | Mean validation VPN F1 |
|---|---:|
| Random Forest | 0.8935 |
| DummyClassifier reference | 0.6828 |
| SGDClassifier | 0.6003 |

The reference model always predicts the most common training class, VPN. Random Forest combines 100 trees with the default settings and a fixed random seed. SGDClassifier learns a linear decision boundary in this setup and scored below the reference model.

GridSearchCV tests Random Forest `max_features` values of 4, 6 and 8:

| Setting | Mean validation VPN F1 | Fold standard deviation |
|---|---:|---:|
| Default (`sqrt`) | 0.8935 | 0.0040 |
| 4 | 0.8932 | 0.0044 |
| 6 | 0.8932 | 0.0039 |
| 8 | 0.8917 | 0.0016 |

The notebook identifies 6 as the best grid candidate using unrounded scores. Settings 4 and 6 both round to 0.8932. None improves on the original Random Forest, so the default forest is retained. These small differences do not establish a reliable advantage on new traffic.

## Saved test results

The following values come from the outputs saved in the uploaded notebook. Running the notebook again recomputes them.

| Metric | Random Forest | Reference model |
|---|---:|---:|
| Accuracy | 90.77% | 51.84% |
| VPN precision | 90.10% | 51.84% |
| VPN recall | 92.33% | 100.00% |
| VPN F1-score | 0.9120 | 0.6828 |
| ROC-AUC | 0.9678 | 0.5000 |

The confusion matrix has actual classes in rows and predicted classes in columns:

| Actual / predicted | NonVPN | VPN |
|---|---:|---:|
| NonVPN | 1,543 | 189 |
| VPN | 143 | 1,721 |

There are **189 false VPN alarms** and **143 missed VPN flows**, giving 332 errors among 3,596 test flows. The reference model finds every VPN flow because it labels every flow as VPN, which also produces false alarms for all NonVPN flows.

## Reproducibility and limitations

The saved environment output reports Python 3.13.16, scikit-learn 1.6.1, pandas 2.2.3 and NumPy 2.1.3. The notebook also uses matplotlib. Its first cell prints the environment versions and installs missing libraries. It does not pin all package versions, so a different Colab environment may produce slightly different results even with a fixed random seed.

- Training accuracy is 99.993%, about 9.23 percentage points above test accuracy. This indicates overfitting.
- The test partition was examined in earlier project versions. It is a development test, not a new independent evaluation.
- Removing conflicting observations excludes some difficult cases. These results describe the cleaned dataset.
- Session identifiers are unavailable, so related flows may appear in both partitions.
- The dataset is historical. A stronger next step would evaluate independent new traffic captures.

This is an offline classification experiment. It requires the nine measured features and does not directly inspect a person's current connection or determine whether their VPN is switched on.

## AI acknowledgment

OpenAI Codex supported explanations, data-source research, code generation and debugging, local execution of an earlier project version, figures and drafting. The students are responsible for reviewing the work and understanding the submitted methods and results. Any remaining individual contribution fields in the notebook should be completed with the actual information before submission.
