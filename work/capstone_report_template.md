# Capstone Report

**Author:** Abdul Moeed

**Lane:** Refresh / Content Opportunity Scoring

**Repo:** https://github.com/abdulmoeed1090/Fly-rank-starter-assignment

**Date:** July 2026

---

# 1. Problem Framing

## Decision

This project supports the decision of identifying which content pages should be reviewed and refreshed first.

## Unit of Analysis

One row represents the daily search performance of one content page.

## Output

A ranked list (or score) indicating the priority of refreshing each content page.

## Human Action

Content editors can review the highest-ranked pages first instead of manually inspecting every page.

## Cost of a Wrong Recommendation

If a page that needs attention is ranked too low, valuable traffic opportunities may be missed.

If a healthy page is ranked too high, editorial time may be wasted refreshing content that does not need improvement.

## Why ML?

Search performance depends on many interacting factors such as impressions, clicks, position, engagement, and traffic sources.

These relationships are difficult to capture using simple rules, making machine learning a suitable decision-support approach.

---

# 2. Data Safety

## Dataset

FlyRank Search Intelligence Warehouse

Current table:

fact_content_daily_performance

## Planned Features

- GSC impressions
- GSC clicks
- Average position
- Pageviews
- Sessions
- Users
- Scroll events
- Organic sessions
- AI referral sessions

## Deliberately Excluded

- client_hash_id as a predictive feature (used only for grouping or validation)
- content_hash_id as a predictive feature
- Any client-identifying information
- Future label-derived columns
- trend_direction (future work)
- trend_pct (future work)

## Leakage Considerations

No future information is used during feature construction.

Label-derived variables will be excluded once labels are created.

No client-identifying information appears anywhere in the repository.

---

# 3. Baseline

Not yet completed.

A simple baseline model or rule-based scoring system will be developed before training machine learning models.

---

# 4. Model / Analysis

Not yet completed.

Current work has focused on:

- Research Question
- ML Task Framing
- Data Contract

The target label will be defined in later assignments using future observed outcomes.

---

# 5. Evaluation

Not yet completed.

Future work will include:

- Train/Test split
- Time-aware validation
- Comparison against baseline
- Error analysis

---

# 6. Interpretation

Not yet completed.

Future work will analyse:

- Feature importance
- Signal relationships
- Model behaviour
- Negative results

---

# 7. Recommendation

Not yet completed.

The final output will provide ranked content refresh recommendations together with explanation codes describing why each page received its score.

---

# 8. Reproducibility

Repository:

https://github.com/abdulmoeed1090/Fly-rank-starter-assignment

Current notebooks completed:

- w01_research_question.ipynb
- w02_ml_task_framing.ipynb
- w03_data_contract.ipynb

Future notebooks will include:

- Feature Leakage Check
- Signal Audit
- Baseline Model
- Machine Learning Model
- Validation
- Action Playbook

Random seeds and environment configuration will be documented once modelling begins.

---

# Claims Checklist

✔ Observed / measured language

✔ Decision-support framing

✔ No causal claims

✔ No client-identifying information

✔ Repository remains reproducible

Metrics and model comparisons will be added after baseline and model development.
