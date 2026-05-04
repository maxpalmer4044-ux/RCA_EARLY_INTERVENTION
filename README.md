# Manufacturing Defect Root Cause Analysis

End-to-end analytics project for detecting and explaining defect risk in a continuous manufacturing process using point-level PLC data.

This repository focuses on a practical root cause analysis workflow:

- rebuild a clean, point-level dataset from raw production and defect files
- create leakage-safe in-defect and pre-defect labels
- train interpretable models for early warning and diagnostic comparison
- translate model output into operational drivers, rules, and business-facing findings

## What This Project Does

The goal is not just classification.

The project is built to answer questions such as:

- Which process variables are associated with defects?
- Which signals appear before a defect region starts?
- What is the tradeoff between longer warning windows and stronger signal quality?
- Which operating regimes should a manufacturing team monitor first?

The pipeline separates data preparation from modeling so the workflow is easier to audit, explain, and reuse.

## Project Highlights

- Built a clean point-level dataset with corrected run keys using `COIL + DATE`
- Removed duplicate physical points deterministically
- Audited defect interval coverage before assigning labels
- Created early-warning labels for `50m`, `200m`, and `500m` upstream windows
- Used grouped train/test splits by manufacturing run to reduce leakage
- Prioritized interpretable models: logistic regression and random forest
- Produced driver tables, risk curves, interaction rules, and business-ready reporting

## Dataset Summary

Latest full clean build:

- `287,988` point-level observations
- `0` duplicate physical points after deduplication
- `102,304` in-defect rows
- `802` pre-defect 50m rows
- `2,713` pre-defect 200m rows
- `5,422` pre-defect 500m rows

Coverage audit summary:

- `134` fully covered defect intervals
- `480` partially covered defect intervals
- `3` defect intervals dropped due to zero production coverage

## Modeling Approach

The modeling notebook compares multiple targets:

- `any_defect`
- `pre_any_defect_200m`
- `pre_any_defect_50m`
- `defect_type3_any_defect`
- `defect_type3_pre_any_defect_50m`

Design choices:

- Use the clean point-level dataset only
- Exclude active defect rows for pre-defect modeling
- Split by run instead of random row split
- Favor interpretability over black-box performance chasing

Main model outputs include:

- coefficient tables from logistic regression
- permutation importance from random forest
- driver tables with risk direction and thresholds
- lift-based driver charts
- risk curves for top features
- grouped system-level importance
- concise markdown reporting

## Key Findings

From the final full-data RCA:

- In-defect detection is much easier than true early warning because the model can see the failure-state region itself
- The `50m` early-warning target is the strongest practical near-term signal by PR-AUC
- The `200m` target provides more lead time and higher top-5% lift, but with weaker sparse-label ranking quality
- The dominant recurring operating systems in the general 50m warning signal are `Combustion`, `Cooling`, and `Thermal`
- The most useful output is not a single score, but a combination of driver ranking, direction-of-risk, and system-level grouping

Important note:

This project is framed as observational root cause analysis. The outputs identify conditions associated with higher defect risk; they do not claim to prove physical causation on their own.

## Repository Structure

Core notebooks:

- `build_clean_point_dataset_submission_ready.ipynb`
- `rca_root_cause_analysis_submission_ready.ipynb`

Core scripts:

- `build_clean_point_dataset.py`
- `rca_root_cause_analysis_v2.py`

Key outputs:

- `rca_point_level_clean.csv`
- `rca_build_validation.json`
- `rca_defect_coverage_audit.csv`
- `rca_outputs/rca_root_cause_report.md`
- `rca_outputs/model_metrics.csv`
- `rca_outputs/comparison_table_presentation.csv`
- `rca_outputs/grouped_variable_importance.csv`

Presentation charts:

- `rca_outputs/dist_pre_any_defect_50m.png`
- `rca_outputs/corr_pre_any_defect_50m.png`
- `rca_outputs/risk_curves_pre_any_defect_50m.png`
- `rca_outputs/driver_lift_pre_any_defect_50m.png`
- `rca_outputs/risk_curves_any_defect.png`
- `rca_outputs/driver_lift_defect_type3_any_defect.png`

## How To Run

### 1. Build the clean dataset

Use:

- `build_clean_point_dataset_submission_ready.ipynb`

or:

```python
python build_clean_point_dataset.py --defects RC_DEFECTS_clean.csv --production RC_PRODUCTION_clean.csv
```

This generates:

- `rca_point_level_clean.csv`
- `rca_defect_coverage_audit.csv`
- `rca_build_validation.json`

### 2. Run the root cause analysis

Use:

- `rca_root_cause_analysis_submission_ready.ipynb`

or run the script-backed workflow from the notebook-generated code base.

This generates the report, comparison tables, driver tables, and charts in `rca_outputs/`.

## Tools Used

- Python
- pandas
- NumPy
- scikit-learn
- matplotlib
- seaborn

## Why This Project Is Different

Many defect-prediction projects stop at model accuracy.

This one is structured more like a production analytics workflow:

- validate the merge logic first
- enforce leakage-safe label creation
- evaluate early-warning targets separately from in-defect targets
- keep the models interpretable
- translate outputs into something operations teams can act on

## Contact

If you want to discuss the modeling approach, data engineering decisions, or manufacturing analytics workflow, feel free to reach out.
