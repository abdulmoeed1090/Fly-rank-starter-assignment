# FlyRank Capstone Report

## Title

# Search Performance Decline: A Decision-Support Study

## Abstract

This study asks whether current search-performance signals can help prioritize pages for possible future search-performance decline. A transparent Week-4 scoring rule was compared with a Random Forest classifier using current-day Google Search Console signals. The future outcome was defined as an observed next-day impression decline of at least 20% for pages with at least five current impressions. The Random Forest achieved higher ROC-AUC and Average Precision than the baseline, but the baseline achieved better Precision@20. Therefore, the transparent baseline remains the preferred decision-support method for a small human review queue, while the model results are treated as directional evidence rather than causal evidence.

## Introduction / Problem Statement

Content teams cannot manually investigate every page every day.

This project supports the decision:

> Which pages should a content team investigate first?

The system is designed as decision support rather than an automatic content-refresh decision.

## Data

The project uses the FlyRank ML Internship warehouse dataset.

The selected current-day signals are:

- GSC impressions
- GSC clicks
- GSC average position
- CTR derived from clicks and impressions

Identifiers, metadata fields, future values, and target-derived fields were excluded from the predictive feature set.

No client names, private queries, credentials, raw exports, or private URLs are included.

## Methodology

### Baseline

The Week-4 baseline combines:

- GSC impressions
- CTR
- GSC average position

into a transparent score and assigns reason codes.

### Future outcome proxy

A direct human-labelled "needs refresh" outcome is not available.

The experiment therefore uses an observed future-performance proxy.

A row is labelled as a future decline when the next observed daily impression count for the same page is at least 20% lower than the current count, provided the current count is at least five impressions.

This is not interpreted as proof that the page needs a refresh.

### Random Forest

A Random Forest classifier was used to model the future-decline proxy.

The model uses current-day signals only.

### Validation

A time-aware 80/20 split was used.

Earlier observations were used for training and later observations were used for testing.

The baseline and Random Forest were evaluated on the same test rows.

## Results

| Method | ROC-AUC | Average Precision | Precision@20 |
|---|---:|---:|---:|
| Week-4 Baseline | 0.4660 | 0.2865 | **0.3500** |
| Random Forest | **0.5107** | **0.3137** | 0.2500 |

Test future-decline rate:

**30.59%**

The Random Forest improved ROC-AUC and Average Precision, but the transparent baseline performed better on Precision@20.

Because the intended workflow is a small ranked review queue, the baseline is preferred for the current use case.

## Limitations & Honest Framing

The target represents next-day impression decline rather than a human-labelled refresh decision.

An impression decline does not prove that content quality caused the change or that a page should be refreshed.

The analysis uses a limited working slice of the warehouse.

Search performance may also be influenced by factors not represented in the selected features.

The findings are therefore:

- observed
- measured
- directional
- decision-support

They should not be presented as causal evidence.

## Ranked Recommendations

### 1. Keep the transparent baseline

The baseline achieved 35% Precision@20 compared with 25% for the Random Forest.

### 2. Review low-CTR pages with meaningful visibility

Pages with search impressions but relatively few clicks can be prioritized for investigation of titles and metadata.

### 3. Investigate weak search positions

Pages with weaker observed positions can be considered for further content investigation.

### 4. Use the Random Forest as an exploratory benchmark

The model provides somewhat stronger broad ranking metrics but does not outperform the baseline for the small top-20 queue.

### 5. Keep human review in the loop

The system should prioritize investigation rather than automatically decide which content should be changed.

## Reproducibility

The repository contains:

- weekly assignment notebooks
- the capstone notebook
- the deployed research paper source
- the paper URL submission file

The main capstone notebook is:

`work/notebooks/w08_capstone.ipynb`

## Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset.

Data source: FlyRank.
