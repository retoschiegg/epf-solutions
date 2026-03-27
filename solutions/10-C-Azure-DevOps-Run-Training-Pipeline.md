```yaml
- job: RegisterModel
  displayName: 'Register Model'
  dependsOn: 
  - RunTrainingPipeline
  - WaitForTrainingJobCompletion
  variables: 
    TRAINING_JOB_NAME: $[ dependencies.RunTrainingPipeline.outputs['SubmitTrainingPipelineJob.TRAINING_JOB_NAME'] ]
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
    displayName: 'Register Model'
    name: RegisterModel
    inputs:
      azureSubscription: $(ARM_SERVICE_CONNECTION)
      scriptType: bash
      scriptLocation: inlineScript
      inlineScript: |
        set -euo pipefail
        SUBSCRIPTION_ID=$(az account show --query id -o tsv)

        python src/azure_register_model.py \
          --subscription-id $SUBSCRIPTION_ID \
          --resource-group-name $(RESOURCE_GROUP) \
          --ml-workspace-name $(ML_WORKSPACE_DEV) \
          --training-job-name $(TRAINING_JOB_NAME)
```
