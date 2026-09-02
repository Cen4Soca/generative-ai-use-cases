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
# Layout follows SDPM v0.5.0+ (skill/ -> sdpm/, mcp-local/ -> servers/local/).
# The engine resolves bundled data from SDPM_SKILL_ROOT, and personas/ must sit
# next to it (SKILL_ROOT.parent/personas), hence the /var/task/personas target.
COPY sdpm-skill/ ./sdpm-skill/
COPY sdpm-servers-local/ ./sdpm-servers-local/
COPY sdpm-personas/ ./personas/
COPY sdpm-shared/ ./shared/
RUN uv pip install --python /tmp/.venv/bin/python ./sdpm-skill
# Document conversion deps required by read_attachment / import_attachment
# (declared in servers/local, not in the sdpm engine package)
RUN uv pip install --python /tmp/.venv/bin/python \
      "pdfplumber>=0.11.0" "pypdf>=6.15.0" "python-docx>=1.0.0" "openpyxl>=3.1.0"
RUN /tmp/.venv/bin/python sdpm-skill/scripts/download_aws_icons.py \
 && /tmp/.venv/bin/python sdpm-skill/scripts/download_material_icons.py
RUN ln -s /var/task/sdpm-skill /var/task/sdpm

# LibreOffice (Impress) + poppler + 日本語フォント: sdpm の run_python
# (preview 生成 / テキスト計測) 用
RUN apt-get update && apt-get install -y --no-install-recommends \
      libreoffice-impress \
      poppler-utils \
      fonts-noto-cjk \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# Expose port 8080 as required by AgentCore
EXPOSE 8080

# Start the application with OpenTelemetry instrumentation
CMD ["uv", "run", "opentelemetry-instrument", "python", "app.py"]
