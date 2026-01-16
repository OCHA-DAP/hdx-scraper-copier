# ----------------------------------------------------------------------------
# Stage 1: Builder
# ----------------------------------------------------------------------------
FROM public.ecr.aws/unocha/python:3.13-stable AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

WORKDIR /app

# 1. Install Basic Build Deps
#    We only need 'git' for setuptools-scm versioning.
#    'build-base' (compilers) is only needed if a pip package lacks Alpine wheels.
RUN --mount=type=cache,target=/var/cache/apk \
    apk add --upgrade git

# 2. Install Dependencies
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project --compile

# 3. Install Project
COPY . .
RUN uv pip install --no-deps .

# ----------------------------------------------------------------------------
# Stage 2: Final Runtime
# ----------------------------------------------------------------------------
FROM public.ecr.aws/unocha/python:3.13-stable

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# 1. Copy the Virtual Environment
ENV VIRTUAL_ENV=/app/.venv
COPY --from=builder /app/.venv ${VIRTUAL_ENV}
ENV PATH="${VIRTUAL_ENV}/bin:${PATH}"

# 2. Copy the runner script
COPY run.py .

CMD ["python3", "run.py"]