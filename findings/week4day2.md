# Machine Learning-Based Air Quality Classification System

## Daily Progress Report — 23 October 2026

### Work Completed

Continued the validation and error analysis of three Decision Tree classifiers: the baseline model, the balanced model, and the custom-weighted model. All three were evaluated using the same complete-case validation dataset containing 58,831 observations.

The baseline model achieved an accuracy of 62.79%, a macro F1-score of 0.4640, and a weighted F1-score of 0.6052. The balanced model achieved 56.07% accuracy, a macro F1-score of 0.4905, and a weighted F1-score of 0.5652. The custom-weighted model achieved 61.84% accuracy, a macro F1-score of 0.4624, and a weighted F1-score of 0.6003.

The balanced model achieved the highest macro F1-score, while the baseline model achieved the highest accuracy and weighted F1-score. The baseline and custom-weighted models did not predict any Good observations, whereas the balanced model achieved a recall of 77.7% for this category.

### Analysis of Serious Pollution Categories

Compared the models' recall for Poor, Very Poor, and Severe observations.

* **Baseline model:** Poor recall 35.80%, Very Poor recall 45.32%, and Severe recall 47.15%.
* **Balanced model:** Poor recall 28.44%, Very Poor recall 49.88%, and Severe recall 58.04%.
* **Custom-weighted model:** Poor recall 37.40%, Very Poor recall 54.48%, and Severe recall 48.63%.

The custom-weighted model achieved the highest recall for Poor and Very Poor observations, while the balanced model performed best on Severe observations. These results demonstrate that class-weighting strategies affect the categories a model identifies most successfully.

### Investigation of Classification Errors

Examined confusion matrices and pollutant distributions to investigate why actual Poor observations were misclassified by the balanced model.

Of the 6,787 actual Poor observations, 1,930 were correctly classified as Poor, 2,701 were classified as Moderate, and 1,853 were classified as Very Poor.

Compared PM2.5 concentrations across these prediction groups. The median concentrations were 100.66 for correctly classified Poor observations, 72.19 for observations misclassified as Moderate, and 143.57 for observations misclassified as Very Poor.

These patterns suggest an association between PM2.5 concentrations and the model's classification errors. However, the overlapping distributions indicate that PM2.5 alone cannot explain every prediction. Other pollutant features and the model's learned decision boundaries may also contribute.

### Conclusions and Next Steps

The validation experiments demonstrate a trade-off between overall predictive performance and detection across individual AQI categories. No model has yet been selected as the final candidate.

The next stage will focus on refining the preprocessing strategy, investigating missing-data handling, and assessing whether more pollutant features can be retained without compromising model reliability. These investigations will also address the supervisor's preference for retaining pollutant features wherever possible.

Further temporal and city-level generalization experiments, robustness evaluation, and assessment of transferability to Pakistan remain part of the broader project plan.
