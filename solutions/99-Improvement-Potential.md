## Improvement potential towards a production-grade MLOps application

The example application covers the main MLOps concepts well, but there is still clear potential for improvement towards a production-grade application.

**Components and versioning**

- Use isolated, versioned components instead of uploading the full `src` directory.
- This improves traceability, independence, versioning, and rollback handling.
- *Not as an improvement, but as an alternative implementation approach: ML components and pipelines could also be defined in YAML instead of Python code.*

**Deployment repository**

- Either maintain it as a full standalone repository with proper structure.
- Or keep it free of application code. In that case, reference a CI pipeline from the application repository to run the integration tests.

**CI/CD, release process and automation**

- Define a clear Git branching strategy.
- Extend the CI pipelines to feature branches.
- Use Git tags for releases.
- Retag the Docker image with the released version.
- Use [semantic commits](https://www.conventionalcommits.org).
- Include execution of unit tests.
- Introduce a separate parameterized model release pipeline.
- Use [Renovate](https://docs.renovatebot.com) for dependency updates.
- Automate build, test, and deployment tasks, including merge requests towards the deployment repository.
- *Not as an improvement, but as an alternative implementation approach: The Azure CLI could be used instead of executing Python scripts in the CI pipeline.*

**Monitoring and operations**

- Alerts on failed pipeline jobs.
- Detect missing incoming data.
- Add monitoring for data/model quality and drift.
