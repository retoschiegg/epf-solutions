```python
def get_training_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get training component."""
    return entities.CommandComponent(
        name=f"{name_prefix}epf_training_command",
        inputs={
            "public_data_root_dir": Input(type=constants.AssetTypes.URI_FOLDER, mode="ro_mount"),
            "train_date_from": Input(type="string"),
            "train_date_to": Input(type="string"),
            "eval_date_from": Input(type="string"),
            "eval_date_to": Input(type="string", optional=True),
        },
        outputs={
            "trained_model": Output(type=constants.AssetTypes.MLFLOW_MODEL, mode="rw_mount"),
            "training_artifacts": Output(type=constants.AssetTypes.URI_FOLDER, mode="rw_mount"),
            "evaluation_artifacts": Output(type=constants.AssetTypes.URI_FOLDER, mode="rw_mount"),
        },
        code="./src/",
        command=(
            "export PYTHONPATH=$PYTHONPATH:$(pwd) &&"
            " python training_script.py"
            " --public-data-root-dir=${{inputs.public_data_root_dir}}"
            " --train-date-from=${{inputs.train_date_from}}"
            " --train-date-to=${{inputs.train_date_to}}"
            " --eval-date-from=${{inputs.eval_date_from}}"
            " $[[ --eval-date-to=${{inputs.eval_date_to}} ]]"
            " --model-dir=${{outputs.trained_model}}"
            " --train-artifact-dir=${{outputs.training_artifacts}}"
            " --eval-artifact-dir=${{outputs.evaluation_artifacts}}"
        ),
        environment=environment,
        resources=_get_resources(),
        is_deterministic=True,
    )
```
