# Usage meters (tokens, storage, egress) for JamAI Base: stock ClickHouse at
# the version upstream compose pins, with upstream's table script.
FROM clickhouse/clickhouse-server:24.10.2.80-alpine

COPY create_tables.sh /docker-entrypoint-initdb.d/create_tables.sh
COPY users.xml /etc/clickhouse-server/users.d/jamai.xml
COPY railway.xml /etc/clickhouse-server/config.d/railway.xml
# The stock file listens on :: and 0.0.0.0; on Railway :: is already
# dual-stack, so the second bind fails. Overwritten, not deleted.
COPY listen.xml /etc/clickhouse-server/config.d/docker_related_config.xml

# Railway hosts run transparent huge pages "always", which makes idle
# ClickHouse memory creep; thp:never keeps it flat.
ENV CLICKHOUSE_DB=jamaibase_owl \
    CLICKHOUSE_USER=owluser \
    CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1 \
    CLICKHOUSE_ALWAYS_RUN_INITDB_SCRIPTS=1 \
    MALLOC_CONF=thp:never
