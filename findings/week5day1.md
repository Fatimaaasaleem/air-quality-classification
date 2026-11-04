# Internship Work Record — 4 November 2026

## Project

**Machine Learning-Based Air Quality Classification System**

## Objective

Investigate missing-value patterns in the labelled air-quality dataset and compare preprocessing strategies for AQI classification. Additionally, examine how training-set size affects model performance.

## 1. Missing-Value Analysis

The analysis was performed on 578,795 labelled observations using ten pollutant features.

### Overall Missingness

| Pollutant | Missing Count | Missing (%) |
| --------- | ------------: | ----------: |
| PM10      |       178,863 |       30.90 |
| NH3       |       163,994 |       28.33 |
| Benzene   |        96,365 |       16.65 |
| NOx       |        51,906 |        8.97 |
| O3        |        33,372 |        5.77 |
| SO2       |        33,058 |        5.71 |
| PM2.5     |        31,035 |        5.36 |
| CO        |        29,146 |        5.04 |
| NO2       |        22,267 |        3.85 |
| NO        |        21,481 |        3.71 |

**Finding:** PM10 and NH3 had the highest missingness among the selected pollutants.

### Missingness Across AQI Categories

Missingness varied across AQI categories.

Examples:

* PM10 was missing in 60.29% of Severe observations.
* NH3 was missing in 68.36% of Severe observations.
* PM10 was missing in 52.38% of Very Poor observations.
* NH3 was missing in 49.84% of Very Poor observations.

**Finding:** Missingness was not uniformly distributed across AQI categories, making it important to investigate its potential effect on classification.

### Missingness Across Years

| Year | PM10 Missing (%) | NH3 Missing (%) |
| ---- | ---------------: | --------------: |
| 2015 |            72.33 |           44.72 |
| 2016 |            57.55 |           51.29 |
| 2017 |            43.37 |           42.31 |
| 2018 |            31.06 |           27.08 |
| 2019 |            18.75 |           21.10 |
| 2020 |             8.86 |           11.58 |

**Finding:** PM10 missingness declined substantially from 72.33% in 2015 to 8.86% in 2020. NH3 also showed a broad decline, although its missingness peaked in 2016.

### Training vs Testing Missingness

| Pollutant | Train Missing (%) | Test Missing (%) |
| --------- | ----------------: | ---------------: |
| PM2.5     |              5.67 |             4.80 |
| PM10      |             41.25 |            12.20 |
| NO        |              3.56 |             3.99 |
| NO2       |              3.81 |             3.92 |
| NOx       |             11.04 |             5.22 |
| NH3       |             35.58 |            15.23 |
| CO        |              6.01 |             3.27 |
| SO2       |              6.31 |             4.64 |
| O3        |              5.76 |             5.77 |
| Benzene   |             17.37 |            15.34 |

**Finding:** The missingness distribution differed between training and testing periods, particularly for PM10 and NH3. This is an important consideration when evaluating model generalization.

## 2. Complete-Case Filtering vs Median Imputation

Two preprocessing approaches were compared:

* **Complete-case filtering:** Observations containing missing values were removed.
* **Median imputation:** Missing values were replaced using pollutant medians calculated from the training data.

### Available Observations

| Dataset               | Complete-Case | Median-Imputed |
| --------------------- | ------------: | -------------: |
| Training observations |       144,992 |        372,595 |
| Testing observations  |       120,334 |        206,200 |

For a fair performance comparison, both models were evaluated on the same 120,334 test observations.

### Model Performance

| Metric      | Complete-Case | Median-Imputed |
| ----------- | ------------: | -------------: |
| Accuracy    |        0.6035 |         0.6179 |
| Macro F1    |        0.4588 |         0.4648 |
| Weighted F1 |        0.5569 |         0.5708 |

**Finding:** Median imputation performed better across all three metrics and retained approximately 2.57 times as many training observations.

However, the models were trained on different numbers of observations. Therefore, this comparison evaluates the overall preprocessing pipelines and does not isolate the effect of imputation itself.

## 3. Class-Wise Performance Analysis

Both models were evaluated on the same test observations.

| AQI Category | Complete-Case F1 | Median-Imputed F1 |
| ------------ | ---------------: | ----------------: |
| Good         |             0.00 |              0.00 |
| Satisfactory |             0.70 |              0.71 |
| Moderate     |             0.63 |              0.65 |
| Poor         |             0.36 |              0.39 |
| Very Poor    |             0.48 |              0.47 |
| Severe       |             0.59 |              0.56 |

Key observations:

* Neither model successfully identified the Good category.
* Median imputation improved Satisfactory, Moderate and Poor classification.
* Complete-case filtering performed slightly better for Very Poor and Severe.
* Both models struggled to distinguish neighbouring AQI severity categories.

### Confusion Matrix Findings

The confusion matrices revealed several important patterns:

* Every Good observation was classified as Satisfactory.
* The median-imputed model correctly classified 2,742 Poor observations, compared with 2,397 for the complete-case model.
* Poor-to-Moderate errors decreased from 3,799 to 3,546 with median imputation.
* Very Poor and Severe classification remained slightly better with complete-case filtering.

**Finding:** Overall accuracy alone does not adequately represent model performance. Individual AQI categories must also be considered.

## 4. Additional Experiment: Reduced-Size Median-Imputed Training Set

An additional experiment was conducted to investigate the relationship between training-set size and model performance.

A third Decision Tree was trained using a random sample of 144,992 observations from the median-imputed training dataset.

All three models used:

* Decision Tree Classifier
* `max_depth=5`
* `random_state=42`
* The same 120,334 common test observations

### Results

| Model | Training Observations
