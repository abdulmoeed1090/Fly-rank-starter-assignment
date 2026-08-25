# Refresh / Content Opportunity Scoring

## Abstract

This project studies whether current search-performance signals can help prioritize pages for content review. A transparent Week-4 baseline score was compared with a Random Forest model using current-day impressions, clicks, average search position, and CTR. The evaluation used a time-aware split and a measurable next-day impression-decline proxy rather than a human-labelled refresh outcome. The Random Forest achieved higher ROC-AUC and Average Precision than the baseline, but the baseline performed better on the top-20 precision measure used for a small review queue. The result is therefore treated as directional decision-support rather than evidence that the model determines which pages should be refreshed.

## 1. Introduction / Problem Statement

Content teams cannot review every page with equal attention. The practical question is:

> Can observable search-performance signals be used to prioritize pages for human review?

This project focuses on building a repeatable ranking approach that identifies pages showing signals that may justify further investigation.

The system is designed as decision-support. It does not automatically decide that a page needs a refresh, and it does not claim to explain Google's ranking algorithm.

## 2. Data

The analysis uses the FlyRank ML Internship warehouse, specifically the `fact_content_daily_performance` data.

The working analysis uses a 50,000-row slice of the available data.

The observed reporting window in the working dataset is:

- Start: 2025-01-27
- End: 2025-02-26

The main current-day signals used in the modeling analysis are:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`
- `ctr`

CTR is calculated from current-day clicks and impressions.

Identifiers such as client and content hashes are not used as predictive features. Date is used for temporal ordering and validation rather than as a direct predictive feature.

Future impression values are used only to construct the evaluation outcome and are not supplied to the model as input.

## 3. Methodology

### 3.1 Future Outcome Proxy

The warehouse does not provide a direct human-labelled "needs refresh" outcome.

Therefore, the project uses an observed future-performance proxy.

For pages with at least 5 current impressions, a positive outcome is assigned when the next observed calendar day's impressions are at least 20% lower than the current day's impressions.

This produces an observed future-decline indicator.

The proxy should not be interpreted as proof that a page needs a content refresh.

### 3.2 Week-4 Baseline

The transparent baseline combines three current-day signals:

- Search impressions
- CTR
- Average search position

The baseline score is:

`0.4 × impressions + 40 × (1 − CTR) + 0.2 × average_position`

Higher scores produce a higher-priority review position.

The baseline also assigns reason codes such as:

- `LOW_CTR`
- `LOW_POSITION`
- `MONITOR`

This provides an interpretable rule-based benchmark.

### 3.3 Random Forest

A Random Forest classifier was trained using:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`
- `ctr`

The model was intentionally kept relatively constrained rather than maximizing complexity.

The model outputs a probability associated with the future-decline proxy. Pages can then be ranked according to this probability.

### 3.4 Validation Design

Because the outcome represents a future event, a time-aware split was used.

Earlier observations were used for training and later observations were used for testing.

The usable dataset contained 27,422 rows with measurable future outcomes.

The split was approximately:

- Training: 21,937 rows
- Testing: 5,485 rows

The test period was kept separate from the earlier training period.

The same test rows were used for the baseline and Random Forest comparison.

### 3.5 Leakage Checks

The model does not use:

- Future impressions as a feature
- The future-decline outcome as a feature
- Baseline score as a feature
- Baseline reason code as a feature
- Baseline action as a feature
- Client identifiers as predictive features
- Content identifiers as predictive features
- Report date as a predictive feature

This keeps the model inputs restricted to information available at the prediction point.

## 4. Results

The test-set future-decline base rate was approximately **30.59%**.

| Method | ROC-AUC | Average Precision | Precision@20 |
|---|---:|---:|---:|
| Week-4 Baseline | 0.4660 | 0.2865 | 0.3500 |
| Random Forest | 0.5107 | 0.3137 | 0.2500 |

The Random Forest produced higher ROC-AUC and Average Precision than the transparent baseline.

However, the Week-4 baseline produced higher Precision@20.

Because the practical use case is a small ranked review queue, this distinction matters. The more complex model did not outperform the simple baseline at the selected top-20 review point.

Therefore, complexity alone is not treated as an improvement.

## 5. Interpretation

The Random Forest learned patterns from the current search-performance signals, but its results should be interpreted as measured associations within this dataset.

Feature importance describes which inputs the model relied on when making its predictions. It does not establish that any feature causes future decline.

The difference between the baseline and model rankings also shows that the learned model does not simply reproduce the Week-4 rule.

The baseline remains useful because it is transparent, easy to inspect, and performed better on the selected top-20 precision measure.

## 6. Ranked Recommendations

The recommended workflow is:

### 1. Review low-CTR pages

Pages receiving meaningful search impressions but capturing relatively few clicks can be placed into a review queue.

Possible human review:

- title wording
- search-result relevance
- meta description
- alignment between search intent and page content

### 2. Review low-position pages

Pages with weaker average search positions can be investigated for content relevance and completeness.

Possible human review:

- content depth
- topical coverage
- search intent alignment
- internal linking
- content freshness

### 3. Monitor pages without strong warning signals

Pages without strong evidence from the selected signals should remain in monitoring rather than automatically receiving an intervention.

### 4. Prefer the transparent baseline for a small review queue

In this experiment, the baseline achieved higher Precision@20 than the Random Forest.

Therefore, for a small human review queue, the transparent baseline is the safer starting point.

The Random Forest remains useful as an experimental comparison and may become more useful with better labels, additional validated features, and a larger evaluation period.

## 7. Limitations & Honest Framing

This study has several limitations.

First, the target is a future impression-decline proxy rather than a human-labelled content-refresh decision.

Second, the working analysis uses a limited slice of the warehouse.

Third, next-day search impressions can change for many reasons that are not represented by the selected features.

Fourth, the model results should not be interpreted as evidence about Google's ranking algorithm.

Finally, the analysis does not demonstrate that refreshing a page causes improved search performance.

The findings are therefore best described as:

- observed
- measured
- directional
- decision-support

rather than causal or definitive.

## 8. Reproducibility

The project notebooks are maintained in the repository under:

`work/notebooks/`

The main modeling notebook is:

`work/notebooks/w05_model.ipynb`

The Week-4 baseline notebook is:

`work/notebooks/w04_baseline_score.ipynb`

The feature and leakage work is documented in:

`work/notebooks/w03_feature_leakage_check.ipynb`

The signal audit is documented in:

`work/notebooks/w06_signal_audit.ipynb`

The notebooks contain the analysis steps used to construct the features, baseline, future-outcome proxy, model, and evaluation.

## 9. Conclusion

The experiment shows that a Random Forest can provide measurable ranking signals for the defined future-decline proxy.

However, the Random Forest did not outperform the transparent Week-4 baseline on Precision@20, the metric most directly aligned with a small human review queue.

The practical conclusion is therefore not "the model wins."

Instead, the observed evidence supports keeping the transparent baseline as a strong decision-support starting point while treating the learned model as an experimental extension.

Future work should focus on better human-labelled outcomes, longer evaluation windows, additional validated signals, and stricter validation before relying on a learned model operationally.

## 10. Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset.

Data credit: [FlyRank](https://flyrank.ai)
