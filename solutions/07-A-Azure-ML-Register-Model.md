**training_script.py**

The added lines should look similar to:

```python
import logging
import mlflow

_LOGGER = logging.getLogger(__name__)

def training(
    # ... existing parameters
    model_name: str = "",
) -> None:
    """Train and evaluate the model."""
    # ... existing code

    if model_name:
        result = mlflow.register_model(f"file://{model_dir}", model_name)
        _LOGGER.info("Registered %s with version %s", model_name, result.version)

@click.option("--model-name", type=str, help="Name of the model to register", default="")
def cli(
    # ... existing parameters
    model_name: str,
) -> None:
    """Entrypoint."""
    training(
        # ... existing arguments
        model_name,
    )
```


**azure_training.py**

The added lines should look similar to:

```python
def get_training_component(environment: str, name_prefix: str = "") -> entities.CommandComponent:
    """Get training component."""
    return entities.CommandComponent(
        # ... existing arguments
        inputs={
            # ... existing inputs
            "model_name": Input(type="string", optional=True),
        },
        command=(
            # ... existing command
            " $[[ --model-name=${{inputs.model_name}} ]]"
        ),
    )


def get_training_pipeline_func(
    environment: str, name_prefix: str = "", compute: str = "aml-compute-cpu"
) -> Callable[..., entities.PipelineJob]:
    """Get training pipeline function."""
    training_component = get_training_component(environment, name_prefix=name_prefix)

    @dsl.pipeline(compute=compute, name=f"{name_prefix}epf_training_pipeline", description="EPF Training Pipeline")
    def epf_training_pipeline(
        # ... existing parameters
        model_name: Input(type="string", optional=True),
    ):
        training_step = training_component(
            # ... existing arguments
            model_name=model_name,
        )
```
