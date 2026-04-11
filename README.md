# Student Academic Success Prediction — OULAD (Phase 1)

Predicting student outcomes (Distinction, Pass, Fail, Withdrawn) in online distance learning using the Open University Learning Analytics Dataset.

## Problem

Over 52% of students in the Open University's online modules either fail or withdraw. Early identification of at-risk students would allow timely intervention. This project builds classification models to predict student outcomes using demographics, VLE engagement patterns, and assessment performance.

A key design decision is the **prediction point** — at what day into the module do we make the prediction? The notebook uses a configurable `PREDICTION_DAY` parameter that filters all time-dependent data (VLE clicks, assessment submissions) to only include information available up to that day, simulating a realistic early-warning deployment.

## Dataset

The [Open University Learning Analytics Dataset (OULAD)](https://analyse.kmi.open.ac.uk/open_dataset) is a relational dataset tracking 32,593 student-module registrations across 7 modules and 4 semesters (2013–2014).

| Table | Rows | Description |
|---|---|---|
| `courses` | 22 | Module metadata (length in days) |
| `assessments` | 206 | Assessment schedule, types, weights |
| `vle` | 6,364 | VLE material catalogue |
| `studentInfo` | 32,593 | Demographics + target (`final_result`) |
| `studentRegistration` | 32,593 | Registration and un-registration dates |
| `studentAssessment` | 173,912 | Individual score submissions |
| `studentVle` | 10,655,280 | Daily click logs per student per VLE page |

Place the CSV files inside a `datasets/` folder in the project root.

## Project Structure

```
├── README.md
├── datasets/               # OULAD CSV files (not included — download from source)
│   ├── courses.csv
│   ├── assessments.csv
│   ├── vle.csv
│   ├── studentInfo.csv
│   ├── studentRegistration.csv
│   ├── studentAssessment.csv
│   └── studentVle.csv
└── main.ipynb              # Phase 1: EDA, Preprocessing & Feature Engineering
```

## EDA Highlights

- **VLE engagement** is the strongest predictor: Distinction students have 9x more clicks and 7x more active days than Withdrawn students.
- **Assessment scores** clearly separate outcome groups but are partially circular — they determine the target rather than just predicting it.
- **Education level** is the strongest demographic predictor (28.1% Distinction for postgraduates vs 4.6% for no-qualifications).
- **IMD deprivation** shows a 3x difference in Distinction rate between least and most deprived areas.
- **Temporal trend**: withdrawal rates increased from 2013 to 2014, noted as a limitation.
- **12.3% of students** appear in multiple modules, requiring group-aware train-test splitting.
- **Un-registration** directly encodes withdrawal and was identified as a leakage variable.
- Statistical tests (Chi-square, Kruskal-Wallis) confirm all major findings.

## Preprocessing

- Missing `imd_band` (3.4%) imputed with mode; registration dates imputed with per-module median.
- 7 relational tables merged into a single student-level dataframe via left joins.
- Post-merge NaN values (students with no VLE/assessment activity) filled with 0.
- Coursework and exam scores computed as separate features (independent grading pools).
- `PREDICTION_DAY` parameter filters VLE clicks and assessment submissions to simulate early prediction.

## Engineered Features

| Category | Features | Available early? |
|---|---|---|
| Demographics | gender, age, education, IMD, disability, region, module, semester | ✓ Always |
| Registration | studied_credits, num_of_prev_attempts | ✓ Always |
| VLE engagement | total_clicks, active_days, distinct_pages, clicks_per_day, activity_diversity | ✓ Filtered to prediction day |
| Engagement trend | clicks_first_half, clicks_second_half, engagement_trend | ✓ Filtered to prediction day |
| Activity types | clicks per type (oucontent, forum, quiz, etc.), forum_ratio | ✓ Filtered to prediction day |
| Assessment | mean_score, std_score, weighted_coursework, exam_score, score_range | ⚠ Only if submitted before prediction day |
| Submission behaviour | submission_rate, num_submissions, pct_late, mean_days_early | ⚠ Only if submitted before prediction day |
| Flags | has_submitted_coursework, has_sat_exam, first_click_date | ⚠ / ✓ Depends on timing |

## Key Methodological Decisions

| Decision | Rationale |
|---|---|
| Separate coursework and exam scores | Independent grading pools (each sums to 100%) |
| Group-aware split by student ID | 12.3% of students appear in multiple modules |
| Exclude `date_unregistration` | Directly encodes withdrawal (leakage) |
| Configurable `PREDICTION_DAY` | Simulates realistic deployment with limited future data |
| Mode imputation for IMD band | Preserves ordinal encoding (vs "Unknown" category) |
| Zero-fill post-merge NaN | Absence of VLE/assessment activity is informative, not missing |

## How to Run

### Requirements

```
python >= 3.10
pandas
numpy
matplotlib
seaborn
scipy
```

### Setup

1. Download the OULAD dataset from [https://analyse.kmi.open.ac.uk/open_dataset](https://analyse.kmi.open.ac.uk/open_dataset)
2. Extract CSV files into a `datasets/` folder
3. Open `main.ipynb` and run all cells

### Prediction Day Configuration

At the top of the notebook, set the prediction cutoff:

```python
PREDICTION_DAY = None   # None = use all data (retrospective)
PREDICTION_DAY = 100    # Only use data from the first 100 days
PREDICTION_DAY = 30     # Very early prediction (limited data)
```

## Authors

*(Add your names here)*

## License

This project uses the [OULAD dataset](https://analyse.kmi.open.ac.uk/open_dataset) released under CC-BY 4.0.
