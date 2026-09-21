# Dataset:
Name: ds004504 — "A dataset of EEG recordings from: Alzheimer's disease, Frontotemporal dementia and Healthy subjects"
Source: OpenNeuro, https://openneuro.org/datasets/ds004504/versions/1.0.9
Citation: Miltiadous et al., 2023, Data, https://doi.org/10.3390/data8060095
Subjects: 88 total — 36 AD / 23 FTD / 29 CN
Signal: 19-channel EEG (standard 10-20 montage, referential A1/A2), 500 Hz sampling rate, resting-state eyes-closed, mean recording length ≈13.4 min (range 5.1–21.5 min)
Labels: Group ∈ {A (AD), F (FTD), C (CN)}, plus demographic/clinical fields (Age, Gender, MMSE score)

# Graph Neural Networks & Explainable AI for EEG-Based Dementia Classification

[![Dataset](https://img.shields.io/badge/OpenNeuro-ds004504_v1.0.9-blue.svg)](https://openneuro.org/datasets/ds004504/versions/1.0.9)
[![Python](https://img.shields.io/badge/Python-3.10%2B-green.svg)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-Deep%20Learning-red.svg)](https://pytorch.org/)
[![PyG](https://img.shields.io/badge/PyTorch%20Geometric-GNN-orange.svg)](https://pyg.org/)

## 📌 Project Overview
This repository presents an end-to-end machine learning and deep learning framework for multi-class differential diagnosis of **Alzheimer's Disease (AD)**, **Frontotemporal Dementia (FTD)**, and **Healthy Controls (CN)** using resting-state EEG recordings from OpenNeuro dataset `ds004504`.

The project is structured across 5 sequential, modular Jupyter notebooks:
1. `task1_eda.ipynb`: Exploratory Data Analysis & Phenotype/Signal Feature Wrangling.
2. `task2_baselines.ipynb`: Data Ingestion, Functional Connectivity Graph Extraction, and Classical ML Baselines.
3. `task2_proposed_model.ipynb`: Graph Convolutional Network (GCN) Architecture Implementation.
4. `task3_improvement_ablation.ipynb`: Model Optimization, Ablation Study, and Leakage-Safe Cross-Validation.
5. `task3_explainability.ipynb`: Model Interpretability via SHAP and LIME.

---

## 📊 Dataset & Feature Processing

* **Dataset ID:** OpenNeuro `ds004504` (v1.0.9)
* **Cohort Size:** $N = 88$ subjects (36 AD, 23 FTD, 29 CN)
* **Recording Parameters:** 19 channels (10-20 international system), referential montage ($A1/A2$), 500 Hz sampling rate.
* **Feature Extraction & Graph Construction:**
  * **Node Features:** $19 \times 5$ matrix per subject containing per-channel relative band powers ($\delta, \theta, \alpha, \beta, \gamma$) computed from 8-second epochs via Welch Power Spectral Density (PSD).
  * **Connectivity Graphs:** Two functional connectivity matrices per subject:
    1. **Pearson Correlation** (broadband signal).
    2. **Phase Lag Value (PLV)** (alpha band).
* **Feature Caching:** Extracted graph features are saved to `features_cache.npz` (with fallback support to `demo_synthetic_features.npz` if raw dataset acquisition is bypassed).

---

## 📈 Experimental Results

Empirical performance evaluation across classical baselines and GNN architectures:

| Model Architecture | Evaluation Scheme | Macro-F1 | Notes & Key Insights |
| :--- | :--- | :---: | :--- |
| **SVM-RBF** | Single Train/Val/Test Split | **0.595** | Best-performing baseline classifier on tabular features |
| **First GNN (2-layer GCN)** | Single Train/Val/Test Split | **0.458** | Uses Pearson correlation graph; underperforms SVM baseline |
| **Best-Ablation GNN** | Single Train/Val/Test Split | **~0.69 – 0.74\*** | Top performance across 9 single-factor ablation configurations |
| **SVM-RBF** | 5-Fold Grouped CV | **0.487 ± 0.086** | Leakage-safe, subject-grouped baseline mean |
| **Best-Ablation GNN** | 5-Fold Grouped CV | **~0.51 – 0.53\*** | Leakage-safe; paired Wilcoxon test vs. SVM is not statistically significant ($p > 0.05, n_{\text{folds}}=5$) |

*\*Exact F1 score depends on specific validation fold and ablation candidate selection.*

---

## 📂 Execution Pipeline & File Dependencies

Run notebooks sequentially in Google Colab or local Jupyter environments to generate dependent artifacts:

---

## 🛠️ Environment & Data Setup

### 1. Data Directory Setup
* **For `task1_eda.ipynb`:** Create a `data/` directory adjacent to the notebook containing:
  * `data/participants.txt`
  * `data/eeg_technical_metadata.csv`
  * `data/eeg_signal_features.csv`
* **For Tasks 2–3:** Raw dataset is automatically downloaded from OpenNeuro via DataLad directly within `task2_baselines.ipynb`.

### 2. Dependency Installation

**Data Acquisition System Requirements:**
```bash
apt-get install -y git-annex
pip install datalad

Core Machine Learning & EEG Libraries:
pip install numpy pandas scipy scikit-learn seaborn matplotlib plotly xgboost mne

PyTorch Geometric (GNN Support):
pip install torch
pip install torch-geometric torch-scatter torch-sparse -f [https://data.pyg.org/whl/torch-](https://data.pyg.org/whl/torch-)<YOUR_TORCH_VERSION>.html

Explainable AI Frameworks:
pip install shap lime
