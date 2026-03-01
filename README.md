# grafana-loki

Docker Compose stack for centralized log aggregation using Grafana and Loki.

## Stack

- **Loki** — log aggregation backend, exposed internally on port 3100
- **Grafana** — visualization frontend, exposed internally on port 3000

Both services are fronted by **Traefik** (external network) with TLS termination, and Grafana is additionally protected by **Authelia** for SSO. Grafana is also attached to the **crabberbot** network to collect stats from that service.

## Key Configuration

| Setting | Value |
|---|---|
| Loki schema | TSDB v13 (from 2025-12-01) |
| Log retention | 6 months |
| Ingestion rate | 100 MB/s (burst 200 MB/s) |
| gRPC max message size | 100 MB |
| Structured metadata | enabled |
| Analytics reporting | disabled |

Logs are stored on a named Docker volume (`loki_data`) using the local filesystem backend.

## Deployment

Copy `.env.example` to `.env` and set `DOCKER_DOMAIN`, then:

```sh
docker compose up -d
```

Grafana will be available at `https://grafana.<DOCKER_DOMAIN>` and Loki at `https://loki.<DOCKER_DOMAIN>`. Loki is pre-configured as the default datasource in Grafana via provisioning.

## Directory Structure

```
.
├── docker-compose.yml
├── grafana/
│   ├── grafana.ini          # Grafana server config
│   ├── datasources.yaml     # Provisioned Loki datasource
│   └── roles.yaml           # Access control roles
└── loki/
    └── loki-config.yml      # Loki server config
```
