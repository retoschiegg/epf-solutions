```python
def create_ml_environment(
    subscription_id: str,
    resource_group_name: str,
    ml_workspace_name: str,
    docker_image: str,
    environment_name_prefix: str,
    environment_version: str,
) -> None:
    """Create ml environment."""
    ml_client = MLClient(
        DefaultAzureCredential(), subscription_id=subscription_id, resource_group_name=resource_group_name, workspace_name=ml_workspace_name
    )

    env_definition = entities.Environment(name=f"{environment_name_prefix}electricity-price-forecasting", image=docker_image)
    if environment_version:
        env_definition.version = environment_version
    created_env = ml_client.environments.create_or_update(env_definition)
    epf_environment = f"azureml:{created_env.name}:{created_env.version}"
    print(epf_environment)
```
