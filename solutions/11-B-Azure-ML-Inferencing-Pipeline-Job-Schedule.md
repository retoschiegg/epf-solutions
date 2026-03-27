```python
def deploy_inferencing_pipeline(
    subscription_id: str,
    resource_group_name: str,
    ml_workspace_name: str,
    deployment_config: DeploymentConfig,
    scheduled_job_name_prefix: str,
) -> None:
    """Deploy inferencing pipeline."""
    ml_client = MLClient(
        DefaultAzureCredential(), subscription_id=subscription_id, resource_group_name=resource_group_name, workspace_name=ml_workspace_name
    )

    epf_inferencing_pipeline_component = ml_client.components.get(
        name=deployment_config.component_name, version=deployment_config.component_version
    )

    epf_inferencing_pipeline_job = entities.PipelineJob(
        component=epf_inferencing_pipeline_component,
        inputs={
            "public_data_root_dir": Input(
                type=constants.AssetTypes.URI_FOLDER, mode="ro_mount", path=deployment_config.inputs_public_data_root_dir
            ),
            "model_name": deployment_config.inputs_model_name,
            "model_version": deployment_config.inputs_model_version,
        },
        outputs={"predictions": Output(type=constants.AssetTypes.URI_FILE, path=deployment_config.outputs_predictions_file)},
        settings=entities.PipelineJobSettings(
            default_compute="aml-compute-cpu",
            force_rerun=True,
        ),
    )

    cron_trigger = entities.CronTrigger(expression="0 14 * * *")  # 15:00 CET
    schedule = entities.JobSchedule(
        name=f"{scheduled_job_name_prefix}daily-epf-inference",
        trigger=cron_trigger,
        create_job=epf_inferencing_pipeline_job,
    )
    created_schedule_poller = ml_client.schedules.begin_create_or_update(schedule)
    created_schedule = created_schedule_poller.result()
```
