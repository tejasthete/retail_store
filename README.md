# Retail Demand Forecasting

**Forecasting the next 8 weeks of retail demand so inventory can be planned, with five models compared using walk-forward validation.**

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange) ![statsmodels](https://img.shields.io/badge/statsmodels-time%20series-green) ![Streamlit](https://img.shields.io/badge/Streamlit-demo-red)

> **Status: work in progress.** The pipeline, models and app are built. The results table below is being finalized, and any `XX` is a number I have not filled in yet. I would rather show an honest placeholder than an unverified claim.

---

## Results

Evaluated with a rolling-origin (walk-forward) backtest: 6 folds, each forecasting 8 weeks ahead, 48 weeks tested in total. Scores are averages across folds.

| Model | MAE | RMSE | MAPE | MAE improvement vs naive |
|---|---|---|---|---|
| Naive (last value) | XX | XX | XX% | baseline |
| Seasonal Naive (same week last year) | XX | XX | XX% | XX% |
| Holt-Winters | XX | XX | XX% | XX% |
| ARIMA + Fourier terms | XX | XX | XX% | XX% |
| Gradient Boosting (lag features) | XX | XX | XX% | XX% |

**Key takeaway:** `XX` (e.g. "Model X cut forecast error by Y% versus the naive baseline, and the gap was largest in the December peak").

![Backtest](reports/figures/05_backtest.png)

![8-week forecast](reports/figures/07_forecast.png)

---

## The business problem

A retailer needs to decide how much stock to order, and ordering takes weeks. A forecast of total weekly demand over the next 8 weeks turns that into a concrete number instead of a guess. Under-forecasting means stockouts and lost sales. Over-forecasting means money tied up in inventory.

## Dataset

[Store Item Demand Forecasting Challenge](https://www.kaggle.com/c/demand-forecasting-kernels-only) (Kaggle): 5 years of daily sales across 10 stores and 50 items.

| Column | Description |
|---|---|
| `date` | Day of sale |
| `store` | Store ID |
| `item` | Item ID |
| `sales` | Units sold |

I summed all stores and items into **total weekly sales**, giving one clean series of roughly 260 weeks.

The data is not included in this repo. Download `train.csv` from Kaggle and place it in `data/raw/`.

---

## Approach

1. **EDA:** trend, yearly seasonality, seasonal decomposition and autocorrelation (`eda.py`).
2. **Baselines first:** naive and seasonal naive. A model that cannot beat these is not useful.
3. **Statistical models:** Holt-Winters (additive trend and seasonality) and ARIMA(1,1,1) with Fourier terms for yearly seasonality.
4. **Machine learning:** gradient boosting on lag, rolling-window and calendar features.
5. **Honest evaluation:** walk-forward validation only. No random train/test splits.
6. **Final forecast:** every model retrained on all data to forecast the next 8 weeks.
7. **Demo:** a small Streamlit app to explore forecasts and compare models.

### Design decisions worth calling out

- **No data leakage.** Time series cannot be split randomly, because the model would learn from the future. Every fold trains only on data before its forecast window.
- **Direct multi-step forecasting.** Every lag and rolling feature for the gradient boosting model is shifted by at least 8 weeks. The model can therefore predict all 8 weeks in one go, without feeding its own predictions back in, which avoids compounding errors.
- **Fourier terms instead of SARIMA(m=52).** A 52-period seasonal ARIMA is very heavy and unstable for weekly data. Fourier terms capture yearly seasonality cheaply.
- **Same interface for every model:** `f(train, horizon) -> forecast`, so adding a new model is one function.

---

## Project structure

```
retail-demand-forecasting/
├── README.md
├── requirements.txt
├── eda.py                  # exploratory analysis, saves figures
├── train.py                # backtest, model comparison, final forecast
├── app/
│   └── streamlit_app.py    # interactive demo
├── src/
│   ├── config.py           # horizon, folds, seasonality
│   ├── data_loader.py      # raw data -> weekly series
│   ├── features.py         # leak-free lag/rolling/calendar features
│   ├── models.py           # the five models
│   └── evaluate.py         # metrics and walk-forward backtest
├── data/
│   ├── raw/                # put train.csv here
│   └── processed/
├── models/
├── notebooks/
└── reports/
    ├── metrics_summary.csv
    ├── forecast_next_8_weeks.csv
    └── figures/
```

---

## How to run

```bash
# 1. Clone and install
git clone https://github.com/<your-username>/retail-demand-forecasting.git
cd retail-demand-forecasting
pip install -r requirements.txt

# 2. Add the data
#    Download train.csv from Kaggle into data/raw/

# 3. Run the pipeline
python eda.py        # EDA figures -> reports/figures/
python train.py      # backtest + forecast -> reports/

# 4. Launch the demo
streamlit run app/streamlit_app.py
```

If `data/raw/train.csv` is missing, the code falls back to synthetic data and prints a warning. That is only for testing the pipeline. Do not read anything into those results.

---

## Limitations

- Forecasts **total** demand only, not per store or per item.
- No holiday, promotion or price data, so unusual spikes are not explained.
- Point forecasts only, with no prediction intervals yet.
- One dataset. The approach has not been tested on a different retailer.

## Next steps

- [ ] Prediction intervals for the forecast
- [ ] Add Prophet as a sixth model
- [ ] Forecast per store and per product category
- [ ] Add holiday and promotion features
- [ ] Deploy the Streamlit app publicly

---

## About me

I'm Tejas Thete, a Robotics and AI graduate building a data science portfolio and learning in public. This project is part of that work.

- GitHub: [your-username](https://github.com/your-username)
- LinkedIn: [your-profile](https://www.linkedin.com/in/your-profile)
- Email: tejasthete007@gmail.com
