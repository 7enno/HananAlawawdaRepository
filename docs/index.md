# Decoding Search Visibility: A Ranking Signal Analysis of Click-Through Rates

## Abstract
This research investigates which historical and query-level signals most strongly drive search Click-Through Rate (CTR) beyond raw ranking position. Using the FlyRank ML Internship dataset, we engineered features including 30-day momentum, query concentration, and visibility metrics. We trained a Random Forest Regressor and validated it across distinct client sets using `GroupShuffleSplit` to prevent domain leakage. The model successfully outperformed naive baseline splits, revealing that historical click-through rate and anonymized query share are the most critical secondary drivers of clicks. These findings provide a decision-support heuristic, allowing content teams to prioritize specific optimizations for pages underperforming their expected CTR.

## Introduction / Problem Statement
Ranking highly in search results is only half the battle; capturing the click is the final conversion. Content teams often struggle to prioritize which pages to optimize. This analysis supports the decision of *where to allocate editorial resources* by identifying the specific signals that maximize CTR, allowing teams to intervene when a page's clicks are declining.

## Data
* **Source:** FlyRank ML Internship dataset (Release: internship-warehouse).
* **Tables Used:** `fact_content_daily_performance`, `fact_content_query_90d`.
* **Date Window:** We utilized a strict 60-day rolling historical window.
* **Exclusions:** We excluded any URL with fewer than 100 impressions to prevent low-volume mathematical noise from distorting percentage metrics. No private client names or raw queries are exposed.

## Methodology
* **Label Definition:** `target_ctr_last30` (clicks / impressions over the target 30-day window).
* **Features:** Engineered from the *previous* 30-day window (`pos_prev30`, `feature_ctr_prev30`) and 90-day query mix (`rare_share`, `anon_share`).
* **Validation Design:** We implemented a `GroupShuffleSplit` on `client_hash_id` (75/25).
* **Leakage Checks:** A naive random split yielded an impossibly low MAE of 0.00004 (confirming client-memorization leakage). Our grouped split eliminated this. Feature importances were audited to ensure no single feature dominated above the 85% threshold.

## Results
The model proved generalization across unseen clients when grouped honestly. 
* **Honest Grouped Split MAE:** 0.00207

![Feature Importance](image_8b9f8c.png)

The model leans heavily on historical momentum (`feature_ctr_prev30` at ~57%) and query-mix signals (`anon_share` and `rare_share` at ~13-14% each).

## Limitations & Honest Framing
This model provides **directional, decision-support guidance**, not causal certainty. We observed strong associations between query share and CTR, but altering a meta description does not force Google's algorithm to award more clicks. Predictions are strictly bounded by the 60-day historical window and cannot account for sudden future algorithmic shifts.

## Ranked Recommendations (Action Playbook)
Based on the feature interactions, editorial teams should prioritize:
1. **Meta Refresh:** Pages with strong ranking positions (e.g., top 5) but abnormally low historical CTR. These require immediate title or description A/B testing.
2. **Long-Tail Optimization:** Pages with a high `anon_share` (>40%). These pages capture hyper-specific, conversational user intent and should be structured to answer direct user questions.

*Note: Automated playbook filters explicitly exclude compliance pages (Legal, Privacy).*

## Reproducibility
All SQL extraction queries, feature engineering logic, and modeling code are available in the public repository.
* **GitHub Repository:** https://github.com/7enno/HananAlawawdaRepository
* **Execution:** Run the notebooks in the `work/notebooks/` directory.

## Acknowledgments
Built on the FlyRank ML Internship dataset. Data provided by [FlyRank](https://flyrank.ai).
