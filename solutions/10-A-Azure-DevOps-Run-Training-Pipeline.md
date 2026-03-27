```yaml
- job: WaitForTrainingJobCompletion
  displayName: 'Wait For Training Job Completion'
  pool: server
  timeoutInMinutes: 30
  dependsOn: RunTrainingPipeline
  variables: 
    TRAINING_JOB_NAME: $[ dependencies.RunTrainingPipeline.outputs['SubmitTrainingPipelineJob.TRAINING_JOB_NAME'] ]
  steps:
  - task: AzureMLJobWaitTask@1
    inputs:
      serviceConnection: $(ARM_SERVICE_CONNECTION)
      resourceGroupName: $(RESOURCE_GROUP)
      azureMLWorkspaceName: $(ML_WORKSPACE_DEV)
      azureMLJobName: $(TRAINING_JOB_NAME)
```
