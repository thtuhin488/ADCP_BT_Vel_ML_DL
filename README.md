# ADCP_BT_Vel_ML_DL

This repository contains processed ADCP datasets, preprocessing notebooks, and ML/DL notebooks to predict the corrected bottom-track velocity proxy (BT_Vel) from ADCP-derived hydraulic, acoustic, and measurement-geometry features.

> v2 notebooks (`*_v2.ipynb`) are the revised versions. v1 notebooks (`LAB_Preprocessing-.ipynb`, `FIELD_Preprocessing.ipynb`, etc.) are retained for reference.

> **Quick access**
>
> - **Open in VS Code for the web:** https://vscode.dev/github/thtuhin488/ADCP_BT_Vel_ML_DL
> - **Open on GitHub.dev:** https://github.dev/thtuhin488/ADCP_BT_Vel_ML_DL
>
> In VS Code for the web, open the **Outline** panel (left sidebar) to navigate headings and fold sections, similar to desktop VS Code.

---

## Contents

- **Data/**
  - `Processed_LAB_Data_with_BS.xlsx`
  - `Processed_Field_Data_with_BS.xlsx`


- **Notebooks/**
  - `LAB_Preprocessing_v2.ipynb`
  - `FIELD_Preprocessing_v2.ipynb`
  - `LAB_ML_v2.ipynb`
  - `FIELD_ML_v2.ipynb`
  - `LAB_DL_v2.ipynb`
  - `FIELD_DL_v2.ipynb`
  - `Model_analysis_revised.ipynb`

- `README.md`
- `requirements.txt`

> All notebook paths are relative (`../Data/`). Update the path variables in the first cell if your directory structure differs.

---

## Data

### Input files (provided in Data/)

- `Processed_LAB_Data_with_BS.xlsx`
- `Processed_Field_Data_with_BS.xlsx`

### Features

Depth, Vel_StdDev, Correlation, Mean_Speed, SNR, Vel_Expected_StdDev, Bin_Distance

Optional supplementary predictor: `BS_rel` (relative bottom-track backscatter strength, extracted from raw `.rsqmb` files)

### Target: BT_Vel

The corrected bottom-track velocity is defined as the horizontal magnitude `sqrt(BT_E² + BT_N²)` and filtered through a staged quality-control workflow:

| Stage | Filters applied |
|-------|----------------|
| `BT_Vel_stage0` | Horizontal magnitude + valid BT lock flag |
| `BT_Vel_stage1` | + direction-consistency filter (**main target**) |
| `BT_Vel_stage2` | + error-velocity filter |
| `BT_Vel_stage3` | + magnitude filter |
| `BT_Vel_stage4` | All filters combined |

`BT_Vel_stage1` is adopted as the main modelling target for both datasets. NaN values (filtered pings) are excluded using `dropna()` — not imputed.

---

## Models

### Linear baselines (in `*_ML_v2.ipynb`)

- `OLS` — Ordinary Least Squares
- `Ridge` — Ridge regression
- `Lasso` — Lasso regression

### Classical ML (in `*_ML_v2.ipynb`)

All models are trained inside a `StandardScaler` pipeline with 5-fold quantile-stratified CV (`KBinsDiscretizer(n_bins=10)`) and report R²/MSE:

- `Random Forest Regressor`
- `Gradient Boosting Regressor`
- `XGBoost Regressor`
- `LightGBM Regressor`
- `CatBoost Regressor`
- `Stacking Regressor` (RF + GBM base; Ridge meta-learner)

### Deep Learning (in `*_DL_v2.ipynb`, TensorFlow/Keras)

- `ANN: Dense 64→32→16`
- `CNN (1D): Conv1D 64→32 → Dense 64`
- `LSTM: LSTM 64 → LSTM 32 → Dense 32 (+ dropout)`
- `GRU: GRU 64 → GRU 32 → Dense 16`
- `SimpleRNN: 64 → 32 → Dense 16`
- `LSTM+CNN (hybrid): Conv1D 64→32 → LSTM 64→32 → Dense 32`

Common protocol: 80/20 split, `StandardScaler` on train only, early stopping on validation MSE (no test leakage). Reports R²/MSE/MAE on the hold-out set. Batch-size sweep used for split evaluation; fixed `batch_size=32` for CV.

---

## How to reproduce

1) **Environment**

conda:
```bash
conda create -n adcp-ml python=3.12 -y
conda activate adcp-ml
```
or venv:
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
```

2) **Install dependencies**

```bash
pip install -U pip
pip install -r requirements.txt
```

3) **Run notebooks (suggested order)**

- `LAB_Preprocessing_v2.ipynb`, `FIELD_Preprocessing_v2.ipynb`
- `LAB_ML_v2.ipynb`, `FIELD_ML_v2.ipynb`
- `LAB_DL_v2.ipynb`, `FIELD_DL_v2.ipynb`
- `Model_analysis_revised.ipynb` (consolidated results)

> Preprocessing notebooks require the raw `.mat` and `.rsqmb` files. ML/DL notebooks require only the processed Excel outputs.

---

## Expected outputs

- 5-fold stratified CV tables (mean ± std R²/MSE) for all models
- 80/20 split R²/MSE/MAE for all models
- Filtering sensitivity: CV R² across BT_Vel_stage0–stage4
- BS_rel contribution: ΔR² with and without BS_rel
- SHAP global bar charts and beeswarm plots
- Permutation importance across 8 models
- ALE plots, local SHAP waterfall plots
- Split-conformal prediction intervals (90% coverage)
- Training/validation curves and batch-size sweep plots
