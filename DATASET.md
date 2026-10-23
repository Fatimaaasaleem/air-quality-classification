# Dataset Documentation

## 1. Dataset Overview

This project uses the Air Quality Data in India dataset, published on Kaggle by Rohan Rao.

The primary dataset used for hourly analysis is `city_hour.csv`.

| Property              | Details                   |
| --------------------- | ------------------------- |
| Dataset name          | Air Quality Data in India |
| Original file         | `city_hour.csv`           |
| Format                | CSV                       |
| Original dimensions   | 707,875 rows × 16 columns |
| Geographic coverage   | 26 Indian cities          |
| Temporal coverage     | January 2015 – July 2020  |
| Observation frequency | Hourly                    |
| Target variable       | `AQI_Bucket`              |
| Task                  | Multiclass classification |

The original dataset contains pollutant measurements and Air Quality Index (AQI) information collected across multiple cities.

## 2. Source and Attribution

* **Publisher:** Rohan Rao
* **Platform:** Kaggle
* **Dataset:** Air Quality Data in India
* **Original dataset page:** https://www.kaggle.com/datasets/rohanrao/air-quality-data-in-india

Refer to the original dataset page for its licensing terms, collection methodology, and attribution requirements.

## 3. Dataset Columns

The original `city_hour.csv` contains the following columns:

| Column       | Description                           |
| ------------ | ------------------------------------- |
| `City`       | City associated with the observation  |
| `Datetime`   | Timestamp of the observation          |
| `PM2.5`      | Fine particulate matter concentration |
| `PM10`       | Particulate matter concentration      |
| `NO`         | Nitric oxide measurement              |
| `NO2`        | Nitrogen dioxide measurement          |
| `NOx`        | Nitrogen oxides measurement           |
| `NH3`        | Ammonia measurement                   |
| `CO`         | Carbon monoxide measurement           |
| `SO2`        | Sulfur dioxide measurement            |
| `O3`         | Ozone measurement                     |
| `Benzene`    | Benzene measurement                   |
| `Toluene`    | Toluene measurement                   |
| `Xylene`     | Xylene measurement                    |
| `AQI`        | Air Quality Index value               |
| `AQI_Bucket` | Categorical air-quality label         |

Pollutant concentrations and their units should be interpreted according to the original dataset documentation.

## 4. Classification Target

The target variable is `AQI_Bucket`, which contains six categories:

* Good
* Satisfactory
* Moderate
* Poor
* Very Poor
* Severe

The classification task aims to predict the air-quality category from pollutant measurements.

The `AQI` column is excluded from the model's input features because it is directly associated with the construction of the categorical target. Including it could introduce target leakage and produce misleading evaluation results.

## 5. Data Availability and Missing Values

The original dataset contains missing measurements across several pollutant columns.

Missingness varies considerably by feature. For example, the initial data audit found approximately:

| Feature | Missing values |
| ------- | -------------: |
| Xylene  |         64.39% |
| PM10    |         41.92% |
| NH3     |         38.50% |
| Toluene |         31.16% |
| Benzene |         23.12% |
| PM2.5   |         20.50% |

These figures describe the original dataset examined during the initial audit. They should not be interpreted as the missingness percentages of a subsequently filtered dataset.

Missing-value handling and feature retention are being investigated as part of preprocessing.

## 6. Data-Quality Observations

The initial audit identified unusually high measurements in some pollutant columns, including:

* `PM2.5 = 999.99`
* `PM2.5 = 995.00`
* `PM10 = 999.99`

Some observations require further investigation to determine whether they represent valid extreme pollution measurements, measurement anomalies, or placeholder values.

These observations are being preserved pending further investigation rather than automatically removed.

## 7. Data Storage and Reproducibility

The original dataset is stored locally and is not included in the GitHub repository.

The repository's `.gitignore` rules exclude local data files from version control.

To reproduce the analysis, users must obtain the dataset from its original source, verify the applicable licensing terms, and place `city_hour.csv` in the expected local data directory.

The dataset should be processed according to the preprocessing steps documented in the project's notebooks and source code.

## 8. Current Status

Dataset evaluation and preprocessing investigations are ongoing.

The current work includes:

* Missing-value analysis
* Feature-retention assessment
* Investigation of suspicious extreme measurements
* Target-leakage prevention
* Evaluation of preprocessing alternatives

The final preprocessing strategy and classification baseline will be documented as the project progresses.
