```python
def get_training_pipeline_func(
    environment: str, name_prefix: str = "", compute: str = "aml-compute-cpu"
) -> Callable[..., entities.PipelineJob]:
    """Get training pipeline function."""
    training_component = get_training_component(environment, name_prefix=name_prefix)

    @dsl.pipeline(compute=compute, name=f"{name_prefix}epf_training_pipeline", description="EPF Training Pipeline")
    def epf_training_pipeline(
        public_data_root_dir: Input(type=constants.AssetTypes.URI_FOLDER, mode="ro_mount"),
        train_date_from: Input(type="string"),
        train_date_to: Input(type="string"),
        eval_date_from: Input(type="string"),
        eval_date_to: Input(type="string", optional=True),
    ):
        training_step = training_component(
            public_data_root_dir=public_data_root_dir,
            train_date_from=train_date_from,
            train_date_to=train_date_to,
            eval_date_from=eval_date_from,
            eval_date_to=eval_date_to,
        )
        return {
            "trained_model": training_step.outputs.trained_model,
            "training_artifacts": training_step.outputs.training_artifacts,
            "evaluation_artifacts": training_step.outputs.evaluation_artifacts,
        }

    return epf_training_pipeline
```
