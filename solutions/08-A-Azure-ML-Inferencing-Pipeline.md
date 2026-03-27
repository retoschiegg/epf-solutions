```python
def inferencing(
    app_conf: app_config.AppConfig,
    public_data_root_dir: pathlib.Path,
    model_name: str,
    model_version: str,
    model_dir: str,
    predictions_file: pathlib.Path,
    date_from: datetime.datetime | None,
    date_to: datetime.datetime | None,
) -> None:
    """Load data and apply the model."""
    prices_df = load_data.load_data_as_df(
        app_conf=app_conf,
        public_data_root_dir=public_data_root_dir,
        date_from=date_from,
        date_to=date_to,
        with_lookback_window=True,
        groundtruth_date_to=None,
    )

    new_predictions_df = apply_model.apply_model(
        app_conf=app_conf,
        prices_df=prices_df,
        model_dir=model_dir,
        date_from=date_from,
        date_to=date_to,
        groundtruth_date_to=None,
    )
    new_predictions_df[constants.PredictionsCols.MODEL_NAME] = model_name
    new_predictions_df[constants.PredictionsCols.MODEL_VERSION] = model_version
    _save_predictions(predictions_file, new_predictions_df)
```
