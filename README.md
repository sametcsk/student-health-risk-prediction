# Predicting Student Health Risk

An end-to-end tabular machine learning project developed for the Kaggle Playground Series S6E7 competition.

The task is to predict a student's health condition as one of three classes:

- `at-risk`
- `fit`
- `unhealthy`

The competition metric is **Balanced Accuracy**, making minority-class performance as important as majority-class performance.

## Result

| Metric | Result |
|---|---:|
| Private leaderboard score | **0.95043** |
| Final rank | **173** |
| Winning score | 0.95085 |
| Gap to winning score | 0.00042 |

The final leaderboard was extremely close. This project therefore focused not only on model performance, but also on validation reliability, class-level recall and public/private leaderboard risk.

## Dataset

| Split | Rows | Columns |
|---|---:|---:|
| Train | 690,088 | 15 |
| Test | 295,753 | 14 |

The dataset contains seven numerical and six categorical predictors related to sleep, heart rate, BMI, exercise, diet, stress and daily activity.

The target is imbalanced:

| Class | Train share |
|---|---:|
| `at-risk` | 85.87% |
| `unhealthy` | 8.36% |
| `fit` | 5.77% |

Because of this imbalance, ordinary accuracy would be misleading. Stratified cross-validation, per-class recall and Balanced Accuracy were used throughout the project.

## Project workflow

1. Inspect train, test and submission structure.
2. Check duplicate rows, duplicate IDs and missing values.
3. Review numerical ranges and IQR-based outlier candidates.
4. Compare train and test distributions.
5. Explore numerical and categorical relationships with the target.
6. Create a small set of interpretable interaction features.
7. Train CatBoost with `StratifiedKFold` and out-of-fold predictions.
8. Handle class imbalance with `SqrtBalanced` class weights.
9. Tune class probability multipliers against OOF Balanced Accuracy.
10. Validate the prediction distribution and create the submission.

## Main feature engineering

The clean notebook uses a limited set of interpretable interactions:

- Sleep duration × BMI
- Sleep duration × step count
- Exercise duration × step count
- Sleep per BMI
- Calories per exercise minute
- Steps per exercise minute
- Stress × sleep quality
- Stress × physical activity
- Sleep quality × physical activity

Each feature was treated as a hypothesis rather than assumed to be useful automatically.

## Experiments beyond the clean notebook

The broader competition study also included:

- CatBoost, LightGBM and XGBoost
- Target-encoded LightGBM
- Multi-seed averaging
- Probability prior correction
- Class-specific multiplier tuning
- Model blending and stacking
- RealMLP experiments
- Missing-value indicators
- Cascade and disagreement-arbiter experiments

The clean notebook intentionally presents the core workflow in a readable and reproducible form. The detailed experiment diary is available in [`docs/competition-retrospective-tr.md`](docs/competition-retrospective-tr.md).

## Repository structure

```text
student-health-risk-kaggle/
├── data/
│   └── README.md
├── docs/
│   └── competition-retrospective-tr.md
├── notebooks/
│   └── student_health_risk_clean_solution.ipynb
├── .gitignore
├── README.md
└── requirements.txt
```

## Getting the data

Competition data is not redistributed in this repository. Download it after accepting the Kaggle competition rules:

```bash
kaggle competitions download -c playground-series-s6e7
unzip playground-series-s6e7.zip -d data
```

The notebook automatically uses Kaggle's input directory when run on Kaggle. For local execution, place the following files inside `data/`:

```text
train.csv
test.csv
sample_submission.csv
```

## Installation

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
pip install -r requirements.txt
```

macOS/Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

Then open the notebook and run the cells from top to bottom.

## Key lessons

- Match the validation metric to the competition metric.
- Evaluate every class separately when the target is imbalanced.
- Keep out-of-fold predictions for honest model and blend comparisons.
- Treat missing values and outliers as hypotheses, not automatic deletion rules.
- Improve probability quality before tuning the final class decision.
- Prefer stable cross-validation gains over small public leaderboard gains.

## Türkçe kısa özet

Bu proje, öğrenci sağlık durumunu üç sınıftan biri olarak tahmin eden dengesiz bir sınıflandırma problemidir. Çalışmada temel veri kontrolleri, görsel EDA, yorumlanabilir feature engineering, StratifiedKFold, CatBoost, sınıf ağırlıkları ve OOF olasılık çarpanı optimizasyonu kullanılmıştır. Yarışma private leaderboard'da **0.95043 skor ve 173. sıra** ile tamamlanmıştır.

