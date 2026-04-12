# Subject Identity Strongly Shapes EEG Spectral Features and Can Inflate Machine-Learning Performance

**Companion code for:** Ugail & Howard (2026). Subject Identity Strongly Shapes EEG Spectral Features and Can Inflate Machine Learning Performance

---

## Overview

Our central argument is that subject identity explains 70–81% of EEG spectral band-power variance across independent datasets, and that this same identity structure can inflate downstream EEG machine-learning evaluation when train/test splits are not enforced at the subject level.

The notebook reproduces all the work reported across four public EEG datasets.

---

## Datasets

You must download and preprocess each dataset before running the notebook. All datasets are publicly available.

| Dataset | Subjects | Paradigm | Download |
|---------|----------|----------|----------|
| **Mumtaz MDD** | 63 | Resting-state (clinical EEG, 19-ch) | [figshare](https://doi.org/10.6084/m9.figshare.4244171) |
| **SignEEG v1.0** | 70 | Signature imagery (5-ch Emotiv Insight) | [Zenodo](https://zenodo.org/records/8332198) |
| **Longitudinal ERP** | 395 | RSVP face-target (57-ch, 4 sessions over 199 days) | [figshare](https://springernature.figshare.com/articles/dataset/27201003) |
| **PhysioNet EEGMMIDB** | 103 | Motor imagery (64-ch, 14 runs) | [PhysioNet](https://www.physionet.org/content/eegmmidb/1.0.0/) |

The preprocessed datasets to replicate the results are available at: 
https://drive.google.com/drive/folders/1LdPIw7vNINWq_PIYkj9-S0nVJ5MzJtag?usp=sharing


---

## Repository structure

```
.
├── EEG_Analysis.ipynb          ← Main analysis notebook (this repo)
├── preprocessing/
│   ├── preprocess_mumtaz.ipynb
│   ├── preprocess_signeeg.ipynb
│   ├── preprocess_longitudinal.ipynb
│   └── preprocess_physionet.ipynb
├── data/                       ← Created by preprocessing; not tracked by git
│   ├── Mumtaz/processed/
│   ├── signeeg/processed/
│   ├── longitudinal_erp/processed/
│   └── physionet_eegmmidb/processed/
└── results/                    ← Created by notebook; not tracked by git
    ├── TABLE1_identification.csv
    ├── TABLE2_verification.csv
    ├── TABLE3_distance.csv
    ├── TABLE4_cross_session.csv
    ├── TABLE5_transfer.csv
    ├── TABLE6_mdd_leakage.csv
    ├── F_mdd_leakage.json
    ├── I_variance_longitudinal.json
    ├── I_variance_physionet.json
```

Each processed dataset folder contains:

```
processed/
├── feature_table.csv      ← One row per segment; columns include subject_key, session/run, features
├── manifest.json          ← Dataset metadata (n_subjects, n_segments, channel list, sfreq, …)
├── X_segments.npy         ← Raw segment array (optional, used by preprocessing only)
└── groups.npy             ← Subject group labels (optional)
```

---

## Running the notebook


### Google Colab

1. Upload `EEG_Analysis.ipynb` to Colab or open it from GitHub.
2. Mount your Google Drive: add `from google.colab import drive; drive.mount('/content/drive')` before the configuration cell.
3. In the configuration cell (Section 0), set:
   ```python
   BASE_DIR = Path("/content/drive/MyDrive/your/path/to/data")
   ```
4. Run all cells. Runtime is approximately 20–30 minutes on a CPU instance. GPU (A100 or equivalent is strongly recommended).

---

## Notebook sections

| Section | Contents | Runtime |
|---------|----------|---------|
| 0 | Configuration — set `BASE_DIR` | < 1 s |
| 1 | Imports | < 1 s |
| 2 | Load and fix datasets | < 30 s |
| 3 | Shared helper functions | < 1 s |
| A | Closed-set identification (10 repeats × 4 datasets) | ~5 min |
| B | Biometric verification, AUC and EER (10 repeats × 4 datasets) | ~5 min |
| C | Distance decomposition, two-sided Mann-Whitney U | ~2 min |
| D | Cross-session identity stability — Longitudinal ERP | ~2 min |
| E | Cross-paradigm identity transfer — PhysioNet and SignEEG | ~2 min |
| F | MDD leakage simulation (10 repeats + 200 permutations) | ~7 min |
| G | Summary tables (printed and saved as CSV) | < 1 s |
| I | Variance decomposition + ICC, subject-level bootstrap (Longitudinal + PhysioNet) | ~5 min |

---

## Key results reproduced

| Result | Value |
|--------|-------|
| Subject identity — % of spectral variance (Longitudinal ERP) | 70.2% |
| ICC(2,1) with 95% CI, subject-level bootstrap | 0.702 [0.609, 0.737] |
| Subject identity — % of spectral variance (PhysioNet replication) | 80.7% |
| ICC(2,1), PhysioNet | 0.807 [0.778, 0.822] |
| Verification AUC (all four datasets) | ≥ 0.999 |
| Closed-set identification vs chance | 27× to 162× |
| Cross-session accuracy at 199 days (Day 1 → Day 200) | 75.0% |
| MDD AUC inflation under naive segment split | +0.061 (p = 0.0002, two-sided) |
| Identity-derived MDD AUC (no clinical labels during training) | 1.000 |
| Permutation test p-value (correct protocol, N = 200) | < 0.005 |

---

## Evaluation protocol

All biometric experiments use strict subject-wise held-out protocols with five explicit leakage-prevention guarantees:

1. Subjects are assigned to training or test groups **before** any feature operation.
2. Prototypes are constructed **separately** within each group from within-group segments only.
3. Positive and negative pairs are formed **independently** within each group — no pair contains subjects from both sides.
4. All normalisation statistics (StandardScaler) are estimated on **training data only**.
5. AUC and EER are computed **exclusively** on subjects never seen during training.

For closed-set identification, segments are split 70/30 within each subject independently. This means the same subject appears in both training and test; the results characterise biometric discriminability, not held-out-subject generalisation.

---

## Feature extraction

For each non-overlapping 2-second segment:

- **Log absolute band power** in five bands: δ (0.5–4 Hz), θ (4–8 Hz), α (8–13 Hz), β (13–30 Hz), γ (30–45 Hz)
- **Relative band power** normalised within each epoch over 0.5–45 Hz
- **Hemispheric asymmetry** (log right − log left) for all homologous electrode pairs in each montage

Asymmetry pairs: AF3–AF4 and T7–T8 for SignEEG (Emotiv Insight); standard 10–20 homologous pairs for the remaining datasets.

No epoch rejection beyond the segmentation boundary is applied. No cross-subject normalisation is used at any stage.

---


## License

The code in this repository is released under the MIT License.  
The datasets are subject to their own terms — see each dataset's original source for details.
