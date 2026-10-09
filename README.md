# Machine Learning-Based Air Quality Classification System

A final year project investigating how machine learning can classify air quality categories from pollutant measurements, with Pakistan's air pollution and smog problem as the intended application.

**Status:** In progress. The repository currently covers dataset evaluation, exploratory analysis, data-quality investigation, and preliminary classification experiments. This is a classification project, not a forecasting system, and no final model exists yet.

## Dataset

- **Primary:** Indian hourly air quality data (`city_hour.csv`), 2015-2020, 707,875 rows x 16 columns, 26 cities.
- **Features:** PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3, Benzene, Toluene, Xylene.
- **Target:** `AQI_Bucket` (Good, Satisfactory, Moderate, Poor, Very Poor, Severe). `AQI` is excluded as an input to avoid target leakage.
- **Also explored:** a small Pakistan/Punjab dataset (`Saba-Gul/Forecasting-Air-Quality-Index`, six stations in Lahore, Multan and Gujranwala, about 2017-2020). It appears limited for the main task; its use for supplementary analysis is still being assessed.

## Tools

Python, Pandas, NumPy, Matplotlib, Seaborn, Scikit-learn, Jupyter Notebook, Git/GitHub.

## Work So Far

**Missing data:** Several pollutants have heavy missingness (Xylene 64.4%, PM10 41.9%, NH3 38.5%, Toluene 31.2%, PM2.5 20.5%). The aim is to preserve pollutant features rather than drop them unnecessarily. Handling strategies are still under investigation.

**Data quality:** No duplicate rows or negative values were found, but there are suspicious extreme readings (193 PM2.5 and 41 PM10 values at 999.99; 75 PM2.5 values at 995.00, all Hyderabad). No cleaning policy has been decided.

## Preliminary Results

Decision Tree (`max_depth=5`, `random_state=42`, stratified 80/20 split):

| Features | Rows retained | Accuracy |
| --- | ---: | ---: |
| PM2.5 | 547,760 | 55.24% |
| PM10 | 399,932 | 58.21% |
| PM2.5 + PM10 | 381,538 | 60.39% |
| PM2.5 + PM10 + NO + NO2 + NOx | 353,347 | 61.42% |
| Ten pollutants | 265,326 | 64.51% |

These are initial results only. Each experiment kept a different number of rows because of missing values, so the differences cannot be credited to feature choice alone.

## Structure

```text
air-quality-classification/
├── notebooks/
├── src/
├── .gitignore
├── LICENSE
├── README.md
└── requirements.txt
```

## Limitations

- The main dataset is from India; the system has not been trained or validated on Pakistani data.
- Missing values and extreme readings are unresolved.
- Results are preliminary and need controlled experiments.
