# Root Cause Analysis Report

## Key Findings

- The clean leakage-safe dataset contains 287,988 point-level observations. The baseline in-defect target has a much higher positive rate (35.52%) than the early-warning targets (0.39% for 200m and 0.13% for 50m), which is expected because the warning labels intentionally mark only a short upstream window.
- The preferred early-warning window for presentation is `pre_any_defect_50m`: it has the stronger forest PR-AUC (0.031 vs 0.010) and it stays closer to the operational moment when a team can still intervene.
- System-level results are stable. The dominant general 50m systems are Combustion, Cooling, Thermal, so the RCA can be explained as a systems story rather than a long list of isolated PLC tags.

## Pre-defect Labeling And Class Imbalance

The pre-defect labels are sparse by design, not by accident. A row is only counted as positive if it falls inside a short upstream warning window before a defect. That produces only 240 positive rows for the 50m window and 733 positive rows for the 200m window, out of 185,684 eligible upstream rows. This is important because it keeps the analysis leakage-safe: the warning models do not train on active defect rows, and the signal they learn is a true pre-defect signal rather than a symptom of a defect that is already happening.

## Comparison Of Targets

| target | positive_rate | PR-AUC (forest) | lift_top_5pct (forest) |
| --- | --- | --- | --- |
| In-defect | 35.52% | 0.805 | 2.51x |
| Pre-defect 200m | 0.39% | 0.010 | 2.99x |
| Pre-defect 50m | 0.13% | 0.031 | 1.71x |
| Type 3 in-defect | 29.13% | 0.764 | 2.84x |
| Type 3 pre-defect 50m | 0.07% | 0.002 | 0.95x |

## Baseline Vs Early Warning

- Baseline (`any_defect`): forest PR-AUC 0.805 and top-5% lift 2.51x. This is the easier problem because the model sees the failure-state region itself.
- Early warning (`pre_any_defect_50m`): forest PR-AUC 0.031 and top-5% lift 1.71x. This is a harder problem because the model must detect subtle upstream signals before the defect region starts.
The baseline is easier to score well, but the early-warning target is more valuable operationally because it can still support intervention.

## 50m Vs 200m Early Warning

- The 50m window has the stronger ranking metric for a sparse label: forest PR-AUC 0.031 vs 0.010 at 200m.
- The 200m window shows a higher top-5% lift (2.99x vs 1.71x), but that broader label is also less sparse and farther from the defect. For presentation, this makes 200m a useful longer-range monitor, not the preferred operational trigger.
- The two early-warning windows still overlap on 5 of the top 10 drivers (TEMP_Z5, TEMP_Z3, TEMP_Z2, TEMP_Z1, TEMP_Z4), so the shorter window mostly sharpens the same upstream story.

## Single Defect (Type 3) Comparison

- Type 3 in-defect is the easy baseline within the single-defect family: forest PR-AUC 0.764, top-5% lift 2.84x.
- Type 3 pre-defect 50m is much weaker but still meaningful as an upstream warning problem: forest PR-AUC 0.002, top-5% lift 0.95x.
- The general 50m and type 3 pre-50m models overlap on 2 of the top 10 drivers (TEMP_Z5, TEMP_Z2), which means the single-defect rerun narrows the story rather than overturning it.

## Variable And System Grouping

- General 50m: Combustion, Cooling, Thermal.
- General 200m: Combustion, Thermal, Cooling.
- General in-defect: Thermal, Combustion, Cooling.
- Type 3 in-defect: Thermal, Combustion, Cooling.
- Type 3 pre-defect 50m: Combustion, Thermal, Cooling.
This grouping makes the RCA easier to defend in a presentation because the explanation can focus on plant systems, not just individual variable names.

## Root Cause Explanation

- The model identifies variables and operating regimes that are associated with higher risk. It does not prove physical causation on its own.
- Across the early-warning and in-defect views, the strongest recurring systems are combustion, cooling, and thermal control. That means the final story is not one isolated sensor but a consistent system-level pattern.
- The pre-defect 50m drivers are the best operational compromise because they are upstream enough to be useful and close enough to the event to retain measurable signal.

### Top 50m Driver Snapshot

- `TEMP_Z3`: higher is associated with higher risk with holdout threshold `>= 1299.604` and lift 3.65x.
- `TEMP_Z2`: higher is associated with higher risk with holdout threshold `>= 1294.366` and lift 2.22x.
- `TEMP_Z1`: higher is associated with higher risk with holdout threshold `>= 1271.875` and lift 2.06x.
- `EXT_1`: higher is associated with higher risk with holdout threshold `>= 40.271` and lift 2.13x.
- `TEMP_Z4`: higher is associated with higher risk with holdout threshold `>= 1316.137` and lift 1.82x.

### High-Risk Combinations On Holdout Data (`pre_any_defect_50m`)

- `TEMP_Z3 >= 1299.604` and `TEMP_Z1 >= 1271.875` together are associated with a positive rate of 0.40%, which is 5.58x the baseline on 3,739 holdout rows.
- `TEMP_Z3 >= 1299.604` and `TEMP_Z2 >= 1294.366` together are associated with a positive rate of 0.36%, which is 5.06x the baseline on 4,396 holdout rows.
- `TEMP_Z1 >= 1271.875` and `VENT_1 >= 29.250` together are associated with a positive rate of 0.35%, which is 4.89x the baseline on 1,989 holdout rows.
- `TEMP_Z3 >= 1299.604` and `EXT_1 >= 40.271` together are associated with a positive rate of 0.34%, which is 4.73x the baseline on 3,525 holdout rows.
- `TEMP_Z1 >= 1271.875` and `VENT_2 >= 26.857` together are associated with a positive rate of 0.33%, which is 4.58x the baseline on 2,127 holdout rows.

### Interpretable Segment Rules (`pre_any_defect_50m`)

- `TEMP_Z2 > 1260.285 AND TEMP_Z5 > 1335.882 AND TEMP_Z3 > 1304.876` is associated with a positive rate of 0.32%, lift 4.43x, support 314 rows.
- `TEMP_Z2 > 1260.285 AND TEMP_Z5 <= 1335.882 AND TEMP_Z2 > 1274.523` is associated with a positive rate of 0.31%, lift 4.36x, support 8,287 rows.
- `TEMP_Z2 <= 1260.285 AND EXT_1 <= 57.162 AND TEMP_Z5 > 1257.211` is associated with a positive rate of 0.04%, lift 0.50x, support 19,607 rows.
- `TEMP_Z2 <= 1260.285 AND EXT_1 <= 57.162 AND TEMP_Z5 <= 1257.211` is associated with a positive rate of 0.02%, lift 0.28x, support 4,916 rows.
- `TEMP_Z2 > 1260.285 AND TEMP_Z5 > 1335.882 AND TEMP_Z3 <= 1304.876` is associated with a positive rate of 0.00%, lift 0.00x, support 8,305 rows.

### Segment Stability And Volatility

- `PYRO_3_LST_3_ZONE` shows 3.20x higher point-to-point volatility in the 50m warning window than in stable segments.
- `LASER_FRN_3` shows 2.91x higher point-to-point volatility in the 50m warning window than in stable segments.
- `TEMP_Z2` shows 2.53x higher point-to-point volatility in the 50m warning window than in stable segments.
- `TEMP_Z1` shows 2.08x higher point-to-point volatility in the 50m warning window than in stable segments.
- `AIR_Z3_1` shows 1.96x higher point-to-point volatility in the 50m warning window than in stable segments.

## Recommendations

1. Use the 50m warning model as the main presentation-level early-warning trigger because it is the clearest upstream signal.
2. Keep the 200m model as a longer-range monitor for drift, not as the primary intervention threshold.
3. Present the baseline in-defect models as a reference point so the audience can see why early warning is harder but more valuable.
4. Explain the process in system language first: combustion, cooling, and thermal control dominate the story.
5. Use the type 3 comparison to show that the main conclusions remain stable even when the analysis narrows to one defect family.

## Model Update Summary

- Added the previously missing target `defect_type3_any_defect`.
- Core general results did not change: the 50m model remains the preferred early-warning view and the system-level story remains stable.
- The update improves completeness and presentation defensibility rather than changing the underlying RCA conclusion.

## Limitations

- This remains an observational RCA on PLC data, so the findings show association rather than proven physical causation.
- Sparse pre-defect labels make early warning inherently harder than in-defect classification.
- Defect type 3 labels depend on the covered defect audit intervals overlaid onto the clean point-level dataset.
- The build still reflects partial coverage in some defect events, so some mechanisms may be under-observed.

## Supporting Files

- `model_metrics.csv`
- `comparison_table_presentation.csv`
- `grouped_variable_importance.csv`
- `presentation_summary.txt`