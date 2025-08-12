# Grafana + InfluxDB (Docker Compose)

Two separate containers for InfluxDB 2.x and Grafana, with a pre-provisioned Grafana datasource that uses Flux.

## Quick start

1) Copy `.env.example` to `.env` and optionally change credentials/tokens:

```bash
cp .env.example .env
# then edit .env
```

2) Launch:

```bash
docker compose up -d
```

3) URLs:
- InfluxDB UI: http://localhost:8086
- Grafana UI:  http://localhost:3000  (login with `GF_SECURITY_ADMIN_USER` / `GF_SECURITY_ADMIN_PASSWORD`)

The Grafana datasource to InfluxDB is auto-provisioned on first start.

## Notes

- InfluxDB is initialized in setup mode using the values from `.env`.
- Data is persisted in named Docker volumes (`influxdb-data`, `influxdb-config`, `grafana-data`).
- Healthchecks ensure Grafana starts only after InfluxDB is ready.
- To reset everything, stop the stack and remove volumes:
  ```bash
  docker compose down -v
  ```

## Writing data

If you don’t already have writers, you can send a quick test point:

```bash
docker exec -it influxdb influx write   --bucket $DOCKER_INFLUXDB_INIT_BUCKET   --org $DOCKER_INFLUXDB_INIT_ORG   --token $DOCKER_INFLUXDB_INIT_ADMIN_TOKEN   --precision s   "test,host=local value=42 $(date +%s)"
```

Then in Grafana, create a panel and run a simple Flux query:

```flux
from(bucket: v.defaultBucket)
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "test")
```

## Switching to InfluxQL

If you need InfluxQL (v1) compatibility, enable the InfluxDB 2.x v1 compatibility API and use an InfluxQL-type datasource in Grafana. For most new projects, Flux is recommended.
