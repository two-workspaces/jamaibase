# Owl, JamAI Base's API, plus its Celery worker (Starling runs this image with
# another start command). Upstream publishes no images for v0.5, so this builds
# docker/Dockerfile.owl from a pinned commit of main. Bump JAMAI_COMMIT to upgrade.
FROM ghcr.io/embeddedllm/jamaibase/owl.base@sha256:1cee40eca8c4fbbe9da473f9af48da2fbdefc3f9020e61788cb57a35ec83da90

ARG JAMAI_COMMIT=834459fcfca7458c03841c40ea1f7b80cc373e21
WORKDIR /usr/src/app
RUN git init -q /tmp/src && cd /tmp/src \
    && git fetch -q --depth 1 https://github.com/EmbeddedLLM/JamAIBase.git "$JAMAI_COMMIT" \
    && git checkout -q FETCH_HEAD \
    && mv services/api /usr/src/app/api && mv clients/python /usr/src/app/client \
    && rm -rf /tmp/src
RUN uv venv --python 3.12 \
    && cd api && uv pip install --no-cache-dir -e . \
    && cd ../client && uv pip install --no-cache-dir -e .

ENV OWL_PORT=6969 \
    OTEL_PYTHON_FASTAPI_EXCLUDED_URLS=api/health \
    OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_REQUEST='X-.*' \
    OTEL_EXPORTER_OTLP_METRICS_DEFAULT_HISTOGRAM_AGGREGATION=base2_exponential_bucket_histogram \
    SKYPILOT_DISABLE_USAGE_COLLECTION=1

# Owl creates its schema on first boot, so it waits for Postgres and ClickHouse
# instead of crash-looping while they start.
CMD ["/bin/bash", "-c", "until (: </dev/tcp/$PGHOST/5432) 2>/dev/null && curl -sf http://$OWL_CLICKHOUSE_HOST:8123/ping >/dev/null; do echo Waiting for Postgres and ClickHouse; sleep 2; done; exec uv run python -m owl.entrypoints.api"]
