```python
def get_load_data_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get load data component."""
    return entities.CommandComponent(
        name=f"{name_prefix}epf_load_data_command",
        inputs={
            "public_data_root_dir": Input(type=constants.AssetTypes.URI_FOLDER, mode="ro_mount"),
            "date_from": Input(type="string", optional=True),
            "date_to": Input(type="string", optional=True),
            "with_lookback_window": Input(type="boolean", optional=True),
        },
        outputs={
            "data_parquet_file": Output(type=constants.AssetTypes.URI_FILE, mode="rw_mount"),
        },
        code="./src/",
        command=(
            "export PYTHONPATH=$PYTHONPATH:$(pwd) &&"
            " python electricity_price_forecasting/steps/load_data.py"
            " --public-data-root-dir=${{inputs.public_data_root_dir}}"
            " --save-to-parquet-file=${{outputs.data_parquet_file}}"
            " $[[ --date-from=${{inputs.date_from}} ]]"
            " $[[ --date-to=${{inputs.date_to}} ]]"
            " $[[ --with-lookback-window=${{inputs.with_lookback_window}} ]]"
        ),
        environment=environment,
        resources=_get_resources(),
        is_deterministic=False,
    )


def get_train_model_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get train model component."""
    return entities.CommandComponent(
        name=f"{name_prefix}epf_train_model_command",
        inputs={
            "train_data_parquet_file": Input(type=constants.AssetTypes.URI_FILE, mode="ro_mount"),
        },
        outputs={
            "trained_model": Output(type=constants.AssetTypes.MLFLOW_MODEL, mode="rw_mount"),
            "artifact_dir": Output(type=constants.AssetTypes.URI_FOLDER, mode="rw_mount"),
        },
        code="./src/",
        command=(
            "export PYTHONPATH=$PYTHONPATH:$(pwd) &&"
            " python electricity_price_forecasting/steps/train_model.py"
            " --data-file=${{inputs.train_data_parquet_file}}"
            " --model-dir=${{outputs.trained_model}}"
            " --artifact-dir=${{outputs.artifact_dir}}"
        ),
        environment=environment,
        resources=_get_resources(),
        is_deterministic=True,
    )


def get_eval_model_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get eval model component."""
    return entities.CommandComponent(
        name=f"{name_prefix}epf_eval_model_command",
        inputs={
            "eval_data_parquet_file": Input(type=constants.AssetTypes.URI_FILE, mode="ro_mount"),
            "trained_model": Input(type=constants.AssetTypes.MLFLOW_MODEL, mode="ro_mount"),
        },
        outputs={
            "artifact_dir": Output(type=constants.AssetTypes.URI_FOLDER, mode="rw_mount"),
        },
        code="./src/",
        command=(
            "export PYTHONPATH=$PYTHONPATH:$(pwd) &&"
            " python electricity_price_forecasting/steps/eval_model.py"
            " --data-file=${{inputs.eval_data_parquet_file}}"
            " --model-dir=${{inputs.trained_model}}"
            " --artifact-dir=${{outputs.artifact_dir}}"
        ),
        environment=environment,
        resources=_get_resources(),
        is_deterministic=True,
    )


def get_training_pipeline_func(environment: str, name_prefix: str = "", compute: str = "aml-compute-cpu") -> Callable[..., entities.PipelineJob]:
    """Get multi-step training pipeline function."""
    load_data_component = get_load_data_component(environment, name_prefix)
    train_model_component = get_train_model_component(environment, name_prefix)
    eval_model_component = get_eval_model_component(environment, name_prefix)

    @dsl.pipeline(compute=compute, name=f"{name_prefix}epf_multistep_training_pipeline", description="EPF Multi-Step Training Pipeline")
    def epf_training_pipeline(
        public_data_root_dir: Input(type=constants.AssetTypes.URI_FOLDER, mode="ro_mount"),
        train_date_from: Input(type="string"),
        train_date_to: Input(type="string"),
        eval_date_from: Input(type="string"),
        eval_date_to: Input(type="string", optional=True),
    ):
        load_train_data_step = load_data_component(
            public_data_root_dir=public_data_root_dir,
            date_from=train_date_from,
            date_to=train_date_to,
        )
        load_eval_data_step = load_data_component(
            public_data_root_dir=public_data_root_dir,
            date_from=eval_date_from,
            date_to=eval_date_to,
            with_lookback_window=True,
        )

        train_model_step = train_model_component(
            train_data_parquet_file=load_train_data_step.outputs.data_parquet_file,
        )
        eval_model_step = eval_model_component(
            eval_data_parquet_file=load_eval_data_step.outputs.data_parquet_file,
            trained_model=train_model_step.outputs.trained_model,
        )

        return {
            "trained_model": train_model_step.outputs.trained_model,
            "training_artifacts": train_model_step.outputs.artifact_dir,
            "evaluation_artifacts": eval_model_step.outputs.artifact_dir,
        }

    return epf_training_pipeline
```
