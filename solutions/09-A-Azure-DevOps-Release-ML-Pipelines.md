```python
def register_training_pipeline(
    subscription_id: str,
    resource_group_name: str,
    ml_workspace_name: str,
    component_version: str,
    component_environment: str,
    component_name_prefix: str,
) -> None:
    """Register training pipeline as component."""
    ml_client = MLClient(
        DefaultAzureCredential(), subscription_id=subscription_id, resource_group_name=resource_group_name, workspace_name=ml_workspace_name
    )

    epf_training_pipeline_func = azure_training.get_training_pipeline_func(
        environment=component_environment, name_prefix=component_name_prefix
    )
    epf_training_pipeline = epf_training_pipeline_func()

    # Register the pipeline as a component
    epf_training_pipeline_component = ml_client.components.create_or_update(epf_training_pipeline.component, component_version)
```
