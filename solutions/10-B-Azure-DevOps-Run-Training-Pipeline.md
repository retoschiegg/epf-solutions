```python
def register_model(subscription_id: str, resource_group_name: str, ml_workspace_name: str, training_job_name: str, model_name: str) -> None:
    """Register model."""
    output_artifact_name = "trained_model"
    model_path = f"azureml://jobs/{training_job_name}/outputs/{output_artifact_name}"

    ml_client = MLClient(
        DefaultAzureCredential(), subscription_id=subscription_id, resource_group_name=resource_group_name, workspace_name=ml_workspace_name
    )
    model = entities.Model(name=model_name, path=model_path, type=constants.AssetTypes.MLFLOW_MODEL)

    registered_model = ml_client.models.create_or_update(model)
```
