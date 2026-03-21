**electricity_price_forecasting/steps/train_model.py**

The lines of the function `train_model` should look similar to:

```python
mlflow.log_params(data_processing_util.flatten_dict(train_params, prefix="train"))
mlflow.log_metrics(data_processing_util.flatten_dict(train_metrics, prefix="train"))
mlflow.log_artifacts(artifact_dir.as_posix())
LOGGER.info("Training finished: %s", train_metrics)
```


**electricity_price_forecasting/steps/eval_model.py**

The lines of the function `eval_model` should look similar to:

```python
mlflow.log_params(data_processing_util.flatten_dict(eval_params, prefix="eval"))
mlflow.log_metrics(data_processing_util.flatten_dict(eval_metrics, prefix="eval"))
mlflow.log_artifacts(artifact_dir.as_posix())
LOGGER.info("Evaluation finished: %s", eval_metrics)
```
