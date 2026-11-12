# Crime Incidents - Data Cleaning, Exploration and Statistical Analysis - Python

A Python workflow applied to a messy, simulated crime dataset: from raw data with realistic quality problems to an analysis-ready table, followed by exploratory analysis and statistical tests. Cleaning decisions are checked and explained, and imputed values are flagged rather than hidden.

## Overview

- **Dataset:** simulated crime incident dataset from [Kaggle](https://www.kaggle.com/datasets/sananshaikh/messy-crime-dataset-for-data-cleaning-practice), 5,250 records describing crimes, locations, victims, suspects, officers, arrests, property loss and case status
- **Task:** clean the raw data, then use it to answer three analysis questions
- **Focus:** transparent cleaning decisions, checks before and after each transformation, and statistical tests chosen to fit the data

## Cleaning at a glance

| Problem | Action | Result |
|---|---|---|
| Duplicate records | Removed | 5,250 → 5,050 rows |
| Typos, abbreviations and synonyms | Similarity detection (`difflib`) + manual review | e.g. crime type: 58 spellings → 4 categories; district: 16 → 10 |
| Malformed amounts (e.g. `35446.9.0`) | Repaired before numeric conversion | 136 values recovered |
| Three date formats, incl. US month-first | Each format converted explicitly | No swapped days and months |
| Impossible values (negative numbers, ages up to 298, latitudes > 90) | Sign corrected or set to missing | 331 ages and 176 coordinates set to missing |
| Missing victim/suspect gender | Inferred from first names (`gender-guesser`) | 1,964 values imputed and flagged |

## Results at a glance

| Question | Test | Result |
|---|---|---|
| Do incident counts differ between cities? | Chi-square goodness-of-fit | Yes (χ² = 15.6, p = 0.03); Riverside has the most incidents (705) |
| Does the crime-type mix differ between cities? | Chi-square test of independence | No (p = 0.26) |
| Did incidents change over time (2018–2024)? | Poisson regression on monthly counts | No significant trend (+1.1% per year, p = 0.12) |
| Do incidents vary by season? | Chi-square goodness-of-fit | No (p = 0.32) |
| Is crime severity linked to property loss? | Kruskal–Wallis, Spearman correlation | No (p = 0.79; ρ = 0.01, p = 0.39) |

## Approach

1. **Initial data audit:** inspected datatypes, missing values, duplicates, categorical levels and numeric ranges to identify the main data-quality problems.\
   → *Data quality assessment, duplicate detection*
2. **Text and datatype standardization:** trimmed whitespace, standardized capitalization, stored identifiers as text and converted wrongly stored numbers to numeric values.\
   → *String normalization, datatype conversion*
3. **Categorical cleaning:** detected spelling errors by string similarity, reviewed every suggested correction manually, harmonized abbreviations and synonyms, and grouped crime types into four broad categories.\
   → *Typo detection, synonym harmonization, manual quality control*
4. **Missing-data handling:** assessed missingness variable by variable instead of applying one imputation rule; imputed gender from first names with explicit imputation and uncertainty flags, and left other missing values as missing.\
   → *Selective imputation with explicit flags*
5. **Numeric and geographic validation:** checked distributions and plausible ranges, corrected sign errors and set impossible values to missing.\
   → *Range validation*
6. **Date and time preprocessing:** split date and time, standardized three date formats, and created time-of-day and season variables.\
   → *Mixed date formats, feature engineering*
7. **Exploration and analysis:** described the main variables (summary statistics, bar charts, histograms, box plots, trend plots), then answered three questions with statistical tests. Numeric variables are not normally distributed, so non-parametric tests were used where needed.\
   → *Chi-square tests, Poisson regression for count data, non-parametric tests (Kruskal–Wallis, Spearman), matplotlib*

## Tech stack

Python · pandas · NumPy · SciPy · statsmodels · matplotlib · difflib · gender-guesser

## Files

- `4. Crimes_cleaning_data.ipynb` — full analysis notebook

---

# 5. PBC2 Clinical Trial - Survival Analysis and Prognostic Factors - R (in progress)

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
