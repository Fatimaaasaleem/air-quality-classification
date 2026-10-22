# Week 4: Data Audit, Preprocessing and Baseline Experiments

**Project:** Machine Learning-Based Air Quality Classification System
**Project Type:** Internship
**Dataset:** Air Quality Data in India (`city_hour.csv`)
**Target:** `AQI_Bucket` — six AQI categories

## 1. Objectives

This session focused on analysing missing pollutant measurements, establishing a chronological train/test split, evaluating initial classification baselines, and investigating methods to improve class-specific performance.

The objective was to establish a reliable baseline before proceeding to more advanced model evaluation.

## 2. Missing-Value Audit

The dataset contains 707,875 hourly observations across 26 Indian cities. Of these, 578,795 have a recorded AQI category.

Missingness was examined across the ten selected pollutant features.

| Pollutant | Missingness |
| --------- | ----------: |
| PM10      |      30.90% |
| NH3       |      28.33% |
| Benzene   |      16.65% |
| NOx       |       8.97% |
| O3        |       5.77% |
| SO2       |       5.71% |
| PM2.5     |       5.36% |
| CO        |       5.04% |
| NO2       |       3.85% |
| NO        |       3.71% |

Overall, 313,469 labelled observations (54.16%) had at least one missing pollutant measurement, while 265,326 (45.84%) were complete.

Missingness also varied across AQI categories and years. For example, PM10 missingness increased from 16.93% in the Good category to 60.29% in the Severe category. NH3 missingness increased from 16.12% to 68.36% across the same categories.

**Decision:** Retain the ten selected pollutant features for initial experiments rather than immediately removing features with substantial missingness. Evaluate preprocessing alternatives to preserve useful information.

## 3. Chronological Train/Test Split

A chronological split was used to train on earlier observations and evaluate on later observations.

| Property     |          Training |           Testing |
| ------------ | ----------------: | ----------------: |
| Observations |           372,595 |           206,200 |
| Period       | Jan 2015–May 2019 | May 2019–Jul 2020 |

The target distribution differed between the periods. For example, Satisfactory observations increased from 27.43% of the training set to 42.31% of the test set.

Missingness also differed, particularly for PM10 (41.25% in training versus 12.20% in testing) and NH3 (35.58% versus 15.23%).

**Interpretation:** These differences indicate temporal variation in the observed data. Their causes have not been established.

**Limitation:** This is a temporal holdout, not an evaluation on entirely unseen cities.

## 4. Baseline Classification Experiments

A Decision Tree with `max_depth=5` and `random_state=42` was used as the initial classifier. A dummy classifier provided a simple reference baseline.

### Results

| Model                        | Test observations | Accuracy | Macro F1 | Weighted F1 |
| ---------------------------- | ----------------: | -------: | -------: | ----------: |
| Dummy classifier             |           120,334 |   30.76% |   0.0784 |      0.1447 |
| Complete-case Decision Tree  |           120,334 |   60.35% |   0.4588 |      0.5569 |
| Median-imputed Decision Tree |           206,200 |  ~59.98% |  ~0.4721 |     ~0.5591 |
| Class-weighted Decision Tree |           206,200 |   ~51.7% |    ~0.51 |       ~0.52 |

The complete-case model used 144,992 training observations because incomplete rows were discarded.

To evaluate imputation fairly, both the complete-case and imputed models were also compared on the same 120,334 test observations:

| Model                        | Accuracy | Macro F1 |
| ---------------------------- | -------: | -------: |
| Complete-case Decision Tree  |   60.35% |   0.4588 |
| Median-imputed Decision Tree |   61.79% |   0.4648 |

Median imputation retained all 372,595 training observations. The imputer was fitted on training data only and then applied to the test data to prevent leakage.

The imputed approach performed modestly better on the shared test observations. However, the models also used different training-set sizes, so the improvement cannot be attributed to imputation alone.

## 5. Class-Weighting Experiment

Both unweighted Decision Trees failed to predict the Good category, producing a Good-class recall of 0%.

To investigate this problem, a second imputed Decision Tree was trained using `class_weight="balanced"`. This automatically assigned greater training weight to underrepresented classes.

### Results

| Metric        | Unweighted imputed model | Class-weighted model |
| ------------- | -----------------------: | -------------------: |
| Accuracy      |                     ~60% |               ~51.7% |
| Macro F1      |                    ~0.47 |                ~0.51 |
| Good recall   |                       0% |                  91% |
| Severe recall |                      59% |                  76% |

The confusion matrix showed that the weighted model identified more Good and Severe observations but frequently misclassified Satisfactory observations as Good. Recall for Satisfactory and Moderate also decreased.

**Interpretation:** Class weighting improved macro F1 and recognition of underrepresented classes, but reduced overall accuracy and weighted F1. Neither model is clearly superior across all metrics.

## 6. Key Findings and Decisions

1. **Missing data:** More than half of labelled observations contain missing pollutant measurements. Complete-case deletion therefore removes substantial training data.
2. **Median imputation:** Retaining incomplete observations produced a modest improvement on the shared test subset and remains a candidate preprocessing strategy.
3. **Class imbalance:** The unweighted model's failure to predict Good motivated an experiment with balanced class weights.
4. **Performance trade-offs:** Class weighting improved Good and Severe recall but increased errors involving other categories.
5. **Evaluation:** Accuracy alone is insufficient. Macro F1, per-class precision and recall, and confusion matrices are needed to assess all six categories.

## 7. Limitations

* The Decision Tree depth has not been established as optimal.
* Median imputation is a baseline method, not necessarily the best solution.
* Class weighting introduces trade-offs between classes.
* The temporal split does not establish generalization to unseen cities or Pakistan-specific data.
* The supplied AQI categories have not been independently verified against every detail of an official AQI calculation procedure.
* No final model has been selected.

## 8. Next Steps

The next session will consolidate the comparison between the unweighted and class-weighted models, then select the next experiment based on their observed errors.

Further model comparisons and generalization evaluation will follow the project's staged plan. Forecasting and transferability remain later objectives, after a reliable classification baseline has been established.

**Session status:** Initial baseline experiments and confusion-matrix analysis completed. Further evaluation and model selection remain pending.
