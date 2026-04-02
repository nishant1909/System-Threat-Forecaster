# System Threat Forecaster

A Kaggle competition notebook for binary classification of system threat likelihood using tabular machine learning. The task is to predict a binary `target` variable from system hardware and security configuration features.

---

## Dataset

Source: `/kaggle/input/System-Threat-Forecaster/`

- `train.csv` — labeled training data with a `target` column (balanced classes)
- `test.csv` — unlabeled test data for competition submission

Features include a mix of categorical and numerical columns covering attributes such as antivirus configuration, display resolution, OS/AS dates, processor model, disk type, boot security settings, and more.

---

## Project Structure

The notebook is organized into the following stages:

1. **Baseline** — DummyClassifier using the most-frequent strategy, used as a sanity check.
2. **EDA** — Exploratory analysis including class distribution, missing value heatmaps, correlation heatmaps, histograms, boxplots vs target, and categorical frequency plots.
3. **Preprocessing** — Custom sklearn-compatible transformers and a full preprocessing pipeline.
4. **Model Training** — Multiple classifiers trained and evaluated on an 80/20 train-test split.
5. **Hyperparameter Tuning** — GridSearchCV and RandomizedSearchCV applied to selected models.
6. **Submission** — Final model predictions exported to `submission.csv`.

---

## Preprocessing Pipeline

The pipeline handles the following transformations:

- **DateFeatureTransformer** — Converts `DateAS` and `DateOS` columns to datetime and extracts year, month, day, day-of-week, and hour. Drops original date columns.
- **VersionFeatureTransformer** — Parses version strings (e.g. `1.2.3`) into separate major and minor version number features.
- **FeatureEngineer** — Derives new features via differences and ratios between existing numerical columns.
- **OneHotEncoderTransformer** — Encodes categorical columns with unknown-category handling.
- **Imputation** — SimpleImputer (mean strategy) for numerical columns.
- **Scaling** — StandardScaler applied after imputation.

Three columns were dropped prior to training based on EDA findings: `IsBetaUser`, `AutoSampleSubmissionEnabled`, and `IsFlightsDisabled`.

---

## Models Evaluated

| # | Model | Notes |
|---|-------|-------|
| 0 | DummyClassifier | Baseline |
| 1 | LogisticRegression | `solver=saga`, `max_iter=7000` |
| 2 | RandomForestClassifier | Default + RandomizedSearchCV attempted |
| 3 | KNeighborsClassifier | Default parameters |
| 4 | GradientBoostingClassifier | Tuned with RandomizedSearchCV |
| 5 | GaussianNB | GridSearchCV and RandomizedSearchCV attempted |
| 6 | SVC | Abandoned — too slow, ~0.5952 accuracy |
| 7 | MLPClassifier | Abandoned — too slow, ~0.5737 accuracy |
| 8 | LinearDiscriminantAnalysis | GridSearchCV best: `solver=svd`, ~0.5966 |
| 9 | QuadraticDiscriminantAnalysis | Default parameters |
| 10 | SGDClassifier | Hyperparameter tuning too slow |
| 11 | AdaBoostClassifier | GridSearchCV attempted |
| 12 | BaggingClassifier | Default parameters |
| 13 | ExtraTreesClassifier | Default parameters |
| 14 | XGBClassifier | Tuned with RandomizedSearchCV — used for final submission |
| 16 | LGBMClassifier | Tuned with RandomizedSearchCV, ~0.60995 accuracy |

---

## Best Model: XGBoost

XGBoost with tuned hyperparameters was used for the final competition submission.

Best parameters found via RandomizedSearchCV:

```python
{
    'colsample_bytree': 0.7380,
    'gamma': 0.1738,
    'learning_rate': 0.01954,
    'max_depth': 8,
    'min_child_weight': 1,
    'n_estimators': 744,
    'subsample': 0.7843
}
```

---

## Dependencies

```
pandas
numpy
scikit-learn
xgboost
lightgbm
matplotlib
seaborn
```

All libraries are available in the standard Kaggle Python environment.

---

## Usage

Run cells sequentially. Hyperparameter tuning cells are commented out — their best-found parameters are hardcoded in subsequent cells for reproducibility. To generate a submission, uncomment the relevant `submit_competition(model)` call at the end of the chosen model's section.

```python
# Example: submit with the XGBoost best-params model
submit_competition(xgb_model_with_best_params)
```

