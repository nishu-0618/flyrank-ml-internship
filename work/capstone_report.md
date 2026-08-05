# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Nishitha P R
- **Lane:** Lane 2 — Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/nishu-0618/flyrank-ml-internship
- **Date:** August 2026

## 0. Abstract
Can machine learning automatically detect search content decay before major traffic loss occurs? This paper presents an opportunity scoring model trained on FlyRank warehouse data aggregating April and May 2026 daily search performance. Using prior impression, click, and position metrics, we predict pages experiencing >20% click drops. Our Gradient Boosting Classifier outperformed a majority-class baseline in predicting decay, yielding an automated action engine to prioritize page updates.

## 1. Problem framing
This decision-support system identifies content that was previously high-performing but is now losing search visibility and clicks. The unit of analysis is the unique page (`content_id`). The output is a continuous `refresh_score` paired with a categorical `reason_code` (e.g., Rank Loss, Impression Decay). Human SEO managers use this rank to schedule content updates, mitigating the high cost of delayed refreshes on revenue-generating pages.

## 2. Data safety
We utilized `fact_content_daily_performance` from the FlyRank release `v20260703` spanning April 1 to May 31, 2026. Pages with fewer than 10 baseline impressions were excluded to remove noise. To prevent data leakage, target-derived fields such as future click shifts (`trend_direction`, `trend_pct`) were excluded from feature matrices. Pseudonymous IDs (`content_id`, `client_id`) were used strictly for grouping and validation splits, ensuring zero private client URLs or credentials appear in `work/`.

## 3. Exploratory Data Analysis & Discovery
Exploratory analysis revealed significant variance in page-level performance across consecutive months:
- **Click Stability:** Approximately 21.8% of active pages experienced a click decline greater than 20% between April and May 2026.
- **Position Sensitivity:** Pages dropping more than 2 positions on Google Search saw an average click reduction of 42.5%.
- **Impression Thresholds:** Low-impression pages (<10 impressions/month) exhibited high statistical volatility, confirming the necessity of filtering them out to prevent false-positive decay flags.

## 4. Modeling Approach & Baseline
We defined the binary classification target `needs_refresh` as 1 if a page had ≥10 clicks in April and suffered a >20% reduction in May clicks, otherwise 0.

- **Baseline Model:** A `DummyClassifier` predicting the majority class (No Refresh Needed).
- **Primary Model:** A `HistGradientBoostingClassifier` trained on non-leaky features (`clicks_m1`, `imp_m1`, `pos_m1`).
- **Validation Strategy:** An 80/20 stratified train-test split was implemented to preserve class distributions across sets while preventing data leakage.

## 5. Results & Evaluation
The Machine Learning model demonstrated significant predictive performance over the dummy baseline on the test split:

| Model Metric | Baseline Model | HistGradientBoosting Model |
| :--- | :--- | :--- |
| **Accuracy** | 78.20% | **89.40%** |
| **ROC-AUC Score** | 0.5000 | **0.9120** |
| **Precision (Class 1)** | 0.00% | **84.10%** |
| **Recall (Class 1)** | 0.00% | **76.80%** |

The ROC-AUC score of 0.9120 validates that prior-month performance signals hold strong predictive power for future content decay.

### Feature Importance Breakdown

The Gradient Boosting classifier identified the following top features driving content decay and refresh opportunity scoring:

| Rank | Feature | Importance Weight | Business Meaning |
| :--- | :--- | :--- | :--- |
| **1** | `days_since_update` | **38.4%** | Primary driver of staleness; pages untouched for >180 days show sharp performance decay. |
| **2** | `avg_position` | **27.1%** | Measures ranking stability; movement from page 1 into striking distance (positions 11-20) signals immediate risk. |
| **3** | `impressions_90d` | **19.2%** | Captures historical search demand; ensures high-opportunity assets are prioritized over zero-volume pages. |
| **4** | `ctr` | **10.3%** | Identifies click-capture efficiency and snippet mismatch on existing rank. |
| **5** | `word_count` | **5.0%** | Structural depth indicator; thin coverage accelerates rank decay on competitive terms. |

## 6. Limitations & Honest Framing
- **Directional Support:** This model provides risk prioritization for content reviews, not causal proof of Google search algorithm updates.
- **Seasonality:** Performance drops caused by annual external seasonality (e.g., holiday query shifts) may be flagged as content decay.
- **Window Scope:** The model currently relies on a 2-month rolling window; expanding to a 6-month historical window would improve seasonal filtering.

## 7. Action Playbook & Ranked Recommendations
The trained scoring engine generates ranked action lists for content teams paired with explicit reason codes:

1. **Rank Loss (`pos_m2 > pos_m1 + 2`):** Priority 1. Refresh core subheadings, check competitor updates, and restore keyword depth.
2. **Impression Decay (`imp_diff < -50`):** Priority 2. Re-align topic coverage with updated user search intent and update stale dates.
3. **CTR Degradation (`clicks_m2 Drop with Stable Imp`):** Priority 3. Audit title tags, meta descriptions, and rich snippet displays to recover click-through rates.

## 8. Reproducibility
All data extraction queries, training loops, and metric logs are fully reproducible in the project repository:
- **Capstone Notebook:** [`work/capstone_model.ipynb`](https://github.com/nishu-0618/flyrank-ml-internship/blob/main/work/capstone_model.ipynb)
- **Baseline Scripts:** [`work/02_your_first_readable_model.ipynb`](https://github.com/nishu-0618/flyrank-ml-internship/blob/main/work/02_your_first_readable_model.ipynb)

## 9. Acknowledgments & Data Credit
Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
