# Stage 1: Install Python dependencies
FROM python:3.11-slim AS builder

# Set working directory
WORKDIR /app

# Install compiler (needed for some Python libs)
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements file
COPY requirements.txt .

# Install dependencies into user directory
RUN pip install --no-cache-dir --user -r requirements.txt


# Stage 2: Runtime container
FROM python:3.11-slim

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set working directory
WORKDIR /app

# Copy installed Python packages from builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy application code with correct ownership
COPY --chown=appuser:appuser main.py .

# Add Python local bin to PATH
ENV PATH=/home/appuser/.local/bin:$PATH

# Logging & performance improvements
ENV PYTHONUNBUFFERED=1        # log instantly
ENV PYTHONDONTWRITEBYTECODE=1 # don't write .pyc files

# Switch to non-root user
USER appuser

# Expose port 8000
EXPOSE 8000

# Health check for Docker/K8s
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import httpx; httpx.get('http://localhost:8000/health')" || exit 1

# Run FastAPI app with Uvicorn server
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
