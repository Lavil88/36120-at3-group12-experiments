# 36120 AT3: Cryptocurrency Financial Intelligence Platform, experimentation repository

36120 Advanced Machine Learning Applications, Assessment 3, **Group 12**

| Student | Name | Student ID | Target | Model | Notebook |
|---|---|---|---|---|---|
| A | Chakshu Karwal | 26038226 | Asset Performance Index (API) | XGBoost regression | `notebooks/36120_26SP_AT3_group12_26038226_experiment.ipynb` |
| B | Prachi Pramod Dhimate | 26060133 | Market Risk Index (MRI) | LightGBM regression | `notebooks/36120_26SP_AT3_group_12_student_26060133_experiment.ipynb` |
| C | Lavil Saldanha | 26036727 | Trend Category (TC) | XGBoost classification | `notebooks/36120_26SP_AT3_group12_26036727_experiment.ipynb` |
| D | Vedang Gosavi | 26059271 | Trading Opportunity Rating (TOR) | LightGBM classification | `notebooks/36120_26SP_AT3_group12_26059271_experiment.ipynb` |

Related repositories: Streamlit front-end and four FastAPI services (one per student), listed in the
final report together with their deployment URLs.

## Project structure (Cookiecutter Data Science)

at3_crypto/ shared Python package used by the notebooks and the APIs
config.py project paths
dataset.py Kraken OHLCV download, dated snapshots (python -m at3_crypto.dataset)
features.py every engineered feature from the brief + TOR engineered features
target.py TOR target (Opportunity Score, 5 classes, t+7)
asset_performance.py Student A target (Asset Performance Index, t+7) and projected index (API_proj)
data/raw/ dated Kraken snapshots (the snapshot used for the results is committed)
data/processed/ train/validation/test splits written by the notebooks (not committed)
models/ final model artefacts, one folder per student and experiment
notebooks/ one experiment notebook per student
reports/figures/ figures saved by the notebooks
requirements.txt pinned dependencies (versions required by the brief)
requirements-lock.txt full environment used to produce the results


## Setup

Requires **Python 3.12** (developed with 3.12.3) and git.

```bash
git clone <this repository URL>
cd 36120-at3-group12-experiments
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1      macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt        # also installs at3_crypto in editable mode
```

On Windows, if activation is blocked: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## Data

Data is collected dynamically from the public Kraken OHLC API (daily candles, no API key required):
`https://api.kraken.com/0/public/OHLC?pair=XBTUSD&interval=1440`.
Kraken only serves the latest 720 days, so each download is saved as a dated snapshot in `data/raw/`.
The notebooks load the latest snapshot; set `REFRESH_DATA = True` in section C.1, or run

```bash
python -m at3_crypto.dataset
```

to download a new one. The shared Kraken snapshot (kraken_ohlc_daily_20261022.csv) is committed for reproducibility of experiments that use it. Student B's experiment uses the assignment-provided dataset, as described below.


### Student B — MRI Experiment Dataset

The experiment uses the historical cryptocurrency dataset provided with the AT3 assignment (CoinMarketCap data) for exploratory data analysis, feature engineering, target construction, training, validation and testing.

The model uses 13 engineered features covering returns, momentum, moving averages, volatility, drawdown, gap risk and trading volume. A LightGBM regressor was optimised using Optuna with 50 trials. Model performance was evaluated using RMSE, MAE and R².

The deployed Student B FastAPI service retrieves live cryptocurrency data from the public Kraken API for inference and integrates engineered features from the other students' FastAPI services.

#### Reproducing Student B's Experiment

1. Clone the repository and install the dependencies listed in `requirements.txt` using Python 3.12.
2. Download the assignment-provided dataset ZIP file, `36120-26SP-AT3-dataset.zip`, from the course materials.
3. Place the ZIP file in the location expected by Student B's notebook. Check the notebook's dataset-loading cell for the exact relative path.
4. Open `notebooks/36120_26SP_AT3_group_12_student_26060133_experiment.ipynb` in JupyterLab or VS Code.
5. Select the project's `.venv` Python kernel and run all notebook cells sequentially.

The experiment generates the trained model and metadata files in `models/`, along with four visualisations in `reports/figures/`.

**Expected final test-set performance:**

| Metric | Value |
|---|---|
| RMSE | 9.4564 |
| MAE | 7.5613 |
| R² | 0.5399 |

The assignment dataset ZIP file is not committed to the repository and must be obtained separately from the course materials.

## Running the notebooks

```bash
jupyter lab notebooks
```

Select the `.venv` kernel and run all cells from the top. Student D's notebook takes about 5 to 10
minutes in total (Optuna runs 60 trials). All models use `n_jobs=1` for reproducibility on Windows.

Student A's notebook takes about 3 minutes (Optuna runs 60 trials in about 2 minutes). Section C.1
also calls the live Kraken API as a check; set `CHECK_LIVE_API = False` there to run fully offline.
Its train, validation and test files are written to `data/processed/26038226/`.

Student C's notebook takes about 7 minutes (Optuna runs 60 trials, each scored on 3 walk-forward folds).
It trains on the shared snapshot `kraken_ohlc_daily_20261022.csv` and computes its features in the notebook
with the same formulas as `at3_crypto/features.py`. It uses all CPU cores (`n_jobs=-1`); repeated runs on
the same machine gave identical results. Its train, validation and test files are written to
`data/processed/26036727/`.

## Model artefacts

| Student | Folder | Contents |
|---|---|---|
| A | `models/26038226/experiment_1/` | `api_xgb.joblib` (XGBoost model), `api_metadata.json` (feature order, importance, outage scenarios, hyperparameters, test metrics) |
| B | `models/` | `student_b_mri_lightgbm.joblib` (final LightGBM regression model) |
| B | `models/` | `student_b_mri_metadata.json` (model features, parameters and evaluation metrics) |
| C | `models/26036727/experiment_1/` | `tc_xgb.joblib` (XGBoost classifier), `tc_metadata.json` (features, importance, labels, hyperparameters, test metrics) |
| D | `models/26059271/experiment_1/` | `tor_lgbm.joblib` (LightGBM model), `tor_metadata.json` (features, importance, labels, hyperparameters, test metrics) |
| D | `models/tor_label_map.json` | fixed class id to label mapping |


## Collaboration workflow

Each student works on their own branch (`student-a` ... `student-d`). The best experiment is merged
into `main` through a pull request approved by another student; `main` is protected against direct
pushes.
