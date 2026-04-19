# 01 — Environment and run

Goal: **one working local environment** where APIs respond and you can open the Aspire dashboard or hit the gateway.

## Prerequisites

| Requirement | This repo |
|-------------|-----------|
| .NET SDK | Version pinned in [`global.json`](../global.json) at repository root (for example `10.0.103` with `rollForward`). Install a matching **.NET 10** SDK. |
| Docker | Required for **Docker Compose** (containers for Postgres, Mongo, RabbitMQ, EventStore, observability stack, etc.). |
| Aspire CLI | Required if you choose the **Aspire** path (`aspire run`). Install per [Microsoft’s Aspire documentation](https://learn.microsoft.com/dotnet/aspire/). |

## HTTPS dev certificate (first-time)

The main [README](../README.md) documents `dotnet dev-certs https` for Windows and macOS/Linux. Compose-based gateway and services often mount `~/.aspnet/https/aspnetapp.pfx` — follow the README so HTTPS endpoints do not fail silently.

## Path A — Aspire (recommended for learning)

From the **repository root**:

```bash
aspire run
```

- **Aspire dashboard**: `http://localhost:18888` (see [`src/Aspire/src/AppHost/Properties/launchSettings.json`](../src/Aspire/src/AppHost/Properties/launchSettings.json), `applicationUrl`).  
- The App Host starts infrastructure and projects defined in [`src/Aspire/src/AppHost/Program.cs`](../src/Aspire/src/AppHost/Program.cs): Postgres, Mongo, Redis, EventStore, RabbitMQ, Elasticsearch, observability containers, then **identity**, **passenger**, **flight**, **booking**, **api-gateway** with fixed ports.

### Aspire — ports you will actually use

These come from `Program.cs` (proxied endpoints on the host):

| Service | HTTP | HTTPS (typical) |
|---------|------|-----------------|
| API Gateway | 5001 | 5000 |
| Flight | 5004 | 5003 |
| Passenger | 6012 | 5012 |
| Booking | 6010 | 5010 |
| Identity | 6005 | 5005 |
| Grafana | 3000 | — |
| Prometheus | 9090 | — |
| Jaeger UI | 16686 | — |
| Zipkin UI | 9411 | — |
| OpenTelemetry Collector OTLP | 4317 (gRPC), 4318 (HTTP) | — |
| RabbitMQ management | 15672 | — |
| EventStore | 2113 | — |

Grafana in Aspire is provisioned with **admin / admin** (see `Program.cs` environment variables).

### Aspire — OTLP note

The dashboard profile sets `ASPIRE_DASHBOARD_OTLP_ENDPOINT_URL` to the **otel-collector** inside the mesh. Your apps still export OTLP as configured in [`src/BuildingBlocks/OpenTelemetryCollector/Extensions.cs`](../src/BuildingBlocks/OpenTelemetryCollector/Extensions.cs); traces and metrics also show up in the Aspire UI for quick feedback.

## Path B — Docker Compose

From the **repository root**:

```bash
docker compose -f ./deployments/docker-compose/docker-compose.yaml up -d
```

This builds and runs API containers plus dependencies. You still need the **dev certificate** mounted at `~/.aspnet/https` as in the compose file for HTTPS.

### Compose — observability URLs (host)

| Tool | URL |
|------|-----|
| Grafana | `http://localhost:3000` (admin / admin per compose env) |
| Prometheus | `http://localhost:9090` |
| Jaeger | `http://localhost:16686` |
| Zipkin | `http://localhost:9411` |
| Loki | `http://localhost:3100` |
| OTLP (collector) | `4317`, `4318` |

Tempo’s published port in compose may vary; run `docker compose ps` and look for the **tempo** service if you need the exact host port for Tempo’s HTTP API.

## OpenTelemetry endpoints (FYI)

Service `appsettings.json` files include `ObservabilityOptions` with `OTLPGrpExporterEndpoint` (often `http://localhost:4317` for the collector) and sometimes `AspireDashboardOTLPOptions` for dashboard-specific ports. When you run under **Aspire**, environment overrides may point exports at the orchestrated collector. You do not need to tune this for the learning path; use it only if traces do not appear ([04](./04-observability-loop.md)).

## Path C — Run a single API project (advanced / partial)

The README describes `dotnet run` from each service’s `*.Api` folder. That path is **partial**: you must already have dependencies (Postgres, EventStore, RabbitMQ, …) running via Compose or Aspire and **matching** connection strings in `appsettings.Development.json` or user secrets. Prefer Path A or B until you know which ports your machine uses.

## “Something is up” checklist

- [ ] Gateway or at least one service responds on the expected port.  
- [ ] Identity is reachable (for token acquisition in [03](./03-first-flow-booking-create.md)).  
- [ ] Aspire dashboard **or** Grafana loads in the browser.

## Screenshot placeholder

Paste a screenshot of your working Aspire dashboard or first successful service response:

<!-- Your screenshot: ![Aspire dashboard](./assets/aspire-dashboard.png) -->

## Next step

[02 — System map (one page)](./02-system-map-one-page.md)
