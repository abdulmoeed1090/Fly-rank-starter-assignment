# Refresh / Content Opportunity Scoring

## Abstract
Digital publishers struggle to allocate limited editorial resources toward updating vast libraries of published content. This paper presents a machine learning case study addressing FlyRank's content opportunity scoring challenge: prioritizing web pages facing imminent search traffic decay. Using a 50,000-row working slice from the FlyRank warehouse (spanning 2025-01-27 to 2025-02-27), we evaluate a rule-based baseline against an advanced Tuned Multi-Lag Gradient Boosted Decision Trees (GBDT) model. By incorporating backward-looking temporal velocity features—including 1-day/2-day impression deltas, position momentum, and 3-day rolling averages—the GBDT model significantly outperforms the baseline under a strict chronological 80/20 split and temporal leakage audit. The model increased ROC-AUC from 0.4622 to 0.6893, Average Precision from 0.2855 to 0.4756 (+66.5% relative uplift over the test base rate), and Precision@20 from 0.2500 to 0.3500, yielding an effective decision-support engine accompanied by transparent reason codes and editorial guardrails.

## 1. Introduction / Problem Statement
Content teams cannot review every page with equal attention. When search engine ranking algorithms shift or search intent evolves, older pages naturally undergo traffic decay. Manually auditing every page on a daily basis is cost-prohibitive. Content operations require an automated, data-driven mechanism to surface pages where editorial intervention will yield the highest return on investment.

This project focuses on building a repeatable ranking approach that identifies pages showing signals that justify further investigation.

> **Research Question:** Can observable search-performance signals be used to prioritize pages for human review to mitigate future performance decline?

The system is designed purely as **decision-support**. It does not automatically decide that a page needs a refresh, and it does not claim to explain Google's ranking algorithm. Rather than predicting an unweighted binary outcome, pages are assigned a continuous probability score to populate a high-density top-$K$ review queue.

## 2. Data
The analysis uses the FlyRank ML Internship warehouse, specifically the `fact_content_daily_performance` dataset (drawn from a 79M-row production release).

- **Working Dataset:** 50,000 rows
- **Start Date:** 2025-01-27
- **End Date:** 2025-02-27

The primary current-day signals and backward-looking temporal lag features are:
- **Current-Day Signals:** `gsc_impressions`, `gsc_clicks`, `gsc_avg_position`, `ctr`
- **Backward-Looking Lags:** `impression_delta`, `impression_ratio`, `position_delta`, `impression_delta_2d`, `impression_vs_3d_avg`

Identifiers such as client and content hashes are not used as predictive features. Report dates are used exclusively for temporal ordering and splitting. Future impression values are restricted to target construction and strictly excluded from model inputs.

## 3. Methodology

### 3.1 Future Outcome Proxy
Because the dataset does not contain a direct human-labelled "needs refresh" label, the project defines an observed future-performance proxy.

For pages with at least 5 current impressions and a valid next-calendar-day observation, a positive target (`1`) is assigned when the next observed day's impressions fall by at least 20% compared to current-day impressions. This target serves as a proxy for measurable short-term decay, not absolute proof that a content rewrite is required.

### 3.2 Week-4 Baseline
The transparent heuristic baseline combines three current-day search signals:

`Baseline Score = 0.4 × gsc_impressions + 40 × (1 − CTR) + 0.2 × gsc_avg_position`

Higher scores indicate higher priority for review. The baseline also assigns reason codes such as `LOW_CTR`, `LOW_POSITION`, and `MONITOR`.

### 3.3 Tuned Multi-Lag GBDT Model
A `HistGradientBoostingClassifier` (regularized GBDT) was trained using current-day metrics alongside multi-period trend velocity signals (1-day/2-day impression deltas, position shifts, and 3-day rolling averages) to capture momentum changes.

### 3.4 Validation Design & Leakage Audit
Because the outcome represents a future event, an honest chronological 80/20 time-aware split was used:
- **Total Usable Rows:** 27,422 (rows with valid next-day observations)
- **Training Rows (Earlier Dates):** 21,937
- **Testing Rows (Later Dates):** 5,485
- **Test Set Base Rate:** 31.17%

**Leakage Checks:** The model strictly excludes future dates, future impressions, target labels, baseline heuristic outputs (scores, reason codes, actions), and entity/client identifiers from its inputs.

## 4. Results
The performance of the models was evaluated on the held-out test set using ROC-AUC, Average Precision (AP), and Precision@20 (queue density for top 20 items):

| Method | ROC-AUC | Average Precision | Precision@20 | Evaluation Status |
| :--- | :---: | :---: | :---: | :---: |
| **Week-4 Rule Baseline** | 0.4622 | 0.2855 | 0.2500 | Transparent Baseline |
| **Initial Random Forest (Static)** | 0.5107 | 0.3137 | 0.2500 | Near-Random |
| **Tuned Multi-Lag GBDT (Enhanced)** | **0.6893** | **0.4756** | **0.3500** | **SUPERIOR** |

The Tuned Multi-Lag GBDT model achieved a **+66.5% relative uplift in Average Precision** over the rule baseline and raised Precision@20 from 25% to 35%, surfacing a significantly higher density of true decline candidates for human editors.

## 5. Limitations & Honest Framing
- The target variable is an observed future impression-decline proxy, not a direct human editorial label.
- The working analysis uses a 50,000-row slice of the full 79M-row warehouse.
- Search performance fluctuations are influenced by external factors (e.g., SERP layout changes, Google core updates, competitor shifts) not captured in GSC metrics alone.
- The model identifies statistical decay risk; it does not establish that refreshing a page will cause traffic recovery.

The findings are therefore best described as **observed, measured, directional, and decision-support** rather than causal or definitive.

## 6. Ranked Recommendations & Action Playbook
1. **Prioritize pages with Position Slippage (`POS_SLIP`):** Investigate queries where average rank degraded by > 2 positions relative to historical baselines.
2. **Audit High-Velocity Impression Drops (`IMP_DROP_VELOCITY`):** Focus on pages whose current impressions fall > 25% below their 3-day rolling average.
3. **Address High-Impression, Low-CTR Anomalies (`LOW_CTR_HIGH_IMP`):** Review title tags, snippet descriptions, and intent alignment for high-exposure pages with CTR < 1%.
4. **Enforce Human Review Guardrails & No-Go Rules:** Editors must manually verify SERP intent, technical health, and recent refresh timestamps (< 30 days) prior to making updates. Never automate content rewrites or URL redirects based solely on model probabilities.

## 7. Reproducibility
The project notebooks and pipeline artifacts are maintained in the repository under:
- `work/notebooks/capstone.ipynb`
- `work/notebooks/w03_feature_leakage_check.ipynb`
- `work/notebooks/w04_baseline_score.ipynb`
- `work/notebooks/w05_model.ipynb`
- `work/notebooks/w06_signal_audit.ipynb`
- `work/outputs/ml08_metrics.json`
- `work/outputs/ml09_validation_receipt.json`
- `work/outputs/ml10_action_queue_top50.csv`
- `work/outputs/ml10_playbook_receipt.json`

## 8. Acknowledgments & Data Credit
Built on the FlyRank ML Internship dataset.

Data credit: [FlyRank](https://flyrank.ai)
