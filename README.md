Group 03:
1. Muhammed Jabed Iqbal Sami
2. Kasheef Ahmed Nihal
3. Sumaiya Rahman
4. Tasmia Rahman

# Dataset:
Name: ds004504 — "A dataset of EEG recordings from: Alzheimer's disease, Frontotemporal dementia and Healthy subjects"
Source: OpenNeuro, https://openneuro.org/datasets/ds004504/versions/1.0.9
Citation: Miltiadous et al., 2023, Data, https://doi.org/10.3390/data8060095
Subjects: 88 total — 36 AD / 23 FTD / 29 CN
Signal: 19-channel EEG (standard 10-20 montage, referential A1/A2), 500 Hz sampling rate, resting-state eyes-closed, mean recording length ≈13.4 min (range 5.1–21.5 min)
Labels: Group ∈ {A (AD), F (FTD), C (CN)}, plus demographic/clinical fields (Age, Gender, MMSE score)

# Environment & Dependencies
Notebooks are written for Google Colab (they mount Google Drive for persistence between notebooks and use !pip install / !apt-get install cells). To run them elsewhere, install the following:

Data acquisition (Task 2/3 notebooks only)
apt-get install -y git-annex
pip install datalad

Core ML / data science
pip install numpy pandas scipy scikit-learn seaborn matplotlib plotly

EEG signal processing
pip install mne

Gradient boosting baseline
pip install xgboost

Graph neural networks (match the torch version installed in your environment)
pip install torch
pip install torch-geometric torch-scatter torch-sparse -f https://data.pyg.org/whl/torch-<YOUR_TORCH_VERSION>.html

Explainability (Task 3b)
pip install shap lime
If not using Colab, replace the google.colab.drive.mount(...) cells with a local path for SAVE_DIR, and remove/skip the Colab-specific install cells (!apt-get, drive.mount).
5. Data Setup
For Task 1 (EDA): place the following files under a local data/ folder next to the notebook:
data/participants.txt
data/eeg_technical_metadata.csv
data/eeg_signal_features.csv
For Tasks 2–3 (baselines / GNN / ablation / explainability): the raw dataset is fetched automatically from OpenNeuro via datalad/git-annex inside task2_baselines.ipynb:
datalad install https://github.com/OpenNeuroDatasets/ds004504.git
cd ds004504
datalad get sub-*/eeg/*.set
From the raw .set files, task2_baselines.ipynb extracts and caches:

Node features: per-channel relative band power (delta/theta/alpha/beta/gamma) from 8-second epochs via Welch PSD → 19×5 matrix per subject
Two connectivity graphs per subject: Pearson correlation (broadband) and PLV (alpha band)

These are cached to features_cache.npz (falls back to a bundled demo_synthetic_features.npz if the real dataset isn't available, clearly flagged as placeholder/demo output).
# How to Run
Run the notebooks in the order listed in §3, top to bottom, in Google Colab (recommended) or a local Jupyter environment with the dependencies from §4 installed.

task1_eda.ipynb — place the 3 CSVs under data/ and run. No dependency on other notebooks.
task2_baselines.ipynb — run first in the modeling chain. Downloads the raw dataset (large: several GB, can take a while), extracts features, trains the 5 baselines, and saves:
features_cache.npz (node features + both connectivity graphs)
baseline_results.json (train/val/test split, all baseline results, best baseline name/score)
task2_proposed_model.ipynb — loads features_cache.npz and baseline_results.json (via Google Drive, SAVE_DIR = /content/drive/MyDrive/EEG_Project_ds004504), trains the first GCN, and saves:
gnn_first_model.pt, gnn_first_result.json
task3_improvement_ablation.ipynb — loads the above, runs the ablation sweep and 5-fold CV, and saves:
gnn_final_model.pt, ablation_results.json
task3_explainability.ipynb — loads gnn_final_model.pt and ablation_results.json, runs SHAP/LIME analysis on selected test subjects.

Important: if not running on Colab, update every SAVE_DIR / cache-file path to a shared local directory so each notebook can find the previous notebook's saved outputs.

The dataset is used two ways across the notebooks:

Task 1 (EDA) reads pre-merged tabular CSVs (data/participants.txt, data/eeg_technical_metadata.csv, data/eeg_signal_features.csv) — demographics + whole-scalp relative band power, no raw signal processing needed.
Tasks 2–3 download and process the raw EEG .set files directly from OpenNeuro to extract per-channel features and functional-connectivity graphs (see §5).

# Results:
| Model | Macro-F1 | Notes |
| :--- | :--- | :--- |
| SVM-RBF (best Task 2 baseline) | 0.595 | Single train/val/test split |
| First GNN (2-layer GCN, correlation graph) | 0.458 | Single split; underperforms SVM baseline |
| Best-ablation GNN (single split) | ~0.69–0.74* | Best of 9 one-factor-at-a-time ablation configs on the validation set |
| SVM, 5-fold CV mean | 0.487 ± 0.086 | Leakage-safe, subject-grouped |
| Best-ablation GNN, 5-fold CV mean | ~0.51–0.53* | Leakage-safe, subject-grouped; paired Wilcoxon vs. SVM **not statistically significant** (p > 0.05, n_folds=5) |
