# Model Comparison and Optimization — Findings

**Project:** Machine Learning-Based Air Quality Classification System  
**Notebook:** `05_model_comparison_and_evaluation.ipynb`  
**Evaluation Stage:** Chronological Training and Validation

## 1. Project Objectives

The objective of this stage was to compare machine learning classifiers for predicting six AQI categories (Good, Satisfactory, Moderate, Poor, Very Poor, and Severe) using pollutant measurements.

The experiments focused on:

- Comparing different classification algorithms.
- Evaluating performance across all six AQI categories.
- Investigating classification errors and their severity.
- Assessing overfitting through training and validation performance.
- Optimizing Random Forest hyperparameters to improve macro F1 without excessive overfitting.

Macro F1 and per-class recall were prioritized alongside accuracy because reliable classification across all categories is important.

## 2. Dataset and Evaluation Setup

The experiments used ten pollutant features:

PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3, and Benzene.

Rows without an `AQI_Bucket` label were excluded. Missing pollutant measurements were replaced using median imputation, with the imputer fitted on the training data only.

A chronological split was used:

| Partition | Samples | Period |
|---|---:|---|
| Training | 242,225 | 2015-01-01 16:00 to 2018-07-09 07:00 |
| Validation | 130,370 | 2018-07-09 08:00 to 2019-05-26 12:00 |
| Test | 206,200 | 2019-05-26 13:00 to 2020-07-01 00:00 |

The split was based on unique timestamps to prevent observations from the same timestamp crossing partition boundaries.

**Evaluation limitation:** The test period had already been used in earlier project experiments. Therefore, it cannot be considered a completely untouched holdout set. The results documented here are validation results; no new test evaluation is reported.

## 3. Initial Model Comparison

Four classification algorithms were evaluated using the same training and validation partitions:

- Decision Tree
- Random Forest
- Extra Trees
- Gradient Boosting

Class weighting was used for Random Forest and Extra Trees. Gradient Boosting was trained using balanced sample weights.

### Table 1. Aggregate Validation Performance

| Model | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| Decision Tree | 0.5930 | 0.4439 | 0.5695 |
| Random Forest | 0.6036 | 0.5499 | 0.6064 |
| Extra Trees | 0.5524 | 0.5153 | 0.5618 |
| Gradient Boosting | 0.5896 | 0.5489 | 0.5952 |

Random Forest achieved the highest accuracy (60.36%) and weighted F1 (0.6064). Its macro F1 (0.5499) was marginally higher than Gradient Boosting (0.5489).

Extra Trees achieved a lower accuracy but performed better than the Decision Tree in terms of macro F1.

The Decision Tree failed to identify any Good observations in this validation run, resulting in zero recall for that category. This demonstrates why accuracy alone is insufficient for evaluating the classifiers.

### Table 2. Per-Class Validation Recall

| AQI Category | Decision Tree | Random Forest | Extra Trees | Gradient Boosting |
|---|---:|---:|---:|---:|
| Good | 0.0000 | 0.6181 | 0.8013 | 0.7878 |
| Satisfactory | 0.6986 | 0.6943 | 0.5552 | 0.6734 |
| Moderate | 0.7248 | 0.6257 | 0.5878 | 0.5863 |
| Poor | 0.1917 | 0.4198 | 0.3863 | 0.4399 |
| Very Poor | 0.5664 | 0.4842 | 0.4832 | 0.4477 |
| Severe | 0.4586 | 0.5969 | 0.6428 | 0.6412 |

Extra Trees achieved the highest Good recall (0.8013), while Gradient Boosting achieved the highest Poor recall (0.4399). Extra Trees and Gradient Boosting had similar Severe recall.

Poor remained a challenging category across all four models.

## 4. Training and Validation Performance

Training and validation performance were compared for Random Forest and Gradient Boosting to investigate overfitting.

### Table 3. Training and Validation Performance

| Model | Train Accuracy | Validation Accuracy | Train Macro F1 | Validation Macro F1 |
|---|---:|---:|---:|---:|
| Random Forest | 0.6911 | 0.6036 | 0.6736 | 0.5499 |
| Gradient Boosting | 0.5745 | 0.5896 | 0.5577 | 0.5489 |

Random Forest showed a training-validation macro F1 gap of 0.1237 and an accuracy gap of 0.0875, indicating a noticeable difference between training and validation performance.

Gradient Boosting had a smaller macro F1 gap of 0.0088. Its validation accuracy was 0.0151 higher than its training accuracy.

This suggests more consistent performance across these two chronological partitions, although it does not establish generalization to all future periods or other locations.

## 5. Confusion Matrix and Error Analysis

Confusion matrices were examined for Random Forest and Gradient Boosting to identify common classification errors.

The analysis showed substantial confusion between neighbouring AQI categories, particularly:

- Poor and Moderate
- Poor and Very Poor
- Very Poor and Severe
- Satisfactory and Moderate

For Random Forest, 6,928 of 16,502 actual Poor observations were correctly classified. A further 5,421 were predicted as Moderate and 3,078 as Very Poor.

For Gradient Boosting, 7,259 of the 16,502 actual Poor observations were correctly classified, demonstrating better performance on this category than Random Forest.

For Random Forest, 6,452 of 13,326 actual Very Poor observations were correctly classified. The main errors included 3,318 predictions as Poor and 1,788 as Severe.

### Table 4. Ordinal Error Severity

Errors were grouped according to the distance between the actual and predicted categories, ordered as:

Good → Satisfactory → Moderate → Poor → Very Poor → Severe.

| Error Measure | Random Forest | Gradient Boosting |
|---|---:|---:|
| Correct Predictions | 78,691 | 76,862 |
| One-Category Errors | 43,808 | 45,834 |
| Errors Two or More Categories Away | 7,871 | 7,674 |
| Total Errors | 51,679 | 53,508 |
| Two-or-More-Category Errors (% of Total Errors) | 15.23% | 14.34% |

Most classification errors were one category away from the correct class.

Random Forest produced fewer total errors, while Gradient Boosting had a slightly lower proportion of errors that were two or more categories away.

These results represent different aspects of error performance and should not be treated as interchangeable.

## 6. Random Forest Hyperparameter Optimization

The previous Random Forest configuration used:

- `n_estimators=200`
- `max_depth=15`
- `min_samples_leaf=5`
- `min_samples_split=20`
- `class_weight="balanced"`
- `random_state=42`
- `n_jobs=-1`

Controlled experiments were conducted by changing one hyperparameter at a time.

Validation macro F1 was used as the primary selection metric, with accuracy and weighted F1 retained as supporting measures.

### 6.1 Effect of Maximum Depth

All other parameters were held constant, with `min_samples_leaf=5` and `min_samples_split=20`.

**Table 5. Maximum Depth Comparison**

| Max Depth | Train Macro F1 | Validation Macro F1 | Validation Accuracy | Validation Weighted F1 |
|---:|---:|---:|---:|---:|
| 10 | 0.5807 | **0.5546** | 0.5946 | 0.5990 |
| 15 | 0.6736 | 0.5499 | 0.6036 | 0.6064 |
| 20 | 0.7486 | 0.5492 | 0.6072 | 0.6089 |

Depth 10 achieved the highest validation macro F1 and the smallest training-validation macro F1 gap (0.0261).

Increasing tree depth improved training macro F1 but did not improve validation macro F1, suggesting increased model complexity without a corresponding validation benefit.

### 6.2 Effect of Minimum Samples per Leaf

Maximum depth was fixed at 10 and minimum samples per split at 20.

**Table 6. Minimum Samples per Leaf Comparison**

| Min Samples Leaf | Train Macro F1 | Validation Macro F1 | Validation Accuracy | Validation Weighted F1 |
|---:|---:|---:|---:|---:|
| 3 | 0.5810 | 0.5544 | 0.5935 | 0.5982 |
| 5 | 0.5807 | **0.5546** | **0.5946** | **0.5990** |
| 10 | 0.5795 | **0.5546** | 0.5940 | 0.5984 |

The three configurations produced nearly identical validation results.

Leaf sizes 5 and 10 achieved the same macro F1, while leaf size 5 had slightly higher accuracy and weighted F1.

Increasing the leaf size did not produce a meaningful improvement.

### 6.3 Effect of Minimum Samples per Split

Maximum depth was fixed at 10 and minimum samples per leaf at 5.

**Table 7. Minimum Samples per Split Comparison**

| Min Samples Split | Train Macro F1 | Validation Macro F1 | Validation Accuracy | Validation Weighted F1 |
|---:|---:|---:|---:|---:|
| 10 | 0.5813 | **0.5556** | **0.5953** | **0.5998** |
| 20 | 0.5807 | 0.5546 | 0.5946 | 0.5990 |
| 40 | 0.5786 | 0.5536 | 0.5926 | 0.5971 |

A split threshold of 10 achieved the highest result among the tested configurations.

However, the improvement over a threshold of 20 was small, with macro F1 increasing by only 0.0010.

### 6.4 Effect of Maximum Features

The `sqrt` and `log2` settings were compared while all other parameters remained fixed.

**Table 8. Maximum Features Comparison**

| Max Features | Train Macro F1 | Validation Macro F1 | Validation Accuracy | Validation Weighted F1 |
|---|---:|---:|---:|---:|
| sqrt | 0.5813 | 0.5556 | 0.5953 | 0.5998 |
| log2 | 0.5813 | 0.5556 | 0.5953 | 0.5998 |

Both configurations produced identical results.

With ten input features, `sqrt` and `log2` select the same number of candidate features at each split. Consequently, this experiment did not distinguish between the two settings.

## 7. Final Comparison of Random Forest Candidates

The tuned Random Forest configuration used:

- `n_estimators=200`
- `max_depth=10`
- `min_samples_leaf=5`
- `min_samples_split=10`
- `max_features="sqrt"`
- `class_weight="balanced"`
- `random_state=42`
- `n_jobs=-1`

It was compared with the previous Random Forest and Gradient Boosting models using the same validation partition.

### Table 9. Final Candidate Aggregate Performance

| Model | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| Previous Random Forest | **0.6036** | 0.5499 | **0.6064** |
| Tuned Random Forest | 0.5953 | **0.5556** | 0.5998 |
| Gradient Boosting | 0.5896 | 0.5489 | 0.5952 |

The tuned Random Forest improved macro F1 by 0.0057 compared with the previous Random Forest, equivalent to 0.57 percentage points.

However, its accuracy decreased by 0.0083 (0.83 percentage points), while weighted F1 decreased by 0.0066 (0.66 percentage points).

### Table 10. Final Candidate Per-Class Recall

| AQI Category | Previous Random Forest | Tuned Random Forest | Gradient Boosting |
|---|---:|---:|---:|
| Good | 0.6181 | 0.7579 | **0.7878** |
| Satisfactory | **0.6943** | 0.6648 | 0.6734 |
| Moderate | **0.6257** | 0.6097 | 0.5863 |
| Poor | 0.4198 | 0.4158 | **0.4399** |
| Very Poor | **0.4842** | 0.4835 | 0.4477 |
| Severe | 0.5969 | 0.6372 | **0.6412** |

The tuned Random Forest substantially improved Good recall compared with the previous Random Forest, increasing from 0.6181 to 0.7579. Severe recall also increased from 0.5969 to 0.6372.

However, recall decreased slightly for Satisfactory, Moderate, Poor, and Very Poor.

Gradient Boosting achieved the highest recall for Good, Poor, and Severe among these three candidates.

The previous Random Forest achieved the highest recall for Satisfactory, Moderate, and Very Poor.

No model consistently performed best across all six categories and aggregate metrics.

## 8. Overall Findings and Current Model Status

The previous Random Forest remains the strongest overall candidate when considering accuracy and weighted F1.

The tuned Random Forest achieved the highest macro F1, but the improvement was modest and came with lower accuracy and weighted F1.

Gradient Boosting demonstrated competitive macro F1 and stronger recall for Poor and Severe.

The results demonstrate a trade-off between aggregate performance and class-wise performance. Final model selection should therefore consider balanced performance across all six AQI categories rather than relying on a single evaluation metric.

**No final model has been selected at this stage.**

## 9. Limitations

1. **Repeated validation-based selection:** Multiple hyperparameter configurations were evaluated using the same validation period. The best observed validation score may therefore be optimistic due to repeated model selection.

2. **Previously used test period:** The test period was used in earlier project experiments and cannot be treated as a completely untouched holdout.

3. **Temporal and geographic scope:** These results come from the India air-quality dataset and do not establish transferability to Pakistan or other regions.

4. **Class-specific performance:** Poor and Very Poor remain challenging categories, and no model consistently achieves strong recall across all six categories.

5. **Feature importance and association:** Model performance and feature importance do not establish causal relationships between individual pollutants and AQI categories.

## 10. Next Steps

- Consolidate the leading candidates and review their complete class-wise precision, recall, and F1 scores.
- Define a transparent final evaluation procedure that acknowledges prior test-period use.
- Evaluate the selected candidate or candidates on the chronological test partition as a final project assessment, clearly documenting the test-history limitation.
- Document the final model selection, supporting evidence, and remaining limitations.