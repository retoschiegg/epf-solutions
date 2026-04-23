```python
class IntegrationTestInferencingPipeline(unittest.TestCase):
    # ... existing code

    @classmethod
    def setUpClass(cls):
        deployment_config = get_deployment_config(cls.deployment_config_file)
        cls.ml_client = MLClient(
            DefaultAzureCredential(),
            subscription_id=cls.subscription_id,
            resource_group_name=cls.resource_group_name,
            workspace_name=cls.ml_workspace_name,
        )

        epf_inferencing_pipeline_component = cls.ml_client.components.get(
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
            settings=entities.PipelineJobSettings(
                default_compute="aml-compute-cpu",
                force_rerun=True,
            ),
        )

        epf_inferencing_pipeline_job = cls.ml_client.jobs.create_or_update(epf_inferencing_pipeline_job, experiment_name="integration-test")
        cls.ml_client.jobs.stream(epf_inferencing_pipeline_job.name)
        cls.pipeline_job = cls.ml_client.jobs.get(epf_inferencing_pipeline_job.name)

    def test_job_status(self):
        """Test if pipeline job completed successfully."""
        self.assertEqual(self.pipeline_job.status, "Completed")

    def test_data_in_predictions_file(self):
        """Test if the csv file can be read and contains expected number of rows."""
        expected_number_of_rows = 24 * 4
        predictions_df = self._get_predictions_from_job(self.ml_client, self.pipeline_job)
        self.assertEqual(len(predictions_df), expected_number_of_rows)
```
