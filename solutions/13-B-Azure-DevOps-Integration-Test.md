```yaml
stages:
- stage: IntegrationTesting
  displayName: 'Integration Testing'
  jobs:
  - job: IntegrationTest
    displayName: 'Integration Test'
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.13' 
      displayName: 'Set Python 3.13'

    - bash: |
        pip install --upgrade pip
        pip install omegaconf==2.3.0 azure-ai-ml==1.31.0 pandas==2.3.3
      displayName: 'Install azure-ai-ml'

    - checkout: self

    - task: AzureCLI@2
      displayName: 'Test Inferencing Pipeline'
      name: TestInferencingPipeline
      inputs:
        azureSubscription: $(ARM_SERVICE_CONNECTION)
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          SUBSCRIPTION_ID=$(az account show --query id -o tsv)

          python test_azure_inferencing_pipeline.py \
          --subscription-id $SUBSCRIPTION_ID \
          --resource-group-name $(RESOURCE_GROUP) \
          --ml-workspace-name $(ML_WORKSPACE_DEV) \
          --deployment-config-file $(DEPLOYMENT_CONFIG_FILE)
```
