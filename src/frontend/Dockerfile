FROM python:3.13-alpine AS builder

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# added these two lines to counter the failure due to symlink to python in .venv
# prevents download of python to /root
# otherwise error "exec /app/.venv/bin/study-tracker-api: no such file or directory"
# because python binary was in container1 but never copied to container2
# uv sync silently copies .venv and also quietly stores the python interpreter outside in /root/.local/share/uv/python
# false reliance on uv sync creating self-contained package with dependencies present
ENV UV_PYTHON_DOWNLOADS=never
ENV UV_NO_MANAGED_PYTHON=1

# Change the working directory to the `app` directory
WORKDIR /app

# Install dependencies in a cache layer. This speeds up build time
RUN --mount=type=cache,target=/root/.cache/uv \
  --mount=type=bind,source=uv.lock,target=uv.lock \
  --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
  uv sync --locked --no-install-project --no-editable

# Copy the project into the intermediate image
COPY . /app

# Sync the project and install it, now that we have access to the source code
RUN --mount=type=cache,target=/root/.cache/uv \
  uv sync --locked --no-editable --no-dev

FROM python:3.13-alpine

# Create a non-root user and group with specific IDs
RUN addgroup -S -g 1000 app && adduser -S -u 1000 -G app app

# Copy the environment from builder
COPY --from=builder --chown=app:app /app/.venv /app/.venv

# Switch to the non-root user
USER app

# Expose the correct port
EXPOSE 22111

# Run the application using the compiled app command
CMD ["/app/.venv/bin/study-tracker-frontend"]
