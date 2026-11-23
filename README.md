# hybrid-ids-minority-overlap

Hybrid ML/DL intrusion detection for **minority** and **overlapping** attack classes on **UNSW-NB15** and **BoT-IoT** (imbalance handling + metric learning + LightGBM).

> Deadline: 2026-12-05 · Report: 15 pages max, IEEE template

## 1. Goal

Improve on the reference paper's two-CNN hierarchy, which relies on class weights only and fails on rare, overlapping classes (Analysis, Backdoor, Worms, Shellcode). We:

1. **Handle imbalance**: Focal Loss, Class-Balanced Loss, SMOTE / SMOTE-Tomek (GAN optional).
2. **Reduce class overlap**: metric learning (Triplet Loss or Prototypical Network) to build embeddings.
3. **Combine** embeddings and the best imbalance method in a single **LightGBM** model.
4. **Prove each contribution** with an ablation study (3 seeds, mean ± std).

Main metric: **macro F1**. Also reported: accuracy, per-class recall, weighted F1, PR-AUC.

## 2. Team

| Member | Role | Code |
|---|---|---|
| M1 | Project manager, final report | `run_all.py`, `check_contract.py` |
| M2 | Data and baseline | `src/data.py`, `src/baseline.py` |
| M3 | Class imbalance | `src/imbalance.py` |
| M4 | Class overlap | `src/embeddings.py` |
| M5 | Hybrid model and evaluation | `src/hybrid.py`, `src/evaluate.py` |

## 3. Repository structure

```
hybrid-ids-minority-overlap/
├── README.md
├── run_all.py            # single entry point
├── check_contract.py     # data contract validation
├── config.py             # seed, paths, CLASS_NAMES (single source of truth)
├── requirements.txt
├── src/
│   ├── data.py           # M2
│   ├── baseline.py       # M2
│   ├── imbalance.py      # M3
│   ├── embeddings.py     # M4
│   ├── hybrid.py         # M5
│   └── evaluate.py       # M5
├── data/                 # not tracked (shared Drive / Kaggle Dataset)
└── outputs/              # not tracked
```

Large files are **never** committed (see `.gitignore`).

## 4. Datasets

Tabular network-flow CSVs (not images, not text).

| Dataset | Source | Use |
|---|---|---|
| UNSW-NB15 | https://research.unsw.edu.au/projects/unsw-nb15-dataset | CSV files only (no pcap) |
| BoT-IoT | https://research.unsw.edu.au/projects/bot-iot-dataset | **5% version only** (~1.07 GB, ~3 M rows), subsampled if memory runs out, rare classes kept in full |

Only M2 downloads the raw data. Everyone else uses the files below.

## 5. Data contract (fixed after Day 2)

- One split per dataset: **60 / 20 / 20**, stratified (same class proportions).
- Six files: `X_train`, `y_train`, `X_val`, `y_val`, `X_test`, `y_test`.
- One fixed list of class names and numbers, defined only in `config.py`.
- Preprocessing: mean imputation, categorical encoding, min-max normalization. All **fitted on the training set only**.
- Resampling (SMOTE, GAN) is applied to the **training set only**, never before the split.
- Hyperparameters are tuned on **validation only**. The test set is used **once**, for final numbers.

Validate any delivery with:

```bash
python check_contract.py --dataset unsw
python check_contract.py --dataset bot
```

## 6. Output format

Every method writes to `outputs/{dataset}/{method}/seed{seed}/`:

| File | Content |
|---|---|
| `pred.npy` | Predicted class per test sample, shape `[n_test]` |
| `proba.npy` | Class probabilities, shape `[n_test, n_classes]`, columns in `CLASS_NAMES` order |
| `emb_train.npy`, `emb_val.npy`, `emb_test.npy` | Embeddings per sample (M4 only) |

## 7. Quick start

```bash
git clone -b <your-branch> https://github.com/<org>/hybrid-ids-minority-overlap.git
cd hybrid-ids-minority-overlap
pip install -r requirements.txt

# full pipeline
python run_all.py --dataset unsw --seed 42

# single steps
python run_all.py --dataset bot --steps imbalance embeddings --seed 42
```

Pipeline steps: `data → baseline → imbalance → embeddings → hybrid → evaluate`.

Each module exposes one function: `run(dataset, seed)`.

## 8. Experiments

### Ablation (mean ± std over 3 seeds, per dataset)

| Configuration | Macro F1 | Accuracy | Recall: Analysis | Backdoor | Worms | Shellcode |
|---|---|---|---|---|---|---|
| Baseline (two-step CNN) | | | | | | |
| + Imbalance (best of M3) | | | | | | |
| + Overlap only (M4 embeddings) | | | | | | |
| + Hybrid (embeddings + imbalance + LightGBM) | | | | | | |

The baseline is expected to reach about 80.65% accuracy on UNSW-NB15, close to the reference paper.

### Overlap analysis

t-SNE before / after metric learning (same sample, same `random_state`, 5,000 to 10,000 points) on Analysis, Backdoor, DoS, Exploits and Worms, plus a numeric measure (e.g. silhouette score).

## 9. Workflow rules

- One branch per member; only M1 merges into `main`.
- Save models and outputs to the shared Drive after every training run.
- Add one line per task to the shared log (who, what, date).
- Results are frozen on **Dec 1** (tag `v1-final`).

## 10. Environment

Python, scikit-learn, imbalanced-learn, LightGBM, SHAP, and one deep learning library (chosen at Meeting 1). Run on Kaggle or Google Colab.

## 11. References

- Moustafa & Slay, *UNSW-NB15: a comprehensive data set for network intrusion detection systems*, MilCIS 2015.
- Koroniotis et al., *Towards the development of realistic botnet dataset in the Internet of Things for network forensic analytics: Bot-IoT dataset*, Future Generation Computer Systems, 2019.
- Reference paper of the task (add full citation).

The full bibliography is in the report (`refs.bib`).

## 12. License and data terms

Code: add a license (e.g. MIT). The datasets are free for academic use and must be cited as above.
