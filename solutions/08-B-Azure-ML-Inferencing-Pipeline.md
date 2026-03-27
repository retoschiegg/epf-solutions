```python
def get_inferencing_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get inferencing component."""
    return entities.CommandComponent(
        name=f"{name_prefix}epf_inferencing_command",
        inputs={
            "public_data_root_dir": Input(type=constants.AssetTypes.URI_FOLDER, mode="ro_mount"),
            "model_name": Input(type="string"),
            "model_version": Input(type="string"),
            "date_from": Input(type="string", optional=True),
            "date_to": Input(type="string", optional=True),
        },
        outputs={
            "predictions": Output(type=constants.AssetTypes.URI_FILE, mode="rw_mount"),
        },
        code="./src/",
        command=(
            "export PYTHONPATH=$PYTHONPATH:$(pwd) &&"
            " python inferencing_script.py"
            " --public-data-root-dir=${{inputs.public_data_root_dir}}"
            " --model-name=${{inputs.model_name}}"
            " --model-version=${{inputs.model_version}}"
            " --predictions-file=${{outputs.predictions}}"
            " $[[ --date-from=${{inputs.date_from}} ]]"
            " $[[ --date-to=${{inputs.date_to}} ]]"
        ),
        environment=environment,
        resources=_get_resources(),
        is_deterministic=False,  # to disable caching
    )

def get_inferencing_pipeline_func(
    environment: str, name_prefix: str = "", compute: str = "aml-compute-cpu"
) -> Callable[..., entities.PipelineJob]:
    """Get inferencing pipeline function."""
    inferencing_component = get_inferencing_component(environment, name_prefix=name_prefix)

    @dsl.pipeline(compute=compute, name=f"{name_prefix}epf_inferencing_pipeline", description="EPF Inferencing Pipeline")
    def epf_inferencing_pipeline(
        public_data_root_dir: Input(type=constants.AssetTypes.URI_FOLDER),
        model_name: Input(type="string"),
        model_version: Input(type="string"),
        date_from: Input(type="string", optional=True),
        date_to: Input(type="string", optional=True),
    ):
        inferencing_step = inferencing_component(
            public_data_root_dir=public_data_root_dir,
            model_name=model_name,
            model_version=model_version,
            date_from=date_from,
            date_to=date_to,
        )
        return {"predictions": inferencing_step.outputs.predictions}

    return epf_inferencing_pipeline
```
