# Owl and Starling push OpenTelemetry traces, metrics and logs every second and
# log a warning on each failed push. Nothing in JamAI Base reads them back
# (usage meters come from ClickHouse), so this collector accepts and drops them.
# Swap the exporters in config.yaml to ship them somewhere useful.
FROM otel/opentelemetry-collector:0.113.0
COPY config.yaml /etc/otelcol/config.yaml
