# Machine Learning-Based Air Quality Classification System

## Project Overview

This project investigates the use of machine learning to classify air quality into six categories using pollutant measurements. It is motivated by air pollution and seasonal smog in Pakistan, with the longer-term goal of exploring how a classification pipeline developed using available Indian air-quality data could be adapted to the Pakistani context.

**Project status:** In progress — dataset evaluation, data-quality analysis, preprocessing investigations, and preliminary classification experiments.

This is an **air-quality classification project, not a forecasting system**. No final model has been selected.

## Dataset

### Primary Dataset

The primary dataset is the Indian Air Quality dataset by Rohan Rao, obtained from Kaggle.

* **File:** `city_hour.csv`
* **Time period:** January 2015 – July 2020
* **Observations:** 707,875
* **Cities:** 26
* **Original columns:** 16

### Input Features

The current classification experiments use ten pollutant features:

* PM2.5
* PM10
* NO
* NO2
* NOx
* NH3
* CO
* SO2
* O3
* Benzene

The dataset also contains Toluene and Xylene, which may be considered in subsequent feature-selection and missing-data investigations.

### Target Variable

The target is `AQI_Bucket`, comprising six classes:

* Good
* Satisfactory
* Moderate
* Poor
* Very Poor
* Severe

The numerical `AQI` column is excluded from model inputs because it is directly associated with the construction of the target categories and could introduce target leakage.

After excluding observations with missing target labels, **578,795 labelled observations** remain.

### Pakistani Data

A smaller Pakistan/Punjab air-quality dataset, associated with `Saba-Gul/Forecasting-Air-Quality-Index`, has also been explored.

It appears to cover six stations across Lahore, Multan, and Gujranwala during approximately 2017–2020. Its suitability for supplementary analysis or external validation remains under investigation.

The current models have **not been validated on Pakistani air-quality data**.

## Tools and Technologies

* Python
* Pandas and NumPy
* Matplotlib and Seaborn
* Scikit-learn
* Jupyter Notebook
* Git and GitHub

## Work Completed

### 1. Dataset Exploration and Target Audit

* Examined dataset dimensions, temporal coverage, available cities, and target distribution.
* Investigated the relationship between `AQI` and `AQI_Bucket`.
* Excluded `AQI` from model inputs to reduce the risk of target leakage.
* Reviewed the original AQI calculation methodology to better understand how the target categories were constructed.

### 2. Data-Quality Investigation

An initial audit found no exact duplicate rows, duplicate city–timestamp combinations, or negative pollutant values.

However, several extreme pollutant readings require further investigation:

* **193** PM2.5 readings of 999.99.
* **41** PM10 readings of 999.99.
* **75** PM2.5 readings of 995.00, all recorded in Hyderabad.

These values have not been automatically removed. A defensible cleaning policy must distinguish potentially invalid measurements from genuine extreme pollution events.

### 3. Missing-Data Analysis

Missingness was examined across pollutant features, AQI categories, years, and the chronological training and test sets.

Initial missingness across the original dataset included:

| Feature | Missing values |
| ------- | -------------: |
| Xylene  |         64.39% |
| PM10    |         41.92% |
| NH3     |         38.50% |
| Toluene |         31.16% |
| Benzene |         23.12% |
| PM2.5   |         20.50% |

Further analysis of the 578,795 labelled observations found that **313,469 observations (54.16%)** had at least one missing value among the ten current pollutant features. The remaining **265,326 observations (45.84%)** were complete across those features.

Missingness was also found to vary across AQI categories and years. For example, PM10 missingness ranged from 72.33% in 2015 to 8.86% in 2020 within the labelled observations.

These patterns matter because complete-case deletion can discard substantial amounts of data and may affect which observations are available for training.

### 4. Chronological Train/Test Split

A chronological split was introduced to evaluate performance on a later time period rather than randomly mixing earlier and later observations.

| Partition | Observations | Period                  |
| --------- | -----------: | ----------------------- |
| Training  |      372,595 | January 2015 – May 2019 |
| Testing   |      206,200 | May 2019 – July 2020    |

The cutoff was `2019-05-26 13:00:00`.

The target distribution and pollutant missingness differ between the two periods. Consequently, this evaluation provides a more realistic temporal test, although it does not establish generalization to unseen cities or countries.

## Preliminary Classification Experiments

### Initial Exploratory Results

Early Decision Tree experiments used a random 80/20 split and complete-case data.

| Features                      | Rows retained | Accuracy |
| ----------------------------- | ------------: | -------: |
| PM2.5                         |       547,760 |   55.24% |
| PM10                          |       399,932 |   58.21% |
| PM2.5 + PM10                  |       381,538 |   60.39% |
| PM2.5 + PM10 + NO + NO2 + NOx |       353,347 |   61.42% |
| Ten pollutants                |       265,326 |   64.51% |

These results are exploratory. Each experiment retained a different number of observations because of missing values, so the accuracy differences cannot be attributed to feature selection alone. They also use a different evaluation setup from the later chronological experiments.

### Chronological Baseline Experiments

A shallow Decision Tree with `max_depth=5` and `random_state=42` was evaluated using the chronological split.

A Dummy Classifier was included as a simple reference point.

| Model                             | Test data           | Accuracy | Macro F1 | Weighted F1 |
| --------------------------------- | ------------------- | -------: | -------: | ----------: |
| Dummy Classifier                  | Complete test cases |   30.76% |   0.0784 |      0.1447 |
| Decision Tree — complete cases    | Complete test cases |   60.35% |   0.4588 |      0.5569 |
| Decision Tree — median imputation | Full test set       |   59.98% |   0.4721 |      0.5591 |
| Decision Tree — class weighting   | Full test set       |     ~52% |    ~0.51 |       ~0.52 |

The Dummy Classifier establishes a basic reference, while the Decision Tree results show that pollutant measurements contain useful information for distinguishing AQI categories.

**Important:** The rows in this table do not all represent identical evaluation conditions. The complete-case model was trained on complete observations only, while the imputed model used the full training set. The full-test imputed and class-weighted results also cover more test observations than the complete-case model.

For a fairer comparison, both the complete-case and median-imputed models were evaluated on the same 120,334 complete test observations:

| Model                        | Accuracy | Macro F1 |
| ---------------------------- | -------: | -------: |
| Complete-case Decision Tree  |   60.35% |   0.4588 |
| Median-imputed Decision Tree |   61.79% |   0.4648 |

The imputed model performed modestly better on this common test subset. However, the two models also used different training sample sizes, so the difference cannot be attributed to imputation alone.

### Class-Weighted Decision Tree

Class weighting was investigated to address differences in class recognition.

On the full chronological test set, the weighted model achieved approximately:

* **Accuracy:** 52%
* **Macro F1:** 0.51
* **Good-class recall:** 91%
* **Severe-class recall:** 76%

Class weighting improved recognition of the Good and Severe categories compared with the unweighted imputed model. However, overall accuracy declined, and many Satisfactory observations were misclassified as Good.

This illustrates the trade-off between overall accuracy and performance across individual classes. Neither approach has been selected as the final model.

## Current Findings

The preliminary experiments suggest that:

1. Pollutant measurements can support classification of the six AQI categories.
2. Missing data substantially affects the number and composition of observations available for training.
3. Median imputation allows the model to use more observations than complete-case deletion.
4. Class weighting can improve minority-class recall but may increase errors in other classes.
5. The Good category remains difficult for the unweighted Decision Tree to predict.
6. The chronological test distribution differs from the training distribution, making temporal evaluation important.
7. Accuracy alone is insufficient for model selection; macro F1, weighted F1, per-class precision and recall, and confusion matrices are also needed.

These are preliminary findings, not evidence of a final or deployment-ready system.

## Limitations

* The primary dataset is from India rather than Pakistan.
* Performance on Pakistani data and unseen cities has not been established.
* Missing-value handling and extreme-value treatment remain under investigation.
* AQI category distributions differ across the chronological training and test periods.
* Preliminary experiments use a shallow Decision Tree and do not establish that it is the best-performing model.
* Class weighting improves some categories while reducing performance on others.
* The original dataset's AQI labels have not been independently verified in full against an authoritative AQI calculation standard.

## Repository Structure

```text
air-quality-classification/
├── notebooks/
├── src/
├── .gitignore
├── LICENSE
├── README.md
└── requirements.txt
```

## Next Steps

1. Consolidate baseline results and confusion-matrix analysis.
2. Investigate the class-specific errors and the failure to predict Good in the unweighted models.
3. Compare missing-data strategies under controlled evaluation conditions.
4. Establish a more reliable classification baseline before undertaking broader model comparisons.
5. Investigate appropriate robustness and generalization tests.
6. Assess the feasibility of supplementary evaluation using compatible Pakistani air-quality data.

The project will progress incrementally, with model decisions supported by measured results and documented limitations.
