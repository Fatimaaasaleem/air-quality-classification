## Random Forest Experiments and Hyperparameter Tuning

### 1. Objective

Random Forest was introduced to evaluate whether an ensemble-based classifier could improve upon the existing Decision Tree baseline.

The experiments investigated:

- Standard Random Forest classification.
- Class weighting using `balanced` and `balanced_subsample`.
- Overfitting through training and validation performance.
- Tree-complexity control using `max_depth`, `min_samples_leaf`, and `min_samples_split`.

All experiments used the existing chronological training and validation partitions, the ten selected pollutant features, and median imputation fitted on the training data.

### 2. Random Forest Baseline

The initial Random Forest used 200 estimators, a fixed random state of 42, and all available CPU cores.

| Metric | Result |
|---|---:|
| Accuracy | 62.23% |
| Macro F1 | 0.4945 |
| Weighted F1 | 0.6042 |

The baseline improved upon the Decision Tree's aggregate performance. However, class-level evaluation revealed weaker recognition of Good, Poor, Very Poor, and Severe observations.

### 3. Class-Weighting Experiments

Two class-weighting strategies were evaluated.

| Model | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| Standard Random Forest | 62.23% | 0.4945 | 0.6042 |
| Balanced Random Forest | 61.15% | 0.5165 | 0.6059 |
| Balanced Subsample Random Forest | 61.48% | 0.4689 | 0.5940 |

The `balanced` strategy improved macro F1 compared with the standard Random Forest, although accuracy decreased.

The `balanced_subsample` strategy did not improve macro F1 and was therefore not selected for further tuning.

### 4. Overfitting Investigation

The balanced Random Forest initially showed a substantial difference between training and validation performance.

| Metric | Training | Validation |
|---|---:|---:|
| Accuracy | 97.74% | 61.15% |
| Macro F1 | 96.89% | 51.65% |

This indicated substantial overfitting.

Tree complexity was subsequently controlled through additional hyperparameter experiments.

| Configuration | Training Accuracy | Validation Accuracy | Training Macro F1 | Validation Macro F1 |
|---|---:|---:|---:|---:|
| Balanced RF, default complexity | 97.74% | 61.15% | 0.9689 | 0.5165 |
| `max_depth=15` | 73.05% | 60.35% | 0.7161 | 0.5426 |
| `max_depth=15`, `min_samples_leaf=5` | 70.37% | 60.35% | 0.6876 | 0.5497 |
| Previous settings, `min_samples_split=10` | 70.37% | 60.35% | 0.6876 | 0.5497 |
| Previous settings, `min_samples_split=20` | 69.05% | 60.28% | 0.6731 | 0.5499 |

Increasing `min_samples_split` from 5 to 10 did not change the results because the minimum leaf-size constraint already required at least ten samples for a valid split.

Increasing it to 20 produced a small additional improvement in validation macro F1.

The final configuration reduced the training-validation macro F1 gap from approximately 45.24 percentage points to 12.32 percentage points. However, some overfitting remained.

### 5. Selected Tuned Random Forest

The best configuration based on validation macro F1 was:

- `n_estimators=200`
- `max_depth=15`
- `min_samples_leaf=5`
- `min_samples_split=20`
- `class_weight="balanced"`
- `random_state=42`
- `n_jobs=-1`

Validation performance:

| Metric | Result |
|---|---:|
| Accuracy | 60.28% |
| Macro F1 | 0.5499 |
| Weighted F1 | 0.6056 |

This configuration achieved the highest macro F1 among the Random Forest experiments.

### 6. Class-Level Evaluation

| AQI Class | Correct Predictions | Total Observations | Recall |
|---|---:|---:|---:|
| Good | 2,754 | 4,378 | 62.91% |
| Satisfactory | 26,800 | 38,766 | 69.13% |
| Moderate | 31,058 | 49,578 | 62.64% |
| Poor | 6,903 | 16,502 | 41.83% |
| Very Poor | 6,360 | 13,326 | 47.73% |
| Severe | 4,712 | 7,820 | 60.26% |

The tuned model improved recognition across several minority classes compared with the original balanced Random Forest.

However, Poor and Very Poor remained the most difficult categories to classify.

Good recall increased substantially, but this came with relatively low precision (approximately 39%), indicating that some observations from other categories were incorrectly predicted as Good.

### 7. Confusion Matrix Analysis

The confusion matrix contained 51,783 incorrect predictions.

Using the natural AQI severity order:

Good → Satisfactory → Moderate → Poor → Very Poor → Severe

The errors were distributed as follows:

- Adjacent-category errors: 17,620 (34.03%).
- Errors involving categories separated by at least one intermediate category: 34,163 (65.97%).

Therefore, although some classification errors occurred between neighbouring AQI categories, a considerable proportion involved more distant categories.

The model's mistakes cannot be described as exclusively minor severity-level confusions.

### 8. Interpretation and Limitations

The experiments demonstrate that class weighting and controlling tree complexity improved macro F1 compared with the original Random Forest baseline.

The tuned configuration achieved a validation macro F1 of 0.5499, compared with 0.4945 for the standard Random Forest.

However, the following limitations remain:

1. Poor and Very Poor observations continue to have relatively low recall.
2. Good predictions have relatively low precision.
3. A training-validation performance gap remains, indicating residual overfitting.
4. The chronological validation strategy may also expose the model to temporal distribution differences between training and validation periods.
5. The current results represent performance on the existing validation population and do not establish generalization to Pakistani air-quality data.
6. The current test partition has appeared in earlier project experiments and must not be described as an untouched final holdout.

### 9. Conclusion

Random Forest provided an improvement over the Decision Tree baseline in aggregate performance. Class weighting improved macro F1, while controlling tree complexity further reduced overfitting and improved minority-class recognition.

The tuned balanced Random Forest is currently the strongest Random Forest configuration based on validation macro F1.

However, it is not yet a final selected model. Further evaluation and comparison under consistent validation conditions are required before making a final model-selection decision.