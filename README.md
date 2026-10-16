# VPN and NonVPN Traffic Classification

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SAS2005/VPN-NonVPN-Classification-project/blob/main/TAM400_VPN_Notebook_Ready.ipynb)

This TAM400 project classifies VPN and NonVPN network flows using nine numerical measurements. The revised notebook contains all data preparation, training, tuning, graphs and final results in one place. All saved results come from the same completed local run.

## Run the complete project

1. Click **Open in Colab** above.
2. Use a normal CPU runtime and choose **Runtime > Run all**.
3. Review every table and graph beneath its code cell. Section 15 prints the final comparison.
4. Save a copy to Google Drive to keep any changes you make.

The Colab button opens this revised GitHub notebook. The earlier Drive notebook does not sync automatically and still contains its separate copy.

The notebook downloads only the raw feature file. All training, search, prediction and results are computed in this one notebook. Saved outputs are included. No saved model or separate results file needs to be imported. The 64-setting, three-fold search requires 192 CV fits. Runtime depends on the machine.

## Final model comparison

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

The final choice is **Tuned RF**: `max_depth=20`, `min_samples_split=10`, `min_samples_leaf=1`, `max_features="sqrt"`, with 100 trees. The changed controls are maximum depth (unlimited to 20) and minimum split size (2 to 10).

The highest mean CV F1 was 0.8949. Its fold standard deviation divided by the square root of three gave an allowance of 0.0027. This made the eligibility threshold 0.8922. Among 9 eligible settings, the selected model had the smallest fold-training/validation F1 gap. Higher mean CV F1 and then grid order break ties. This is a practical selection heuristic, not a test of statistical equivalence.

The selected model gives up 0.0015 mean validation F1 relative to the default forest. Its training–validation F1 gap decreases from 0.1054 to 0.0844. This supports reduced overfitting in this experiment, but a substantial gap remains. The test scores did not select the parameters.

## Notebook and revisions

- [Complete executed notebook](TAM400_VPN_Notebook_Ready.ipynb)
- [Detailed revision guide](CHANGES.md)

The saved notebook includes all outputs. It downloads the raw feature file and recomputes its results without importing an external result file. The report and presentation are supplied separately.

## Data and interpretation

The [ISCXVPN2016 dataset](https://www.unb.ca/cic/datasets/vpn.html) is described by the University of New Brunswick. The notebook uses [this pinned feature-file copy](https://github.com/shafwanur/thd-ss25-ml-project/blob/b4488cf1022157a319d9ccdfa36f86cb940e2b80/data/15s.arff), checked with SHA-256 `8b12e8c0cd49f56af28cdaa7c2aa7795ee24de0b1bedf638280e00404f7c1563`. Its identity with the registration-gated original archive has not been independently checked.

The [original ISCXFlowMeter code](https://github.com/ahlashkari/ISCXFlowMeter/blob/f3287486f114350b0b79dcfcf1050bc6a85190cf/src/main/java/iscx/cs/unb/ca/ifm/data/BasicFlow.java#L168-L172) documents duration in microseconds. The histogram divides it by 1,000,000 and labels seconds. It shows the complete range and a clearly labelled 0–15 second zoom. All longer flows remain in the experiment. Low Pearson correlation means weak linear association, not that a feature has no predictive value.

## Reproducibility and limitations

The run used Python 3.12.14, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.1.3, matplotlib 3.10.7. The notebook prints installed versions. A different runtime may produce slightly different results, even with the same random seed.

The search and final selection use training-fold results only. The test partition was already viewed in earlier versions and in an initial strict-maximum-CV search. The rule was then revised from training-fold diagnostics and frozen before the final run. This is a reused development test, not independent confirmation. The selected CV score also has selection optimism. Related sessions may cross the split, conflicting rows were removed, and the traffic is historical. New independent captures are needed before deployment claims.

OpenAI Codex supported research, explanations, code, debugging, local execution, figures and document drafts. The students remain responsible for understanding and explaining the work and completing their actual contribution details.
