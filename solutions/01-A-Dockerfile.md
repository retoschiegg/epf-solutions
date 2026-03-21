
```Dockerfile
# Base image with Debian and Python 3.13 installed
FROM python:3.13.11-slim-bookworm

# Installation of curl to install `uv`
RUN apt-get update && \
   apt-get install -y --no-install-recommends \
   curl \
   && rm -rf /var/lib/apt/lists/*

# Set environment variables and working directory
ENV APP_DIR="/app"
WORKDIR $APP_DIR
# Set path of virtual environment installed by `uv`
ENV VIRTUAL_ENV="$APP_DIR/.venv"
# Add executables to PATH variable for `uv` and `python` from the virtual environment
ENV PATH="$VIRTUAL_ENV/bin:/root/.local/bin:$PATH"

# Installation of `uv`
ARG UV_VERSION=0.9.26
ENV UV_NO_CACHE=1
ENV UV_PROJECT_ENVIRONMENT=$VIRTUAL_ENV
RUN curl -Lsf --proto '=https' https://astral.sh/uv/${UV_VERSION}/install.sh | sh

# Initialize virtual environment with packages
COPY pyproject.toml uv.lock ./
RUN uv sync --no-group dev --no-install-project
```

**Good points**
- small (slim) base image
- referencing specific versions for base image and uv
- uv does not use cache when installing packages (no duplicated files in the image)
- only copying necessary files (not adding all files)
- no development dependencies installed in the image (using `--no-group dev` flag)

**Possible improvements**
- Currently runs with root user
