```yaml
- stage: ReleaseToProd
  displayName: 'Release to Prod'
  dependsOn:
    - Build
    - ReleaseToDev
  jobs:
  - job: ReleaseMLAssets
    displayName: 'Release ML Assets'
    variables:
      DOCKER_IMAGE: $[ stageDependencies.Build.BuildDocker.outputs['SetOutput.DOCKER_IMAGE'] ]
      AZURE_ML_ENVIRONMENT: $[ stageDependencies.ReleaseToDev.ReleaseMLAssets.outputs['RegisterMLEnv.AZURE_ML_ENVIRONMENT'] ]
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.13' 
      displayName: 'Set Python 3.13'

    - bash: |
        pip install --upgrade pip
        pip install azure-ai-ml==1.31.0
      displayName: 'Install azure-ai-ml'

    - checkout: self
    
    - task: AzureCLI@2
      displayName: 'Register ML Environment'
      name: RegisterMLEnv
      inputs:
        azureSubscription: $(ARM_SERVICE_CONNECTION)
        scriptType: bash
        scriptLocation: inlineScript
        inlineScript: |
          set -euo pipefail
          SUBSCRIPTION_ID=$(az account show --query id -o tsv)
          ENVIRONMENT_VERSION="${AZURE_ML_ENVIRONMENT##*:}"

          python src/azure_register_ml_environment.py \
            --subscription-id $SUBSCRIPTION_ID \
            --resource-group-name $(RESOURCE_GROUP) \
            --ml-workspace-name $(ML_WORKSPACE_PROD) \
            --docker-image $(DOCKER_IMAGE) \
            --environment-version $ENVIRONMENT_VERSION

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

          python src/azure_register_inferencing_pipeline.py \
          --subscription-id $SUBSCRIPTION_ID \
          --resource-group-name $(RESOURCE_GROUP) \
          --ml-workspace-name $(ML_WORKSPACE_PROD) \
          --component-version $COMMIT_SHA \
          --component-environment $(AZURE_ML_ENVIRONMENT)
```
