**1) What is the difference between a stage, a job, and a step?**

- A **stage** is the highest-level grouping in a pipeline. Stages typically represent major phases such as Build, Test, or Deploy, and they can be run sequentially or with dependencies/approvals.
- A **job** is a unit of work inside a stage. Each job runs on an agent (VM) and has its own workspace and execution context. Jobs within the same stage can run in parallel unless dependencies are defined.
- A **step** is an individual action inside a job, such as running a task (e.g., `AzureCLI@2`), checking out code, or executing a script (`bash`).


**2) What is `ARM_SERVICE_CONNECTION` used for?**

In this pipeline, the variable `ARM_SERVICE_CONNECTION` contains the name of an Azure DevOps service connection that authenticates the pipeline to the Azure subscription.

Azure Resource Manager (ARM) is Azure's control plane API layer: it's the management endpoint used to create, update, and query Azure resources and to enforce permissions via Azure AD and RBAC. Azure DevOps uses an `Azure Resource Manager` service connection type to establish this trusted link between Azure DevOps and Azure using a service principal or workload identity.


**3) What does `- checkout: self` do?**

Checks out the repository that contains the pipeline into the agent workspace. Without this step, the agent won't have the source files (for example, `Dockerfile`, `pyproject.toml`, `uv.lock`) needed by subsequent steps, including the Docker build.


**4) What does the Bash variable `DEPS_HASH` do, and why is it included? Under what circumstances could this be dangerous?**

The idea is:
- If dependency-related inputs don't change, the Docker image tag stays the same
- If they change, a new Docker image tag is generated automatically

You will see the impact in the next notebook, where the ML environment is created by referencing the Docker image. When a new Docker image is referenced, the ML environment version is incremented.

When could this be dangerous?

If the `Dockerfile` is changed to include additional source files, but those files are not included in the hash computation, the following can happen: the newly added source files change, the resulting Docker image changes, but the computed hash stays the same. In that case, the pipeline would overwrite an existing image tag with different content (same tag, new image). This would be fatal if that tag were referenced in a production environment, because it breaks traceability and can lead to unintended, silent changes in deployed workloads.


**5) Why is there a Docker image with the tag `cache`?**

The `cache` tag exists to enable Docker layer caching across pipeline runs. The build can reuse layers from the existing `cache` image and only rebuild layers that have changed. This speeds up the CI pipeline and typically reduces compute and network usage.


**6) Why is `##vso[task.setvariable ...]` included?**

`##vso[task.setvariable ...]` is a special logging command that tells Azure Pipelines to create or update a pipeline variable from within a script step. This lets you pass values computed at runtime (for example, the fully qualified Docker image reference) to later steps in the same job.

If you set `isOutput=true`, the variable becomes an **output** variable, meaning it can also be consumed by other jobs in other stages.
