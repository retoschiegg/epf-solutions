```python
def training(
    app_conf: app_config.AppConfig,
    public_data_root_dir: pathlib.Path,
    train_date_from: datetime.datetime,
    train_date_to: datetime.datetime,
    eval_date_from: datetime.datetime,
    eval_date_to: datetime.datetime | None,
    model_dir: pathlib.Path,
    train_artifact_dir: pathlib.Path,
    eval_artifact_dir: pathlib.Path,
) -> None:
    train_data_df = load_data.load_data_as_df(
        app_conf=app_conf,
        public_data_root_dir=public_data_root_dir,
        date_from=train_date_from,
        date_to=train_date_to,
        with_lookback_window=False,
        groundtruth_date_to=None,
    )
    eval_data_df = load_data.load_data_as_df(
        app_conf=app_conf,
        public_data_root_dir=public_data_root_dir,
        date_from=eval_date_from,
        date_to=eval_date_to,
        with_lookback_window=True,
        groundtruth_date_to=None,
    )

    train_model.train_model(
        app_conf=app_conf,
        prices_df=train_data_df,
        model_dir=model_dir,
        artifact_dir=train_artifact_dir
    )
    eval_model.eval_model(
        app_conf=app_conf,
        prices_df=eval_data_df,
        model_dir=model_dir,
        artifact_dir=eval_artifact_dir,
    )
```
