# ADCP_BT_Vl_ML_DL

This repository contains the processed ADCP datasets preprocessing, and ML/DL notebooks to predict BT_Vel from signal/quality features
> **Quick access**
>
> - **Open in VS Code for the web:** https://vscode.dev/github/thtuhin488/ADCP_BT_Vl_ML_DL  
> - **Open on GitHub.dev:** https://github.dev/thtuhin488/ADCP_BT_Vl_ML_DL  
>
> In VS Code for the web, open the **Outline** panel (left sidebar) to navigate headings and fold sections, similar to desktop VS Code.
## Contents

- **Data/**
  - `Processed_Field_Data_with_ABS.xlsx`
  - `Processed_Flow_Rate_NEW_BIG_Data_with_ABS.xlsx`

- **Notebooks/**
  - `LAB_Preprocessing-.ipynb`
  - `FIELD_Preprocessing.ipynb`
  - `LAB_ML.ipynb`
  - `FIELD_ML.ipynb`
  - `LAB_DL.ipynb`
  - `FIELD_DL.ipynb`
  - `Model analysis.ipynb`

- `README.md`
- `requirements.txt`

> If filenames differ, update the first cells in the notebooks where `file_path` is set.


## Data

### Input files (provided in Data/):

  - `Processed_Field_Data_with_ABS.xlsx`
  - `Processed_Flow_Rate_NEW_BIG_Data_with_ABS.xlsx`

### Features (where available):
Depth, Vel_StdDev, Correlation, Mean_Speed, SNR, Vel_Expected_StdDev, Bin_Distance

### Target: BT_Vel

Preprocessing notebooks coerce non-numeric to NaN and drop rows with missing values in selected features/target.

## Models
### Classical ML (in *_ML.ipynb)

All models are trained inside a StandardScaler pipeline with 5-fold quantile-stratified CV (KBinsDiscretizer(n_bins=10)) and report R²/MSE:


  - `Random Forest Regressor`
  - `Gradient Boosting Regressor`
 - ` XGBoost Regressor`
  - `LightGBM Regressor`
  - `CatBoost Regressor`
  - `Stacking Regressor`


### Deep Learning (in *_DL.ipynb, TensorFlow/Keras)

  - `ANN: Dense 64→32→16`
  - `CNN (1D): Conv1D 64→32 → Dense 64`
  - `LSTM: LSTM 64 → LSTM 32 → Dense 32 (+ dropout)`
  - `GRU: GRU 64 → GRU 32 → Dense 16`
  - `SimpleRNN: 64 → 32 → Dense 16`
  - `LSTM+CNN (hybrid): Conv1D 64→32 → LSTM 64→32 → Dense 32`
  
> Common protocol: 80/20 split, StandardScaler on train only, early stopping on validation MSE (no test leakage). Reports R²/MSE/MAE on the hold-out set. Batch-size sweeps and KerasTuner used where noted

## How to reproduce

1) Environment

 conda

  ```bash
  conda create -n adcp-ml python=3.12 -y
  conda activate adcp-ml
 ```
 or venv
  ```bash
  python -m venv .venv
  # Windows:
  .venv\Scripts\activate
  # macOS/Linux:
  source .venv/bin/activate
  ```

2) Install deps

```bash 
pip install -U pip
pip install -r requirements.txt
 ```

3) Run notebooks (suggested order)

  - `LAB_Preprocessing-.ipynb, FIELD_Preprocessing.ipynb`
  - `LAB_ML.ipynb, LAB_DL.ipynb`
  - `FIELD_ML.ipynb, FIELD_DL.ipynb`
  - `Model analysis.ipynb (optional summaries)`
## Expected outputs

  - `Per-model R² / MSE / MAE on hold-out tests (DL)`

  - `5-fold stratified CV tables (mean±std R²/MSE) for ML`

  - `Training/validation curves and batch-size sweep plots`
