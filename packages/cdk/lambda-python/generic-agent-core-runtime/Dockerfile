# Use ARM64 base image for Bedrock AgentCore compatibility
FROM --platform=linux/arm64 public.ecr.aws/docker/library/python:3.13

WORKDIR /var/task

# Environment variables for UV package manager
ENV UV_NO_CACHE=1
ENV UV_NO_MANAGED_PYTHON=1
ENV UV_PYTHON=/usr/local/bin/python
ENV UV_INSTALL_DIR=/usr/local/bin
ENV UV_PROJECT_ENVIRONMENT=/tmp/.venv

# Install system dependencies
RUN apt-get update -y && apt-get install curl nodejs npm graphviz -y
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# Pre-install mcp-remote for faster startup
RUN npm install -g mcp-remote

# Copy dependency files
COPY pyproject.toml .python-version uv.lock ./
# Install Python dependencies
RUN uv sync

# Copy application files
COPY app.py ./
COPY src/ ./src/
COPY mcp-configs ./mcp-configs

# --- SDPM: spec-driven-presentation-maker ---
COPY sdpm-skill/ ./sdpm-skill/
COPY sdpm-mcp-local/ ./sdpm-mcp-local/
COPY sdpm-shared/ ./sdpm-shared/
RUN uv pip install --python /tmp/.venv/bin/python ./sdpm-skill
RUN /tmp/.venv/bin/python sdpm-skill/scripts/download_aws_icons.py \
 && /tmp/.venv/bin/python sdpm-skill/scripts/download_material_icons.py
RUN ln -s /var/task/sdpm-skill /var/task/skill
RUN ln -s /var/task/sdpm-shared /var/task/shared

# LibreOffice (Impress) + poppler + 日本語フォント: sdpm の measure_slides / preview 用
RUN apt-get update && apt-get install -y --no-install-recommends \
      libreoffice-impress \
      poppler-utils \
      fonts-noto-cjk \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# Expose port 8080 as required by AgentCore
EXPOSE 8080

# Start the application with OpenTelemetry instrumentation
CMD ["uv", "run", "opentelemetry-instrument", "python", "app.py"]
