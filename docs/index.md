# Content Refresh Prioritization Using Search Performance Signals

## Abstract

This study asks how search-performance signals can be used to prioritize pages for potential content-refresh review. Monthly Google Search Console performance data from the FlyRank internship warehouse was aggregated and paired across consecutive months to create an observed month-over-month impressions-decline label. A simple baseline using CTR and average position was compared with a HistGradientBoostingClassifier using impressions, clicks, CTR, and impression-weighted average position. On the held-out March–April 2026 test period, the ML model achieved Precision@20 of 0.850 compared with 0.800 for the baseline, while Average Precision increased from 0.585 to 0.622. The results provide directional decision-support evidence for creating a human review queue, but do not establish causal effects, guarantee future performance, or explain Google's ranking mechanisms.

## Introduction / Problem Statement

Content teams may have many pages that could potentially require review, while the available time for manual investigation is limited. This study focuses on the following decision:

**Which pages should be reviewed first for potential content-refresh opportunities based on observable search-performance signals?**

The objective is not to predict Google's ranking algorithm. Instead, the analysis uses historical search-performance observations to prioritize pages for human review.

A useful prioritization system should identify pages that resemble previously observed cases of month-over-month search-impression decline while avoiding information from the future evaluation period.

## Data

The analysis uses the **FlyRank internship warehouse release v20260703**, accessed through the FlyRank Hugging Face warehouse using DuckDB.

The analysis uses the `fact_content_daily_performance` table and aggregates Google Search Console performance to monthly page-level observations.

### Analysis period

- Feature months: October 2025 through April 2026
- Training feature months: October 2025 through February 2026
- Held-out test feature months: March 2026 and April 2026
- The following month's impressions are used only to construct the evaluation label.

### Features

Four search-performance features were used:

1. Monthly impressions
2. Monthly clicks
3. CTR
4. Impression-weighted average position

Rows without positive impressions were excluded from the modeling dataset.

No client names, domains, private queries, credentials, or raw identifying exports are included in this paper.

## Methodology

### Target definition

The target is an observed month-over-month decline:

`is_declining = 1` when the following month's impressions are lower than the feature month's impressions.

The future-month impressions and the resulting label were used only for evaluation. They were not provided to the model as input features.

### Validation design

A time-aware split was used to avoid training on future observations.

- **Training:** October 2025 – February 2026
- **Testing:** March 2026 – April 2026

The training set contained 490,524 observations with a decline rate of approximately 32.9%. The held-out test set contained 341,894 observations with a decline rate of approximately 58.7%.

The difference in decline rates between the two periods is an important limitation and is discussed below.

### Baseline

A simple baseline score was created from two signals:

- Low CTR relative to the training-period median
- Average position worse than the training-period median

The two components were combined with equal weights:

`baseline_score = 0.5 × low_CTR_score + 0.5 × position_score`

Pages were ranked by this score for comparison with the ML model.

### Machine learning model

A `HistGradientBoostingClassifier` was trained using:

- impressions
- clicks
- CTR
- average position

The model was configured with 150 boosting iterations, a learning rate of 0.08, a maximum of 15 leaf nodes, and a fixed random state of 42.

### Leakage check

The following fields were explicitly excluded from model features:

- `future_impressions`
- `is_declining`

The final feature list was:

`impressions, clicks, ctr, avg_position`

The leakage check found no forbidden future or label-derived fields in the model inputs.

## Results

The model and baseline were evaluated on the same held-out March–April 2026 test period.

| Metric | Baseline | ML Model |
|---|---:|---:|
| Precision@20 | 0.800 | 0.850 |
| Average Precision | 0.585 | 0.622 |
| ROC-AUC | — | 0.549 |

The ML model identified 17 of the top 20 ranked test observations as declining cases, giving a Precision@20 of 0.850. The baseline identified 16 of the top 20, giving a Precision@20 of 0.800.

Average Precision increased from 0.585 for the baseline to 0.622 for the ML model.

The ROC-AUC of 0.549 indicates that the model's overall ranking separation was modest. Therefore, the improvement should be interpreted as a limited prioritization improvement rather than evidence of a highly predictive system.

## Ranked Recommendation Playbook

The model output is intended to create a human review queue.

### Priority 1 — Review first

Pages with the highest model scores should be considered first for manual content review.

Reviewers can investigate factors such as:

- outdated information
- weak search-intent alignment
- incomplete topic coverage
- other content-quality issues

The model score itself is not evidence that a page requires a content update.

### Priority 2 — Validate the opportunity

High model scores should be considered together with sufficient search exposure and business relevance before significant refresh effort is allocated.

A human reviewer should validate the opportunity before making changes.

### Priority 3 — Monitor

Pages with lower model scores can remain in a monitoring queue and be reconsidered when new search-performance data becomes available.

## Limitations and Honest Framing

The analysis uses monthly aggregated search-performance signals and a simple month-over-month impressions-decline label.

The observed decline rate differs substantially between the training and test periods. Therefore, the evaluation period is not distributionally identical to the training period.

The model identifies pages that resemble observed declining cases based on the available features. It does not establish why a page declined or whether refreshing its content would cause recovery.

The analysis also does not explain Google's ranking mechanisms and should not be interpreted as a model of Google's search algorithm.

The ranked recommendations are intended for human decision-support and prioritization rather than automatic content decisions.

## Reproducibility

The analysis was performed using DuckDB over the FlyRank internship warehouse hosted on Hugging Face.

The workflow is:

1. Aggregate daily search-performance data into monthly page-level observations.
2. Pair each feature month with the following month.
3. Define the observed impressions-decline label.
4. Apply a time-aware training/test split.
5. Calculate the CTR/position baseline.
6. Train the HistGradientBoostingClassifier.
7. Rank the held-out observations by model score.
8. Compare Precision@20 and Average Precision.
9. Perform a leakage check to ensure future and label-derived fields are excluded.

The complete analysis notebook and generated public-safe outputs are available in this repository.

## Conclusion

This study demonstrates a practical approach for prioritizing content-refresh review using observable search-performance signals. The ML model produced a modest improvement over the simple CTR/position baseline on the held-out test period, with Precision@20 increasing from 0.800 to 0.850 and Average Precision increasing from 0.585 to 0.622.

The result supports using the model as a prioritization aid for human review. It should not be treated as evidence of causal content-refresh effects or as an explanation of search-engine ranking mechanisms.

## Acknowledgments and Data Credit

Built on the FlyRank ML Internship dataset.

This work was completed as part of the FlyRank Machine Learning Internship program.

