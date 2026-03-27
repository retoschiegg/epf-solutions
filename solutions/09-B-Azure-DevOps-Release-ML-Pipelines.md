```yaml
- task: AzureCLI@2
  displayName: 'Register ML Pipelines'
  name: RegisterMLPipelines
  inputs:
    azureSubscription: $(ARM_SERVICE_CONNECTION)
    scriptType: bash
    scriptLocation: inlineScript
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
```
