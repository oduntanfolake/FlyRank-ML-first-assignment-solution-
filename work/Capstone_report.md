# Capstone Report

**Author:**  
**Lane:** Engagement Opportunity Scoring  
**Repo:**  
**Date:**  

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
