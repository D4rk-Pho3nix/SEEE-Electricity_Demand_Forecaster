# Short-Term Electricity Demand Forecasting with Surge Alerts

Leakage-safe next-hour electricity forecasts with surge alerts, using open data, in one click.

A machine learning project that forecasts a city's electricity demand 60 minutes ahead and flags expected demand surges. It runs from one Google Colab notebook that downloads its own data from Kaggle.

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Proposed Solution](#proposed-solution)
- [Datasets](#datasets)
- [Methodology](#methodology)
  - [Process Flow](#process-flow)
  - [Features](#features)
  - [Evaluation Design](#evaluation-design)
  - [Models](#models)
- [Results](#results)
- [Getting Started](#getting-started)
  - [Run in Google Colab](#run-in-google-colab)
  - [Run Locally](#run-locally)
  - [Outputs](#outputs)
  - [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [Technology Stack](#technology-stack)
- [Deployment Architecture](#deployment-architecture)
- [Privacy and Data Protection](#privacy-and-data-protection)
- [How This Differs from Existing Solutions](#how-this-differs-from-existing-solutions)
- [Benefits to the Community](#benefits-to-the-community)
- [Success Factors](#success-factors)
- [Roadmap](#roadmap)
- [Cost of Implementation](#cost-of-implementation)
- [Risks and Mitigation](#risks-and-mitigation)
- [Challenges](#challenges)
- [Viability and Market Feasibility](#viability-and-market-feasibility)
- [Limitations](#limitations)
- [References](#references)

## Overview

| | |
|---|---|
| Category | Energy analytics and demand forecasting |
| Stage | Working prototype, validated on one year of historical data |
| Task | Regression (time-series forecasting), 60 minutes ahead |
| Surge alert | Next-hour demand at least 10% above the trailing 24-hour average |
| Runtime | Google Colab, about 5 minutes on a free CPU runtime |
| Main dataset | Tetouan city power consumption, 10-minute readings for 2017 |

## Problem Statement

Electricity demand rises and falls every day. When it peaks, supply costs more and the grid is under strain. Operators and households usually learn about a peak only after it has started, which leaves little time to respond.

This project answers one question: how much electricity will the city need in the next hour, and is a surge coming? With that answer, operators can plan reserves, and households and apps can move flexible usage such as laundry, water heating and EV charging to calmer hours.

Other problems were considered and rejected after checking the data:

| Candidate | Decision |
|---|---|
| Household bill prediction | The data is city level and has no household meters or tariffs. |
| Anomaly detection | There are no anomaly labels, so results could not be validated. |
| High versus low consumption classification | Better used as a view of the forecast, which is how the surge alert works. |
| Short-term demand forecasting | Selected. The series is complete and regular, the target is numeric, and the use is clear. |

## Proposed Solution

The notebook predicts total city demand (the sum of three zones) 60 minutes ahead from past demand, current weather and calendar information. A surge alert is then derived from the same forecast, so no second model is needed.

The alert compares the forecast with the average demand of the previous 24 hours. If the forecast is at least 10% higher, the next hour is flagged as a surge. The relative definition adapts to the season, which matters because the test months differ from the training months.

Accuracy for a forecasting task is not a built-in metric, so it is defined here as `100 - MAPE`. This is valid because total demand is always positive. MAE, RMSE, R2, sMAPE and the share of forecasts within 5% and 10% of the actual value are reported as well.

## Datasets

| Dataset | Source | Use in this project |
|---|---|---|
| Tetouan city power consumption | [Kaggle: fedesoriano/electric-power-consumption](https://www.kaggle.com/datasets/fedesoriano/electric-power-consumption) | Training, validation and testing |
| Individual household electric power consumption | [Kaggle: uciml/electric-power-consumption-data-set](https://www.kaggle.com/datasets/uciml/electric-power-consumption-data-set) | Downloaded and inspected only |

The Tetouan data has 52,416 rows at 10-minute intervals from 2017-01-01 to 2017-12-30, with temperature, humidity, wind speed, two solar radiation measures and the power consumption of three zones. It has no missing values or duplicate timestamps.

The UCI data describes a single household at 1-minute resolution with different variables and a different period. The two datasets cannot be merged, so the models use the Tetouan data and the notebook prints the structure of the UCI data for comparison.

## Methodology

### Process Flow

1. Download both datasets with `kagglehub`.
2. Standardise column names and parse timestamps.
3. Clean the data, rebuild a complete 10-minute time grid and fill short gaps with forward fill only.
4. Build features from information available at forecast time.
5. Run a leakage test that corrupts all future data and confirms earlier features do not change.
6. Split the year chronologically into train, validation and test periods.
7. Train the baselines and models and tune the best ones on training data.
8. Select the best model on the validation period.
9. Refit on train plus validation and evaluate once on the test period.
10. Derive the surge alert from the forecast and score it as a classifier.
11. Plot the results and save the model and result tables.

### Features

| Group | Examples |
|---|---|
| Current load | Total and per-zone consumption at the forecast origin |
| Lags | Total load from 10 minutes to 24 hours earlier |
| Seasonal references | Load at the same clock time yesterday and last week |
| Rolling statistics | Mean over 1, 6 and 24 hours, standard deviation over 6 and 24 hours, 1-hour change |
| Weather | Temperature, humidity, wind speed, solar flows, temperature change and 24-hour mean |
| Calendar | Hour, sine and cosine of the hour, day of week and weekend flag for the target time |

Month and day of year are not used. The data covers one year, so tree models cannot extrapolate to a season they have barely seen.

### Evaluation Design

- The split is chronological: 60% train, 20% validation, 20% test.
- A gap equal to the forecast horizon separates consecutive periods.
- Hyperparameters are tuned with `TimeSeriesSplit` on training data only.
- The validation period is used to choose the model.
- The test period is used once, after the model is chosen and refitted.

### Models

| Model | Purpose |
|---|---|
| Persistence | Baseline: next hour equals the current reading |
| Same time yesterday | Baseline: repeat the daily pattern |
| Same time last week | Baseline: repeat the weekly pattern |
| Ridge Regression | Linear reference with scaling inside a pipeline |
| Random Forest | Bagged trees, robust non-linear model |
| Extra Trees | Randomised trees, often smoother and faster |
| HistGradientBoosting | Gradient boosting for tabular data |

## Results

Test period, 60 minutes ahead, from one full run of the notebook:

| Measure | Value |
|---|---|
| Selected model | Ridge Regression |
| MAE | 940 power units |
| RMSE | 1,300 power units |
| R2 | 0.9915 |
| Accuracy (100 - MAPE) | 98.5% |
| Forecasts within 5% of actual | 96.6% |
| Forecasts within 10% of actual | 99.5% |
| Persistence baseline accuracy | 93.5% |
| MAE reduction versus persistence | 77.5% |

Surge alert on the test period:

| Rule | Accuracy | Balanced accuracy | Precision | Recall | F1 |
|---|---|---|---|---|---|
| Forecast-based | 98.3% | 98.1% | 95.8% | 97.8% | 96.8% |
| Persistence | 90.2% | 87.3% | 81.9% | 81.2% | 81.6% |

The 90% accuracy target is met. Because the persistence baseline already scores above 90%, the more useful evidence of skill is the 77.5% lower error against it. A horizon study in the notebook shows how accuracy falls as the forecast looks further ahead. Exact values can differ slightly between runs and machines.

## Getting Started

### Run in Google Colab

1. Download `Electricity_Demand_Forecasting_Colab.ipynb` from this repository.
2. Open [Google Colab](https://colab.research.google.com/) and choose File, then Upload notebook.
3. Choose Runtime, then Run all.

The notebook installs `kagglehub`, downloads both datasets and runs every step. No upload or configuration is needed. A full run takes about 5 minutes on a free CPU runtime.

### Run Locally

Requirements: Python 3.10 or newer.

```bash
pip install numpy pandas matplotlib seaborn scikit-learn joblib kagglehub jupyter
jupyter notebook Electricity_Demand_Forecasting_Colab.ipynb
```

Then choose Run All Cells. The notebook uses `display()` and inline plots, so run it in Jupyter or Colab rather than as a plain script.

### Outputs

A run creates an `outputs` folder next to the notebook:

| File | Content |
|---|---|
| `best_model.joblib` | Final fitted model |
| `test_results.csv` | Test metrics for the baselines and the final model |
| `validation_results.csv` | Validation metrics for every candidate |

The notebook also shows the model comparison, actual versus predicted values, residual plots, a week-long forecast view, feature importance, error by hour and weekday, the accuracy-versus-horizon study, and the surge alert confusion matrix, ROC and precision-recall curves.

### Troubleshooting

| Problem | Fix |
|---|---|
| The download asks for Kaggle credentials | Set `KAGGLE_USERNAME` and `KAGGLE_KEY` as environment variables, or place `kaggle.json` in `~/.kaggle/`. |
| The UCI dataset fails to load | The notebook prints a message and continues, because the models use only the Tetouan data. |
| The run is slow | The tuning step is the longest. Reduce `n_iter` in the search cell. |

## Project Structure

```
.
├── Electricity_Demand_Forecasting_Colab.ipynb   # full pipeline
├── README.md
└── outputs/                                     # created when the notebook runs
    ├── best_model.joblib
    ├── test_results.csv
    └── validation_results.csv
```

## Technology Stack

| Area | Tools |
|---|---|
| Language | Python |
| Data | pandas, NumPy |
| Modelling | scikit-learn |
| Visualisation | Matplotlib, Seaborn |
| Data access | kagglehub |
| Model storage | joblib |
| Runtime | Google Colab or Jupyter |

## Deployment Architecture

The notebook is a prototype. A deployed version would add:

- A scheduled job that pulls new meter and weather data.
- A model service exposed through a REST API.
- A database for readings and forecasts.
- A dashboard or alert channel such as SMS, email or app notifications.
- Scheduled retraining and error monitoring.

## Privacy and Data Protection

The data used here is aggregated at city level and describes no individual. A version that uses household meters would need:

- Consent from the people whose data is used.
- Data minimisation, so only the readings the model needs are collected.
- Aggregation or anonymisation before storage.
- Encryption in transit and at rest, with access controls.
- Compliance with the data protection law of the region.

Forecasts for a neighbourhood or feeder can be built from aggregated readings, which avoids exposing any single home.

## How This Differs from Existing Solutions

Utilities already use load forecasting software, and some smart thermostat and home energy apps send peak alerts. Most of these products are closed. This project is open and reproducible, runs on public data, tests for data leakage, and reports results next to simple baselines so the gain is visible. It also ties the forecast to a plain surge alert that does not require reading a chart.

## Benefits to the Community

- Lower costs when load moves away from peak periods.
- Fewer strained periods on the grid.
- Better planning for operators, which reduces waste.
- A simple signal for households about when electricity is under pressure.
- Open data and open-source tools, so other cities can repeat the work.

## Success Factors

- Clean, regular data that includes weather.
- Features built only from information known at forecast time.
- Honest evaluation on a held-out period.
- A simple output, either a number or a surge flag.
- Fast training, so models can be refreshed often.

## Roadmap

| Phase | Indicative duration | Scope |
|---|---|---|
| 1 | Completed | Prototype notebook on the Tetouan data |
| 2 | Weeks 3 to 6 | Add holidays, forecast weather and more years of data, and test across seasons |
| 3 | Weeks 7 to 10 | Wrap the model in an API and build a dashboard |
| 4 | Weeks 11 to 16 | Pilot with a utility or a group of households and measure behaviour change |

## Cost of Implementation

The prototype needs only staff time, since it uses free Colab and open-source libraries. Training takes a few minutes on a CPU, so no GPU spend is needed. Extending it requires weather forecast data, which may carry a licence fee, and a small server for the API. Exact costs depend on the pilot partner, data sources and hosting choice.

## Risks and Mitigation

| Risk | Mitigation |
|---|---|
| One year of data from one city | Add more years and test on other regions |
| No holiday or event calendar | Add a holiday calendar |
| Only current weather, no forecasts | Use forecast weather as an input |
| Model drift as habits change | Retrain on a schedule and monitor errors |
| Privacy risk with household data | Aggregate readings and follow data protection rules |
| Over-trusting a high accuracy figure | Report results next to simple baselines |

## Challenges

| Challenge | Approach |
|---|---|
| Getting clean real-world meter data | Work with a utility or a pilot community |
| Accuracy drops at longer horizons | Use forecast weather and more history, and use the horizon study to set limits |
| Getting people to act on alerts | Keep alerts rare and specific, and test the wording in a pilot |
| Turning alerts into measured savings | Run a controlled pilot with tariffs and track real load shifts |

## Viability and Market Feasibility

Technically, the approach is proven on public data. It trains in minutes and needs no special hardware, so it is cheap to build and refresh.

Three groups are the likely customers:

- Utilities and grid operators, who need short-term forecasts for dispatch and reserves.
- Building and campus energy managers, who want to avoid peak charges.
- Smart home and energy app makers, who want alerts that help users reduce bills.

Demand for this kind of tool is likely to grow as smart meters and time-of-use tariffs spread, since both reward users who can shift consumption. A realistic path is to start with one pilot partner, show measured savings, and then widen to more sites.

The main limits are the single-year dataset, the lack of real consumer response data and competition from established vendors. More data and a pilot would address the first two. Market size and willingness to pay have not been studied and would need research before a business case is written.

## Limitations

- One year of data from one city, so seasonal patterns are seen once and results may not transfer to other years or regions.
- The data is at zone level, not household level, so household bill prediction is out of scope.
- Only weather at the forecast origin is used, with no weather forecasts.
- No holiday or event calendar is included.
- The power unit in the dataset is undocumented, so absolute errors are in dataset units. Percentage metrics are unit free.
- The test set is a single hold-out period, so results do not reflect variation between years.
- High accuracy at 60 minutes is partly structural, since demand is strongly autocorrelated and simple baselines also score well.

## References

1. Salam, A., and El Hibaoui, A. (2018). Comparison of Machine Learning Algorithms for the Power Consumption Prediction: Case Study of Tetouan City. 2018 6th International Renewable and Sustainable Energy Conference (IRSEC). This paper is the source of the Tetouan dataset.
2. Hong, T., and Fan, S. (2016). Probabilistic electric load forecasting: A tutorial review. International Journal of Forecasting, 32(3), 914-938.
3. Hippert, H. S., Pedreira, C. E., and Souza, R. C. (2001). Neural networks for short-term load forecasting: A review and evaluation. IEEE Transactions on Power Systems, 16(1), 44-55.
4. Kuster, C., Rezgui, Y., and Mourshed, M. (2017). Electrical load forecasting models: A critical systematic review. Sustainable Cities and Society, 35, 257-270.
5. Cerqueira, V., Torgo, L., and Mozetic, I. (2020). Evaluating time series forecasting models: An empirical study on performance estimation methods. Machine Learning, 109, 1997-2028.
6. Lim, B., Arik, S. O., Loeff, N., and Pfister, T. (2021). Temporal Fusion Transformers for interpretable multi-horizon time series forecasting. International Journal of Forecasting, 37(4), 1748-1764.
7. Hyndman, R. J., and Athanasopoulos, G. (2021). Forecasting: Principles and Practice, 3rd edition. OTexts.
8. Hebrail, G., and Berard, A. (2012). Individual household electric power consumption. UCI Machine Learning Repository.
9. Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. Journal of Machine Learning Research, 12, 2825-2830.
