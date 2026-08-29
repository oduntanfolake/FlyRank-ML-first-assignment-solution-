# Capstone Report

**Author:**  Oduntan Afolake 
**Lane:** Engagement Opportunity Scoring  
**Repo:** https://github.com/oduntanfolake/FlyRank-ML-first-assignment-solution-.git
**Date:**  29/08/2026


## Abstract

This study asks which content pages a website manager should investigate first when they show potential engagement problems. Using the FlyRank ML Internship warehouse, the analysis focuses on March 2026 content-performance data and combines observed search and engagement signals at the client-content-page level. A Random Forest ranking model was evaluated using a client-grouped test split and compared with the Week-4 rule-based baseline using Precision@50. The Random Forest achieved an observed Precision@50 of 0.70 compared with 0.00 for the Week-4 baseline, meaning 35 of its top 50 ranked pages were identified as opportunities under the engagement-opportunity proxy. The resulting ranked queue is intended as directional decision support that helps website managers prioritize pages for human investigation, rather than automatically determining which pages should be changed.

Question: Which pages should we investigate?
Data: FlyRank warehouse + March 2026.
Method: Random Forest + client-grouped evaluation + Precision@50.
Result: 0.70 vs 0.00 → 35/50 under the proxy.
Purpose: Prioritization/decision support, not automatic decisions.

## 1. Problem framing

This capstone supports the decision of which content pages a website manager should investigate first when they show potential engagement problems.

The unit of analysis is a content page for a client, aggregated over the March 2026 development window. The output is a ranked engagement-opportunity score that prioritizes pages for investigation.

A website manager can use the ranking to focus limited review time on pages that appear most worthy of investigation. The human action is to investigate the page, diagnose the possible engagement issue, and then decide whether an improvement is appropriate. The model does not automatically decide what change should be made.

A wrong ranking can cause a manager to spend time investigating a page that does not need attention, while potentially delaying attention to a page that may be more useful to investigate.

Machine learning is useful here because multiple observed search and engagement signals may interact in ways that are difficult to capture with a simple fixed rule. The Random Forest therefore provides a learned ranking signal that can be compared against the transparent Week-4 baseline.

The analysis is intended as decision support: it identifies pages that may deserve investigation rather than proving that a page has a real engagement problem or that a particular content change will improve performance.


## 2. Data safety

This analysis uses the public-safe FlyRank ML Internship warehouse release hosted on Hugging Face. The development data comes from the `fact_content_daily_performance` table for March 2026.

The March 2026 slice contains 9,841,378 daily observations across 55 clients and 331,437 content pages, covering March 1–31, 2026. For modeling, these daily observations are aggregated to one row per client and content page.

The modeling signals used are `gsc_impressions`, `gsc_clicks`, average search position, `sessions_ai`, and `scroll_events`.

Client and content identifiers are pseudonymous hashes and are used only for grouping, evaluation, and traceability. They are not used as predictive features.

I deliberately excluded client names, domains, URLs, private queries, credentials, and raw data exports from the public-facing work. I also excluded data-availability flags as predictive signals because they describe whether data exists rather than the engagement opportunity itself.

Future-month performance was excluded from modeling because it could introduce information that would not be available at the time of ranking. Label-derived fields such as `trend_direction` and `trend_pct` were also not used as model features because they could leak information about the outcome being studied.

The analysis therefore uses only the March 2026 development window and observed signals available within that window.


## 3. Baseline

The first benchmark was the transparent Week-4 rule-based baseline. It prioritizes pages using a simple visibility-and-staleness rule rather than a learned model.

The Week-4 baseline was designed to identify pages that were both visible in search and potentially overdue for review. It used page visibility through GSC impressions together with the earlier refresh-opportunity framing.

This is a useful comparison because it provides a simple, interpretable ranking that can be understood without machine learning. The Random Forest therefore needs to demonstrate that a learned combination of signals provides a more useful ranking than the simpler rule.

For the final comparison, the baseline and Random Forest were evaluated on the same held-out client-grouped test set using the same Precision@50 metric.

On this test split, the Week-4 baseline achieved a Precision@50 of 0.00, while the Random Forest achieved 0.70 against the engagement-opportunity proxy.

The baseline is treated as a benchmark rather than ground truth. Its purpose is to establish how the learned model performs relative to a transparent rule.


## 4. Model / analysis

The selected method is a Random Forest used for ranking/scoring content pages by engagement opportunity.

Random Forest was chosen because the lane involves multiple observed search and engagement signals, and the relationships between those signals may be nonlinear or involve interactions. A learned model can therefore provide a more flexible ranking than a single hand-written rule.

The model uses the following five features:

- `gsc_impressions`
- `gsc_clicks`
- `avg_position`
- `sessions_ai`
- `scroll_events`

The target is an engagement-opportunity proxy based on observed engagement signals. In the evaluation, a page is treated as an opportunity when it has positive GSC impressions but zero GSC clicks. This proxy is used to evaluate prioritization and is not treated as an independently validated business outcome.

The model produces an opportunity score. Pages are ranked by this score so that a website manager can investigate the highest-ranked pages first.

Future-month performance is not used as a feature. Pseudonymous client and content identifiers are also excluded from the model because they are used for grouping and traceability rather than prediction.

The model is therefore intended to support prioritization, not to automatically determine what content change should be made.


## 5. Evaluation

The model was evaluated using a client-grouped train/test split. Clients were separated before model fitting so that content from the same client did not appear in both the training and evaluation sets. The March 2026 development data was split into 80% training and 20% testing, with 44 clients in training and 11 clients in the test set.

The primary evaluation metric is Precision@50 because the practical decision is to identify a small number of pages for investigation first. The Week-4 baseline and Random Forest were evaluated on the same held-out test set using the same metric.

The Week-4 baseline achieved a Precision@50 of 0.00, while the Random Forest achieved 0.70 against the engagement-opportunity proxy. This corresponds to 0 of the baseline's top 50 pages and 35 of the Random Forest's top 50 pages being identified as opportunities under the proxy.

The error analysis shows that the model still ranks some pages highly that are not marked as opportunities by the proxy. For example, some highly ranked pages have substantial impressions and non-zero clicks, so they are flagged for review rather than automatically treated as engagement opportunities.

This evaluation provides observed, directional evidence that the Random Forest produced a stronger top-50 ranking than the Week-4 baseline on this test split. It does not establish that the model will perform at the same level on future months or different client populations.


## 6. Interpretation

The Random Forest's feature importance indicates that `gsc_impressions` was the strongest signal used by the fitted model, with an observed importance of approximately 0.734. `gsc_clicks` followed at approximately 0.204, while `avg_position`, `scroll_events`, and `sessions_ai` had smaller observed importance values.

This suggests that the model's ranking is driven primarily by search visibility and click activity in the available March 2026 data. Pages with substantial search visibility but limited click activity can therefore receive high opportunity scores.

The ranked output also shows both successful and imperfect recommendations. Several of the highest-ranked pages have positive impressions and zero clicks and are marked as opportunities under the proxy. Other highly ranked pages have non-zero clicks and are therefore marked for review rather than immediate investigation as opportunities.

An important negative result is that the smaller feature-importance values do not mean those signals have no real-world value. They only describe their contribution within this fitted Random Forest and this dataset. Feature importance should therefore be interpreted as model-specific evidence, not causal evidence.

Overall, the model appears useful as a prioritization mechanism: it narrows a large set of content pages into a smaller ranked queue for human investigation.


## 7. Recommendation

The recommended use of the model is as a ranked investigation queue for website managers and editors.

A FlyRank editor could begin with the highest-ranked pages and investigate pages that have high model scores, strong search visibility, and low click capture. These pages should be reviewed for possible issues with search-result presentation, title and description relevance, search-intent alignment, or content usefulness.

The recommended workflow is:
**Rank → Investigate → Diagnose → Act → Monitor**

The model should determine the order in which pages are reviewed, not the final action taken on each page. A page ranked highly should therefore be investigated by a human before any content change is made.

The ranked recommendations can be grouped into two practical review paths:
- **Investigate engagement:** pages identified as opportunities under the current proxy, particularly pages with impressions but zero clicks.
- **Review before action:** highly ranked pages that do not satisfy the proxy and therefore require additional human diagnosis before any change is recommended.

The model's confidence should be treated as directional rather than absolute. The Precision@50 result of 0.70 was measured against the engagement-opportunity proxy on the March 2026 client-grouped test split. It should not be interpreted as proof that 70% of recommendations will represent genuine engagement problems in production.

The recommended action is therefore to use the model for prioritization and decision support, followed by human review and subsequent monitoring of outcomes.


## 8. Reproducibility

The analysis was developed in the capstone notebook using DuckDB for querying the FlyRank warehouse and scikit-learn for the Random Forest model.

The development window is March 2026. Daily content-performance observations were aggregated to one row per client and content page before modeling.

The modeling features were:
- gsc_impressions
- gsc_clicks
- avg_position
- sessions_ai
- scroll_events

The validation split used `GroupShuffleSplit` with `client_hash_id` as the grouping variable, a test size of 20%, and `random_state=42`. The Random Forest used `random_state=42`, 30 trees, `max_depth=10`, and parallel processing.

The analysis can be reproduced by running the capstone notebook from the beginning after providing the required Hugging Face read token through the notebook's credential mechanism. The notebook contains the data checks, modeling workflow, evaluation, interpretation, and ranked recommendation outputs.

The analysis does not require client names, domains, private queries, credentials, or raw data exports to reproduce the public-facing findings. Hashed identifiers are used only for grouping and traceability.

The final notebook and supporting assignment notebooks are committed under `work/` in the project repository. The deployed paper URL is recorded separately in `submission/paper_url.txt`.

The reported results in this paper correspond to the documented March 2026 development window and the client-grouped evaluation described above.


## 9. Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset.

Data source: [FlyRank](https://flyrank.ai)
