**azure_register_inferencing_pipeline.py**

```python
"""Script to register the inferencing pipeline as component."""

import argparse

from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

import azure_inferencing


def register_inferencing_pipeline(
    subscription_id: str,
    resource_group_name: str,
    ml_workspace_name: str,
    component_version: str,
    component_environment: str,
    component_name_prefix: str,
) -> None:
    """Register inferencing pipeline as component."""
    ml_client = MLClient(
        DefaultAzureCredential(), subscription_id=subscription_id, resource_group_name=resource_group_name, workspace_name=ml_workspace_name
    )

    epf_inferencing_pipeline_func = azure_inferencing.get_inferencing_pipeline_func(
        environment=component_environment, name_prefix=component_name_prefix
    )
    epf_inferencing_pipeline = epf_inferencing_pipeline_func()

    # Register the pipeline as a component
    epf_inferencing_pipeline_component = ml_client.components.create_or_update(epf_inferencing_pipeline.component, component_version)
    print(f"Component {epf_inferencing_pipeline_component.name} with version {epf_inferencing_pipeline_component.version} is registered")  # noqa: T201


def cli():
    """Entrypoint."""
    parser = argparse.ArgumentParser()
    parser.add_argument("--subscription-id", type=str, help="Subscription ID containing the azure resources")
    parser.add_argument("--resource-group-name", type=str, help="Name of resource group containing the aml workspace")
    parser.add_argument("--ml-workspace-name", type=str, help="Name of ml workspace")
    parser.add_argument("--component-version", type=str, help="New release version to be used to register component")
    parser.add_argument("--component-environment", type=str, help="Environment used by component")
    parser.add_argument("--component-name-prefix", type=str, help="Prefix for the component name", default="")
    args = parser.parse_args()
    register_inferencing_pipeline(
        args.subscription_id,
        args.resource_group_name,
        args.ml_workspace_name,
        args.component_version,
        args.component_environment,
        args.component_name_prefix,
    )


if __name__ == "__main__":
    cli()
```


**azure-pipelines.yml**

```yaml
inlineScript: |
  set -euo pipefail
  SUBSCRIPTION_ID=$(az account show --query id -o tsv)
  COMMIT_SHA=$(Build.SourceVersion)

  python src/azure_register_training_pipeline.py \
  --subscription-id $SUBSCRIPTION_ID \
  --resource-group-name $(RESOURCE_GROUP) \
  --ml-workspace-name $(ML_WORKSPACE_DEV) \
  --component-version $COMMIT_SHA \
  --component-environment $(AZURE_ML_ENVIRONMENT)

  python src/azure_register_inferencing_pipeline.py \
  --subscription-id $SUBSCRIPTION_ID \
  --resource-group-name $(RESOURCE_GROUP) \
  --ml-workspace-name $(ML_WORKSPACE_DEV) \
  --component-version $COMMIT_SHA \
  --component-environment $(AZURE_ML_ENVIRONMENT)
```
