# Student Academic Success Prediction — OULAD

Predicting student outcomes (Distinction, Pass, Fail, Withdrawn) in online distance learning using the Open University Learning Analytics Dataset.

## Problem

Over 52% of students in the Open University's online modules either fail or withdraw. This project builds classification models to predict student outcomes using demographics, VLE engagement patterns, and assessment performance, enabling early intervention for at-risk students.

The key design question is: **at what point during the module can we reliably predict who will struggle?** We build datasets at two prediction points (day 50 and day 100) and compare models with and without assessment scores.

## Results

| Setup | Target | Best Model | F1 Score |
|---|---|---|---|
| Day 100, all features | Binary | XGBoost | **0.849** |
| Day 100, no scores | Binary | XGBoost | **0.814** |
| Day 100, all features | Multi-class | XGBoost | **0.646** |
| Day 50, no scores | Binary | XGBoost | **0.762** |

**Key finding:** VLE engagement alone (no grades) achieves **95.9%** of the full-feature binary F1 (0.814 vs 0.849) — a practical early-warning system is viable without any assessment data. The most important feature is `engagement_trend` (ratio of second-half to first-half clicks, importance: 0.109).

## Dataset

The [Open University Learning Analytics Dataset (OULAD)](https://analyse.kmi.open.ac.uk/open_dataset) — 32,593 student registrations, 7 modules, 10.6M VLE click logs.

## Project Structure

```
├── README.md
├── datasets/               # OULAD CSV files (download from source)
│   ├── courses.csv
│   ├── assessments.csv
│   ├── vle.csv
│   ├── studentInfo.csv
│   ├── studentRegistration.csv
│   ├── studentAssessment.csv
│   └── studentVle.csv
└── main.ipynb              # Full notebook (Phase 1 + Phase 2)
```

## Methods

### Models
1. **Decision Tree** — baseline (tuned: max_depth=8, criterion=gini)
2. **SVM (RBF)** — nonlinear classifier (tuned: C=100, gamma=0.001)
3. **XGBoost** — gradient boosting (tuned: 500 trees, depth 5, lr=0.05, regularised)

All models tuned via GridSearchCV / RandomizedSearchCV with GroupKFold (5 folds).

### Experiments

| Experiment | Day | Features | Purpose |
|---|---|---|---|
| A1 | 50 | All (62) | Early prediction |
| A2 | 100 | All (62) | Mid-module prediction |
| B1 | 50 | No scores (51) | Early-warning without grades |
| B2 | 100 | No scores (51) | Mid-module early-warning |

Each experiment evaluated on both multi-class (4 outcomes) and binary (Success vs At-Risk) targets.

### Techniques
- **SMOTE** oversampling for multi-class (Distinction is only 9.3%)
- **Feature selection** — dropped 2 zero-importance features (exam_score, has_sat_exam)
- **Weekly trajectory features** — click_slope, active_weeks, zero_weeks, last_week_ratio, click_weekly_std
- **Group-aware split** by student ID (12.3% appear in multiple modules)
- **Time-aware framework** — configurable prediction day filters VLE/assessment data

### Key Engineered Features

| Feature | Description |
|---|---|
| `engagement_trend` | Second-half / first-half clicks — #1 predictor (importance: 0.109) |
| `click_slope` | Linear regression slope of weekly clicks |
| `active_weeks` | Weeks with any VLE activity (Distinction median: 14, Withdrawn: 6) |
| `zero_weeks` | Weeks with zero clicks (gaps in engagement) |
| `submission_rate` | Fraction of expected assessments submitted |
| `weighted_coursework` | Weighted average of TMA+CMA scores (separate from exam) |

### Key Decisions

| Decision | Rationale |
|---|---|
| Separate coursework and exam scores | Independent grading pools (each sums to 100%) |
| Group-aware split by student ID | 12.3% of students appear in multiple modules |
| Exclude `date_unregistration` | Directly encodes withdrawal (leakage) |
| Two prediction days (50, 100) | Studies accuracy vs timing tradeoff |
| Two experiments (with/without scores) | Separates circular accuracy from genuine prediction |
| SMOTE for multi-class only | +0.29pp for multi-class, −0.11pp for binary |
| Drop exam_score, has_sat_exam | Zero importance at day 100 (exam not yet taken) |

## How to Run

### Requirements

```
pip install pandas numpy matplotlib seaborn scipy scikit-learn xgboost imbalanced-learn
```

### Setup

1. Download OULAD from [https://analyse.kmi.open.ac.uk/open_dataset](https://analyse.kmi.open.ac.uk/open_dataset)
2. Extract CSV files into `datasets/`
3. Run `main.ipynb` (Restart Kernel → Run All)

**Note:** VLE aggregation runs twice (~2 min each). Hyperparameter tuning takes ~3 hours total (can be skipped by hardcoding the tuned parameters).

## Authors

*(Add your names here)*

## License

This project uses the [OULAD dataset](https://analyse.kmi.open.ac.uk/open_dataset) released under CC-BY 4.0.
