```yaml
- stage: ProdDeployment
  displayName: 'Prod-Deployment'
  dependsOn: DevDeployment
  jobs:
  - deployment: DeployToProd
    displayName: 'Deploy to Prod'
    environment: 'prod'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.13' 
            displayName: 'Set Python 3.13'

          - bash: |
              pip install --upgrade pip
              pip install omegaconf==2.3.0 azure-ai-ml==1.31.0
            displayName: 'Install azure-ai-ml'

          - checkout: self

          - task: AzureCLI@2
            displayName: 'Register Model'
            name: RegisterModel
            inputs:
              azureSubscription: $(ARM_SERVICE_CONNECTION)
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail
                SUBSCRIPTION_ID=$(az account show --query id -o tsv)

                python azure_copy_registered_model.py \
                --subscription-id $SUBSCRIPTION_ID \
                --resource-group-name $(RESOURCE_GROUP) \
                --ml-workspace-name-from $(ML_WORKSPACE_DEV) \
                --ml-workspace-name-to $(ML_WORKSPACE_PROD) \
                --deployment-config-file $(DEPLOYMENT_CONFIG_FILE)

          - task: AzureCLI@2
            displayName: 'Deploy Inferencing Pipeline'
            name: DeployInferencingPipeline
            inputs:
              azureSubscription: $(ARM_SERVICE_CONNECTION)
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail
                SUBSCRIPTION_ID=$(az account show --query id -o tsv)

                python azure_deploy_inferencing_pipeline.py \
                --subscription-id $SUBSCRIPTION_ID \
                --resource-group-name $(RESOURCE_GROUP) \
                --ml-workspace-name $(ML_WORKSPACE_PROD) \
                --deployment-config-file $(DEPLOYMENT_CONFIG_FILE)
```
