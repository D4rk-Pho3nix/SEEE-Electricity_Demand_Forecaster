# Short-Term Electricity Demand Forecasting

## Short description

A machine learning project that forecasts a city's electricity demand one hour ahead and flags expected demand surges. It runs from a single Google Colab notebook and downloads its own data from Kaggle.

## What problem does it address?

Electricity demand goes up and down every day. When it peaks, supply costs more and the grid is under strain. People and operators usually find out about a peak only after it has started.

The project answers one question: how much electricity will the city need in the next hour, and is a surge coming? A good answer lets operators plan reserves and lets households and apps move flexible usage, such as laundry, water heating or EV charging, to calmer hours.

## What was implemented?

The notebook is `Electricity_Demand_Forecasting_Colab.ipynb`. It does the following:

- Downloads the two Kaggle datasets automatically.
- Cleans the Tetouan data and checks it for gaps, duplicates and impossible values.
- Builds features that only use information available at the moment of the forecast: current load, past loads, same time yesterday and last week, rolling averages, weather and the hour and day of the target time.
- Splits the year chronologically into train, validation and test periods.
- Compares three simple baselines with Ridge Regression, Random Forest, Extra Trees and HistGradientBoosting.
- Tunes the tree models with time-aware cross-validation on training data only.
- Evaluates the chosen model once on the untouched test period.
- Turns the forecast into a surge warning and scores it as a classifier.
- Plots the results and saves the model and result tables.

## How did we proceed?

1. Looked at both datasets. The UCI data is one household at 1-minute readings. The Tetouan data is city level at 10-minute readings with weather. They cannot be merged, so the Tetouan data is used for modelling and the UCI data is only loaded and inspected.
2. Compared possible problems against what the data actually supports. Household bill prediction needs household meters and tariffs, which the data does not have. Anomaly detection has no labels to check against. Short-term demand forecasting fits the data and has a clear use.
3. Cleaned the data and built the features, then ran a test that corrupts all future data and confirms earlier features do not change.
4. Split the data by time, with a gap equal to the forecast horizon between periods.
5. Trained the baselines and models, tuned the best ones on training data, and picked the winner on the validation period.
6. Refitted the winner on train plus validation and tested it once.

Results from one full run on the Tetouan data, 60 minutes ahead, test period:

| Measure | Value |
|---|---|
| Best model | Ridge Regression |
| MAE | 940 power units |
| RMSE | 1,300 power units |
| R² | 0.9915 |
| Accuracy (100 minus MAPE) | 98.5% |
| Forecasts within 5% of actual | 96.6% |
| Persistence baseline accuracy | 93.5% |
| MAE reduction vs persistence | 77.5% |
| Surge warning balanced accuracy | 98.1% |

The 90% target was met. Because the plain persistence baseline already scores 93.5%, the better evidence of skill is the 77.5% drop in error against it.

## Title

Short-Term Electricity Demand Forecasting with Surge Alerts

## Problem statement

Demand peaks raise costs and stress the grid, and people get little warning before they happen. We need a reliable forecast of the next hour's demand that planners and households can act on.

## How does the solution align with the problem?

The model predicts demand 60 minutes ahead, which is early enough to change behaviour or adjust supply. The surge flag compares the forecast with the last 24 hours of demand and warns when the next hour is expected to be at least 10% above that average. This gives a simple signal that does not need anyone to read a forecast chart.

## Stage of development

Working prototype. The notebook runs end to end on one year of data from one city. It has not been deployed or tested with live data.

## Expected result

On the test period the model forecast the next hour with an MAE of 940 units and 98.5% accuracy as defined above. It cut the error of the persistence baseline by about three quarters. The expected result in use is earlier and more accurate peak warnings than a "same as now" assumption.

## Product category

Energy analytics and demand forecasting software, with a smart energy management use for households.

## Success factors

- Clean, regular data with weather included.
- Features built only from information known at forecast time.
- Honest evaluation on a held-out period.
- A simple output, either a number or a surge flag.
- Fast training, so models can be refreshed often.

## Achievable timeline

This is a proposed plan, not something already done.

- Weeks 1 to 2: prototype notebook (done).
- Weeks 3 to 6: add holidays, forecast weather and more years of data, and test across seasons.
- Weeks 7 to 10: wrap the model in an API and build a small dashboard.
- Weeks 11 to 16: pilot with a utility or a group of households and measure real behaviour change.

## Key benefits to the wider community

- Lower costs when load is moved away from peaks.
- Fewer strained periods on the grid.
- Better planning for operators, which reduces waste.
- A simple way for households to see when electricity is under pressure.
- Open data and open-source tools, so other cities can repeat the work.

## Underlying tech stack

Python, pandas and NumPy for data, scikit-learn for models, Matplotlib and Seaborn for plots, kagglehub for data download, and Google Colab as the runtime. A deployed version would add a REST API and a dashboard.

## Cost of implementation

The prototype costs nothing beyond staff time, since it uses free Colab and open-source libraries. Extending it needs weather forecast data, which may carry a licence fee, and a small server for the API. Training takes a few minutes on a CPU, so no GPU spend is needed. Exact costs depend on the pilot partner and hosting choice and have not been estimated here.

## How does it differ from solutions already in the market?

Utilities already use load forecasting software, and some smart thermostat and home energy apps send peak alerts. Most of these are closed products. This project is open and reproducible, runs on public data, tests for data leakage, and reports results against simple baselines so the gain is clear. It also links the forecast to a plain surge alert.

## IT infrastructure

For the prototype, only a browser and a Google account are needed. For a deployed version:

- A scheduled job that pulls new meter and weather data.
- A model service exposed through an API.
- A database for readings and forecasts.
- A dashboard or alert channel such as SMS, email or an app notification.

## Privacy and data protection

The data used here is city-level, so it describes no individual. If the project moves to household meters, it would need consent, data minimisation, anonymised or aggregated storage, encryption, and access controls, and it would have to follow local data protection law. Forecasts for a neighbourhood or feeder can be built from aggregated readings, which avoids exposing a single home.

## Process flow

1. Download the datasets.
2. Clean the data and fix the time grid.
3. Build features from past load, weather and calendar.
4. Split the data by time.
5. Train and compare the models, and tune the best.
6. Test the chosen model once.
7. Predict the next hour and raise a surge alert when the forecast is 10% above the 24-hour average.

## What differentiates this from competitors? What is the unique value proposition?

The value is trust. Every result is computed from a leakage-checked pipeline and compared with honest baselines, so the stated accuracy can be believed. It is free to run, easy to repeat, and gives a usable alert instead of only a number.

## Cost

Prototype: staff time only. Pilot and deployment: hosting, weather data and integration work, to be priced once a partner and scale are known.

## Uniqueness in 15 words

Leakage-safe next-hour electricity forecasts with surge alerts, using open data, in one click.

## Key risks and how to reduce them

- Only one year of data from one city. Add more years and test on other regions.
- Missing holidays and special events. Add a holiday calendar.
- No weather forecasts, only current weather. Add forecast weather as an input.
- Models can drift as habits change. Retrain on a schedule and monitor errors.
- Privacy risk with household data. Aggregate readings and follow data protection rules.
- Over-trusting a high accuracy figure. Always report results next to simple baselines.

## Challenges and how to overcome them

- Getting real, clean meter data. Work with a utility or a pilot community.
- Accuracy falls at longer horizons. The notebook includes a horizon study, and forecast weather and more history should help.
- Getting people to act on alerts. Keep alerts rare and specific, and test the wording in a pilot.
- Turning alerts into measured savings. Run a controlled pilot with tariffs and track real load shifts.

## Viability and market feasibility

Technically, the approach is proven on public data. It trains in minutes and needs no special hardware, so it is cheap to build and refresh.

On the market side, there are three likely customers. Utilities and grid operators need short-term forecasts for dispatch and reserves. Building and campus energy managers want to avoid peak charges. Smart home and energy app makers want alerts that help users cut bills. Demand is likely to grow as smart meters and time-of-use tariffs spread, since both reward people who can shift usage. A realistic path is to start with one pilot partner, show measured savings, and then widen to more sites. Market size and willingness to pay have not been studied here and should be checked before any business case is written.

The main limits to viability are the single-year dataset, the lack of real consumer response data, and competition from established vendors. These are fixable with more data and a pilot.

## References

1. Salam, A., and El Hibaoui, A. (2018). Comparison of Machine Learning Algorithms for the Power Consumption Prediction: Case Study of Tetouan City. 2018 6th International Renewable and Sustainable Energy Conference (IRSEC). This paper is the source of the Tetouan dataset.
2. Hong, T., and Fan, S. (2016). Probabilistic electric load forecasting: A tutorial review. International Journal of Forecasting, 32(3), 914-938.
3. Hippert, H. S., Pedreira, C. E., and Souza, R. C. (2001). Neural networks for short-term load forecasting: A review and evaluation. IEEE Transactions on Power Systems, 16(1), 44-55.
4. Kuster, C., Rezgui, Y., and Mourshed, M. (2017). Electrical load forecasting models: A critical systematic review. Sustainable Cities and Society, 35, 257-270.
5. Cerqueira, V., Torgo, L., and Mozetic, I. (2020). Evaluating time series forecasting models: An empirical study on performance estimation methods. Machine Learning, 109, 1997-2028.
6. Lim, B., Arik, S. O., Loeff, N., and Pfister, T. (2021). Temporal Fusion Transformers for interpretable multi-horizon time series forecasting. International Journal of Forecasting, 37(4), 1748-1764.
7. Hyndman, R. J., and Athanasopoulos, G. (2021). Forecasting: Principles and Practice, 3rd edition. OTexts.
8. Hebrail, G., and Berard, A. (2012). Individual household electric power consumption. UCI Machine Learning Repository.

Please check volume and page numbers against the publishers before citing them in a formal submission.

## Datasets

- Tetouan city power consumption: https://www.kaggle.com/datasets/fedesoriano/electric-power-consumption
- UCI household electric power consumption: https://www.kaggle.com/datasets/uciml/electric-power-consumption-data-set

## How to run

Open `Electricity_Demand_Forecasting_Colab.ipynb` in Google Colab and choose Runtime, then Run all. The notebook downloads the data itself.
