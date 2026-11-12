# PBC2 Clinical Trial - Survival Analysis and Prognostic Factors - R (in progress)

A survival analysis of a randomized clinical trial with repeated patient visits. The aim is to test whether the treatment improved survival, and to find which patient characteristics predict the risk of death. Every choice (data cleaning, missing data, variable selection) is explained and justified.

> **Status:** the baseline analysis is done; the analysis of repeated measurements over time is coming next.

## Overview

- **Dataset:** `pbc2` from the R package JMbayes2 — Mayo Clinic randomized trial in primary biliary cirrhosis (PBC), D-penicillamine vs placebo. 312 patients, 1,945 visits (1 to 16 per patient, median 5), 20 variables (lab values, clinical signs, treatment, outcome)
- **Task:** compare survival between the two arms, then identify prognostic factors for death
- **Focus:** careful data checks, understanding why data are missing, and a Cox model with no more variables than the number of deaths allows

## Results at a glance

| Question | Method | Result |
|---|---|---|
| Did D-penicillamine improve survival? | Kaplan–Meier, log-rank test | No, the curves overlap (p = 0.99) |
| Any treatment effect after adjusting for risk factors? | Cox regression | No (HR 0.84, 95% CI 0.59–1.19, p = 0.33) |
| What increases the risk of death? | Cox regression | Older age (HR 1.04 per year), higher bilirubin (HR 2.48 per log unit), histologic stage 4 vs 1–2 (HR 1.85) |
| What lowers the risk of death? | Cox regression | Higher albumin (HR 0.52 per g/dL) |
| How well does the model rank patients by risk? | Concordance index | 0.84 (good discrimination) |

## Approach

1. **Data audit:** checked dimensions, data types, duplicates (0), empty rows (0) and missing values; counted patients and visits per patient.\
   → *Longitudinal data handling (one row per visit)*
2. **Distributions:** histograms and summary tables at baseline and across all visits. Skewed lab values (bilirubin, alkaline phosphatase, SGOT) were log-transformed for modelling.\
   → *Baseline extraction, tidyverse pipelines, flextable tables*
3. **Missing data:** checked whether missing values depend on treatment arm, visit number, outcome or disease stage. Cholesterol (42% missing over follow-up) was excluded. Missing alkaline phosphatase and platelet values are linked to disease severity: not MCAR, consistent with MAR (MNAR cannot be excluded).\
   → *Missing-data mechanisms (MCAR / MAR / MNAR), missingness by visit*
4. **Survival comparison:** Kaplan–Meier curves by arm with a log-rank test. Death is the event; patients alive or transplanted at the end of follow-up are censored.\
   → *Kaplan–Meier curves, log-rank test, censoring, comparing treatment arms*
5. **Variable selection:** 140 deaths for 18 candidate coefficients = 7.8 events per coefficient, below the usual minimum of 10 (Peduzzi et al., 1995). Reduced to 8 coefficients (17.5 events each), with no patient dropped, using literature published before this trial, correlation checks (edema dropped, redundant with ascites) and merging of small categories (histologic stages 1 and 2).\
   → *Events-per-variable rule, Spearman and Cramér's V, selecting prognostic factors from the literature*
6. **Cox regression:** main model on baseline data (treatment, age, bilirubin, albumin, histologic stage, hepatomegaly, ascites), compared with an extended model adding 4 lab values (likelihood-ratio test and AIC, same 308 patients).\
   → *Cox regression, hazard ratios, forest plots, nested model comparison*

## Coming next

- Interpretation of the main vs extended Cox model comparison
- Use of the repeated measurements over follow-up, not only baseline values

## Tech stack

R · tidyverse · survival · survminer · flextable · corrplot · DescTools · Quarto

## Files

- `5. PBC2_clin-trial_longitudinal-survival-IN-PREPARATION.md` — rendered analysis and interpretation, viewable on GitHub (in progress)
- `5.-PBC2_clin-trial_longitudinal-survival-IN-PREPARATION_files/` — graphs used by the .md file
