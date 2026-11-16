# SWYNEX-Data-Preparation

Task 1 of the SWYNEX Technologies Data Science Internship: prepare a public dataset for data-science work.

**Dataset:** [IBM Telco Customer Churn](https://github.com/IBM/telco-customer-churn-on-icp4d/blob/master/data/Telco-Customer-Churn.csv) (7,043 customers, 21 columns).
**Business question it supports:** which customers are likely to cancel their service?

## Headline finding
`pandas` reports **zero** missing values in this file, but `TotalCharges` contains **11 blank strings** that force the column to text.
All 11 belong to customers with `tenure = 0` (not billed yet), so the correct value is `0`, not a mean or median.
The pipeline enforces this and raises an error if a blank ever appears for a customer with billing history.

## Results
| | Raw | Clean |
|---|---:|---:|
| Rows / columns | 7,043 / 21 | 7,043 / 21 |
| Hidden blanks in `TotalCharges` | 11 | 0 |
| Missing cells | 0 (hidden) | 0 |
| Validation checks passed | n/a | 9 / 9 |
| Memory | 7.15 MB | 0.66 MB |

## Key decisions
| Issue | Action | Why |
|---|---|---|
| Blank `TotalCharges` (11) | Cast to float, fill `0.0` | All have tenure 0; verified and enforced in code |
| "No internet/phone service" levels (9,838 cells) | Merged into "No" | Redundant with `internet_service` / `phone_service` |
| Yes/No text columns (12) | Encoded as int8 0/1 | Model-ready, compact |
| Categorical columns | `category` dtype; `contract` ordered | Correct semantics, less memory |
| Outliers | None found by IQR rule; nothing removed | |
| Column names | snake_case | Consistent, code-friendly |

Full detail: [`reports/data_quality_report.md`](reports/data_quality_report.md) and [`docs/data_dictionary.md`](docs/data_dictionary.md).

## Project structure
```
SWYNEX-Data-Preparation/
├── data/
│   ├── raw/telco_churn_raw.csv          # untouched source file
│   └── processed/telco_churn_clean.csv  # pipeline output
├── notebooks/01_data_preparation.ipynb  # exploration, evidence, visuals
├── src/
│   ├── data_prep.py                     # clean() + validate() + CLI
│   └── report.py                        # generates the data-quality report
├── tests/test_data_prep.py              # 9 unit tests
├── reports/                             # auto-generated report + figures
├── docs/data_dictionary.md
├── requirements.txt
└── pytest.ini
```

## How to run
```bash
pip install -r requirements.txt
python -m src.data_prep --download   # raw -> validated clean CSV
python -m src.report                 # regenerate quality report and figures
pytest                               # run the unit tests
jupyter notebook notebooks/01_data_preparation.ipynb
```

## Design choices
- **Logic lives in `src/`, not in the notebook**, so it is testable and reusable; the notebook imports it.
- **Every step is logged** (`AuditLog`), and the report is generated from that log, so documentation cannot drift from the code.
- **Fail loudly:** unexpected categories or blanks raise errors instead of being silently patched.
- **Raw data is never modified**; `clean()` works on a copy.

## Limitations
- Static public sample: no time dimension, so drift and leakage cannot be assessed.
- 13 rows have `total_charges` far from `tenure x monthly_charges`; left unchanged and flagged in the report.
- Churn is about 26.5% of customers; use stratified splits and PR-AUC/recall when modelling.

## Next steps
Feature engineering, a baseline model (logistic regression, gradient boosting), and threshold tuning against retention-campaign cost.
