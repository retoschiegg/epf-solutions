**electricity_price_forecasting/utils/data_processing_util.py**

Extending the the lag list with `24`.

```python
def _add_additional_features(windows: pd.DataFrame, forecast_date_column: str, ts_features: pd.DataFrame) -> pd.DataFrame:
    """Add additional features per window."""
    prices_per_window = windows.groupby(forecast_date_column)[constants.PricesCols.PRICE].apply(list).to_dict()
    for lag in [1, 24]:  # last and first price of previous day
        ts_features[f"lag_{lag}"] = [prices_per_window[forecast_date][-lag] for forecast_date in ts_features.index]
    ts_features = ts_features.assign(dayofweek=ts_features.index.dayofweek)
    return ts_features
```
