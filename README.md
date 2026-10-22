<div align="center">

# 🔐 xnids
### Explainable AI-Based Network Intrusion Detection System
**Deep Neural Network + SHAP on NSL-KDD & UNSW-NB15**

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![SHAP](https://img.shields.io/badge/SHAP-Explainability-00C7B7?style=flat-square)
![Colab](https://img.shields.io/badge/Google%20Colab-T4%20GPU-F9AB00?style=flat-square&logo=googlecolab&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Research%20%2F%20Paper-blueviolet?style=flat-square)

*Bridging the gap between ML accuracy and operational deployability in network security*

</div>

---

## 📌 Overview

Modern intrusion detection systems powered by deep learning achieve high accuracy — but they are **black boxes**. A network analyst cannot trust, audit, or deploy a system that flags traffic as malicious without explaining *why*.

**xnids** addresses this directly. It trains a Deep Neural Network on real network traffic data, then uses **SHAP (SHapley Additive Explanations)** to make every single prediction fully interpretable:

- Which features drove this connection to be classified as an attack?
- Does the model rely on semantically meaningful network features?
- Are the learned patterns consistent across different datasets?

These questions form the **novel contribution** of the accompanying research paper.

---

## 🔬 Novel Contribution

> **"Systematic SHAP-based interpretability analysis of a DNN intrusion detection system across two benchmark datasets, with semantic validation of learned feature attributions."**

No existing NIDS paper provides:
- ✅ Per-prediction feature attribution via SHAP DeepExplainer
- ✅ Per-attack-family SHAP analysis (what drives DoS vs Probe vs R2L vs U2R?)
- ✅ Cross-dataset feature consistency validation (NSL-KDD ↔ UNSW-NB15)
- ✅ Semantic alignment check between SHAP top features and network security domain knowledge

---

## 🏗️ Architecture

```
Input (122 features after OHE + StandardScaler)
        │
        ├─ Dense(256) → BatchNorm → ReLU → Dropout(0.3)
        ├─ Dense(128) → BatchNorm → ReLU → Dropout(0.3)
        ├─ Dense(64)  → BatchNorm → ReLU → Dropout(0.3)
        └─ Dense(32)  → BatchNorm → ReLU → Dropout(0.3)
                │
        ┌───────┴────────┐
  Binary: Dense(1, σ)   Multi-class: Dense(5, softmax)
  BCE Loss              Sparse Categorical CE
```

**Training:**
- Adam optimiser · lr = 1e-3 · Early stopping (patience=10, monitors val AUC)
- Class-weight balancing — prevents ignoring rare U2R/R2L attacks
- ReduceLROnPlateau — halves LR when val_loss plateaus

**SHAP Analysis:**
- DeepExplainer on 500 balanced background samples
- Global importance · Beeswarm · Waterfall · Per-class heatmap · Cross-dataset comparison

---

## 📊 Datasets

| Dataset | Samples (Train) | Samples (Test) | Attack Families | Notes |
|---------|----------------|----------------|-----------------|-------|
| **NSL-KDD** | 125,973 | 22,544 | 22 train / 37 test | Auto-downloaded |
| **UNSW-NB15** | 175,341 | 82,332 | 9 categories | Kaggle / UNSW website |

### NSL-KDD Attack Distribution

| Family | Sub-types | Count | % of Train |
|--------|-----------|-------|------------|
| Normal | — | 67,343 | 53.5% |
| DoS | 13 | 45,927 | 36.5% |
| Probe | 6 | 11,656 | 9.3% |
| R2L | 10 | 995 | 0.8% |
| U2R | 7 | 52 | **0.04%** ← extreme imbalance |

---

## 📈 Results

| Model | Dataset | Accuracy | AUC-ROC | F1 (macro) | MCC |
|-------|---------|----------|---------|------------|-----|
| DNN Binary | NSL-KDD | ~99.4% (train) | ~0.999 (val) | — | — |
| DNN Binary | KDDTest+ | ~80.8% | 0.923 | 0.808 | — |
| DNN 5-class | KDDTest+ | ~82% | — | ~0.65 | — |

> **Note on the train/test gap:** KDDTest+ contains 15 attack families *never seen during training* — this is intentional in the benchmark and widely reported in literature. The AUC of 0.923 is consistent with state-of-the-art results on this split.

### 🔍 SHAP Key Findings

1. **`serror_rate`, `dst_host_serror_rate`, `same_srv_rate`** are the most globally impactful features — aligning with known SYN-flood DoS signatures
2. **DoS attacks** are primarily driven by connection error rates
3. **Probe attacks** are driven by distinct `dst_host_count` and `dst_host_same_srv_rate` patterns — consistent with port scanning behaviour
4. **Cross-dataset validation:** Rate and byte-count features appear in the top-5 of *both* NSL-KDD and UNSW-NB15 models, confirming the DNN learns generalizable network behaviour, not dataset artefacts

---

## 🚀 Quick Start

### Option A — Google Colab (Recommended)

1. Open [colab.research.google.com](https://colab.research.google.com)
2. **File → Upload notebook** → select `NIDS_SHAP_Colab.ipynb`
3. **Runtime → Change runtime type → T4 GPU**
4. **Runtime → Run all**

The notebook auto-downloads NSL-KDD and runs the full pipeline end-to-end.

### Option B — Local (Python 3.11 / 3.12 only)

TensorFlow does not yet support Python 3.14. Use `pyenv` to install a compatible version:

```bash
# Install pyenv (macOS)
brew install pyenv
echo 'eval "$(pyenv init -)"' >> ~/.zshrc && source ~/.zshrc

# Install Python 3.11
pyenv install 3.11.9
cd xnids
pyenv local 3.11.9

# Create venv and install
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Run
python main.py --task binary
python main.py --task multiclass
```

---

## 📁 Project Structure

```
xnids/
│
├── NIDS_SHAP_Colab.ipynb     ← Main notebook (run this on Colab)
│
├── main.py                   ← CLI entry point (local run)
├── config.py                 ← All hyperparameters & settings
├── requirements.txt          ← Python dependencies
│
├── src/
│   ├── data_loader.py        ← NSL-KDD auto-download & loading
│   ├── preprocessor.py       ← OHE + StandardScaler + label encoding
│   ├── model.py              ← DNN architecture (Keras Functional API)
│   ├── trainer.py            ← Training loop + class weights + callbacks
│   └── evaluator.py          ← Metrics + SHAP plots + all visualisations
│
├── data/                     ← Auto-populated on first run
│   ├── KDDTrain+.txt
│   └── KDDTest+.txt
│
├── models/                   ← Saved checkpoints
│   ├── nids_binary.keras
│   └── preprocessor_binary.pkl
│
└── plots/                    ← All generated figures (300 DPI)
    ├── paper_figure.png      ← Combined publication figure
    ├── shap_global.png
    ├── shap_beeswarm.png
    ├── shap_waterfall.png
    ├── shap_heatmap.png
    ├── shap_cross_dataset.png
    ├── cm_nsl_bin.png
    └── roc_nsl_bin.png
```

---

## 🗂️ Generated Figures

| File | Description | Used in Paper |
|------|-------------|---------------|
| `paper_figure.png` | 2×3 combined figure: training, CM, ROC, SHAP global, beeswarm, heatmap | Main results figure |
| `shap_global.png` | Top-15 features by mean \|SHAP\| — global model explanation | §Explainability |
| `shap_beeswarm.png` | SHAP summary plot — impact distribution per feature | §Explainability |
| `shap_waterfall.png` | Single-prediction waterfall — per-sample feature attribution | §Case Study |
| `shap_heatmap.png` | Per-attack-family importance heatmap (5-class model) | §Per-class Analysis |
| `shap_cross_dataset.png` | Feature importance comparison: NSL-KDD vs UNSW-NB15 | §Cross-dataset Validation |
| `cm_nsl_bin.png` | Confusion matrix (counts + normalised) | §Results |
| `roc_nsl_bin.png` | ROC curve + Precision-Recall curve | §Results |

---

## ⚙️ Configuration

Edit `config.py` (local) or the **§1 Configuration** cell (Colab) to change any setting:

```python
MODEL_CFG = dict(
    hidden_layers = [256, 128, 64, 32],  # DNN layer sizes
    dropout_rate  = 0.3,
    learning_rate = 1e-3,
    epochs        = 50,                  # early stopping will terminate sooner
    batch_size    = 512,
    patience      = 10,                  # early stopping patience
)

SHAP_CFG = dict(
    n_background = 500,   # more = slower but more accurate SHAP values
    n_explain    = 500,   # test samples to explain
    max_display  = 15,    # features shown in summary plots
)
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Deep Learning | TensorFlow / Keras |
| Explainability | SHAP (DeepExplainer / GradientExplainer) |
| ML Utilities | scikit-learn |
| Data | pandas, NumPy |
| Visualisation | matplotlib, seaborn |
| Runtime | Google Colab T4 GPU |

---

## 📚 References

1. Tavallaee et al. (2009) — *A Detailed Analysis of the KDD CUP 99 Data Set* — introduced NSL-KDD
2. Lundberg & Lee (2017) — *A Unified Approach to Interpreting Model Predictions* — SHAP
3. Javaid et al. (2016) — *A Deep Learning Approach for Network Intrusion Detection System*
4. Moustafa & Slay (2015) — *UNSW-NB15: A Comprehensive Data Set for Network Intrusion Detection*
5. Vinayakumar et al. (2019) — *Deep Learning Approach for Intelligent IDS*

---

## 📄 Citation

If you use this code or findings in your research, please cite:

```bibtex
@misc{xnids2026,
  author    = {Your Name},
  title     = {xnids: Explainable AI-Based Network Intrusion Detection
               using Deep Neural Networks and SHAP},
  year      = {2026},
  publisher = {GitHub},
  url       = {https://github.com/yourusername/xnids}
}
```

---

<div align="center">
Made for academic research · NSL-KDD benchmark · Google Colab T4 GPU
</div>
