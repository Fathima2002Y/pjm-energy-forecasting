# ⚡ PJM West Electricity Demand Forecasting

Daily electricity-demand forecasting for the PJM West region: data cleaning, EDA, ARIMA vs SARIMA vs XGBoost comparison on a held-out year, and an interactive Streamlit app.

**🔴 Live demo:** https://hourly-consumption-forecast-abtva8ingoigzns37dgvyk.streamlit.app/

## Business problem
Grid operators need to anticipate demand to plan generation and avoid shortages or waste. Can we forecast daily electricity load accurately enough to be useful, and which model works best?

## Dataset
PJM Interconnection hourly energy consumption for the PJM West region (`PJMW_MW_Hourly.xlsx`): **143,206 hourly readings from April 2002 to August 2018**, in megawatts (MW).

## What I did
| Step | Highlights |
|---|---|
| Cleaning | Fixed data types, removed 4 duplicate timestamps (daylight-saving repeats), found 30 missing hourly timestamps (mostly daylight-saving gaps) and kept them as source gaps rather than inventing values |
| EDA | Hourly, daily, weekly, monthly, seasonal and yearly patterns, lag correlations, ADF stationarity test, ACF/PACF |
| Modeling frequency | Aggregated to **5,962 daily observations** |
| Validation | Chronological split: final **365 days (4 Aug 2017 - 3 Aug 2018)** held out as the test period for all models |
| ARIMA | Order (3,0,3) chosen by AIC grid search |
| SARIMA | Order (2,0,2)(0,1,1,7), weekly seasonality, chosen by AIC grid search |
| XGBoost | 15 calendar, lag and rolling features; `GridSearchCV` with `TimeSeriesSplit`; recursive day-by-day forecasting |
| App | Streamlit app with 1-90 day forecasts, live evaluation and hourly patterns |

## Results (365-day held-out test year)
| Model | MAE (MW) | RMSE (MW) | MAPE (%) | R² |
|---|---|---|---|---|
| ARIMA (3,0,3) | 586.42 | 779.11 | 10.06 | -0.034 |
| SARIMA (2,0,2)(0,1,1,7) | 565.59 | 758.14 | 9.59 | 0.021 |
| **XGBoost** | **455.23** | **602.51** | **7.82** | **0.382** |

XGBoost has the lowest error on every metric: about **22% lower MAE** than ARIMA and about **20% lower** than SARIMA. Selected parameters: 500 trees, max depth 3, learning rate 0.05, subsample 0.8, column sample 0.8.

## Key findings
- **Daily pattern:** lowest around 4 AM, rising from about 7 AM and peaking around 7-8 PM.
- **Weekly pattern:** weekdays average about 5,755 MW vs about 5,222 MW on weekends (roughly 10% higher).
- **Seasonal pattern:** winter is highest (about 6,269 MW), then summer (5,734 MW); spring (5,224 MW) and autumn (5,200 MW) are lowest. January is the highest month.
- **No strong long-term trend** across 2002-2018.

## Limitations
- The XGBoost forecast covers a whole year recursively, with each prediction feeding the next, so errors compound. R² of 0.38 is modest; ARIMA and SARIMA were close to predicting the average.
- Only calendar and past-demand features are used. Weather (especially temperature) and holidays would likely improve accuracy a lot.
- ARIMA and SARIMA residuals still show autocorrelation (Ljung-Box test), so they leave some structure unexplained.
- Daily averaging hides intra-day peaks.
- Next steps: add temperature and holiday features, try direct multi-step forecasting and hourly models, and add prediction intervals.

## Run locally
```bash
git clone https://github.com/<your-username>/pjm-energy-forecasting.git
cd pjm-energy-forecasting
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements-notebook.txt

jupyter notebook pjm_energy_forecasting.ipynb   # run all cells
streamlit run app.py
```

## Project structure
```
├── pjm_energy_forecasting.ipynb   # cleaning, EDA, ARIMA, SARIMA, XGBoost, evaluation
├── app.py                         # Streamlit app (trains XGBoost on startup)
├── PJMW_MW_Hourly.xlsx            # data
├── requirements.txt               # what the deployed app needs
└── requirements-notebook.txt
```

## Tech stack
Python · pandas · statsmodels · XGBoost · scikit-learn · Altair · Streamlit · Git
