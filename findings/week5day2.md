# Internship Work Record — November 6, 2026

**Project:** Machine Learning-Based Air Quality Classification System  
**Supervisor:** Dr. Raza Ali  
**Notebook:** `04_missingness_and_feature_retention.ipynb`

## 1. Objectives

Today's work focused on:

1. Testing whether missingness indicators improve AQI classification.
2. Comparing retention of 10, 11, and 12 pollutant features.
3. Updating internship progress documentation and presentation slides.

## 2. Experimental Setup

A new chronological train-validation-test split was established using unique timestamps so observations sharing the same timestamp remained in the same partition.

| Partition | Observations | Period |
|---|---:|---|
| Training | 242,225 | 2015-01-01 16:00 to 2018-07-09 07:00 |
| Validation | 130,370 | 2018-07-09 08:00 to 2019-05-26 12:00 |
| Test | 206,200 | 2019-05-26 13:00 to 2020-07-01 00:00 |

All six AQI categories were present in both training and validation partitions.

**Important:** The test period has appeared in earlier project experiments, so it should not be described as an untouched final holdout. Today's model comparisons used the validation partition.

The baseline used:
- Ten pollutant features: PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3, and Benzene.
- Median imputation fitted on training data only.
- `DecisionTreeClassifier(max_depth=5, random_state=42)`.
- Accuracy, macro F1, and weighted F1 for evaluation.

## 3. Missingness Indicator Experiment

Binary indicators were added to identify whether each pollutant value was missing. The imputed pollutant values and their corresponding indicators were supplied together to a second Decision Tree.

### Results

| Metric | Median-Imputation Baseline | With Missingness Indicators |
|---|---:|---:|
| Accuracy | 0.5930 | 0.5930 |
| Macro F1 | 0.4439 | 0.4439 |
| Weighted F1 | 0.5695 | 0.5695