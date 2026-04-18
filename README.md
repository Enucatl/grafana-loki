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
| Log retention | 2 months |
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

## Dashboards

Grafana dashboards are provisioned automatically via the `/etc/grafana/provisioning/dashboards/provider.yaml` configuration. External dashboards can be mounted by adding volume entries in the `docker-compose.yml` file. For example, the **crabberbot** dashboard is mounted from `../crabberbot/grafana-dashboard.json` and will be available in Grafana after startup.

To add additional dashboards:
1. Mount the dashboard JSON file as a volume in the `grafana` service
2. Target a path under `/var/lib/grafana/dashboards/`
3. Restart the stack: `docker compose up -d`

## Directory Structure

```
.
├── docker-compose.yml
├── grafana/
│   ├── grafana.ini          # Grafana server config
│   ├── datasources.yaml     # Provisioned Loki datasource
│   ├── dashboards/
│   │   └── provider.yaml    # Dashboard provisioning config
│   └── roles.yaml           # Access control roles
└── loki/
    └── loki-config.yml      # Loki server config
```

## Security baseline

This compose project uses the shared [docker-compose-security-baseline](https://github.com/Enucatl/docker-compose-security-baseline) for common container hardening defaults, including capabilities, no-new-privileges, memory/swap, and PID limits.
