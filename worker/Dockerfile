# Stage 1: Build
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2: Final Runtime
FROM python:3.12-slim
WORKDIR /app

# Non-root user
RUN adduser --disabled-password --gecos "" hnguser
COPY --from=builder /install /usr/local
COPY . .
RUN chown -R hnguser:hnguser /app

USER hnguser

# Healthcheck: Since worker isn't a web server, we check if the process is alive
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD pgrep -f "python worker.py" || exit 1

CMD ["python", "worker.py"]