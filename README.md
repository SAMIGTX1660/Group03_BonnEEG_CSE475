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

The dataset is used two ways across the notebooks:

Task 1 (EDA) reads pre-merged tabular CSVs (data/participants.txt, data/eeg_technical_metadata.csv, data/eeg_signal_features.csv) — demographics + whole-scalp relative band power, no raw signal processing needed.
Tasks 2–3 download and process the raw EEG .set files directly from OpenNeuro to extract per-channel features and functional-connectivity graphs (see §5).

