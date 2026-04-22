# 01 — Environment and run

Goal: **one working local environment** where APIs respond and you can open the Aspire dashboard or hit the gateway.

## Quick start — pick **one** way to run

You only need **one** of these. Do not mix **Path A** with **Path D** (port clash).

### Option 1 — Everything via .NET (simplest for learning and prefered way for learning)

1. Install **.NET 10** so `dotnet --list-sdks` shows `10.0.x` (you used Homebrew; if commands still fail, open a **new terminal** so `PATH` picks up `/opt/homebrew/bin/dotnet`).
2. From the **repository root** (`booking-microservices/`, where `global.json` lives):

```bash
dotnet --list-sdks
dotnet run --project src/Aspire/src/AppHost/AppHost.csproj
```

3. Open **`http://localhost:18888`** (Aspire dashboard).  
4. `aspire run` is optional; if the shell says `command not found: aspire`, **ignore it** — `dotnet run` on the App Host is enough.

### Option 2 — Everything via Docker (no local `dotnet run` of the App Host)

1. HTTPS PFX on the host (see [HTTPS](#https-dev-certificate-first-time)); Compose mounts `~/.aspnet/https/aspnetapp.pfx`.
2. From the **repository root**:

```bash
docker compose -f ./deployments/docker-compose/docker-compose.yaml up -d --build
```

3. Check status (note the **same `-f` file`** — if you omit it, `ps` may show **no containers**):

```bash
docker compose -f ./deployments/docker-compose/docker-compose.yaml ps
```

Gateway (HTTPS): **`https://localhost:5000`** (see compose port mapping for `api-gateway`).

### Why `docker-compose ps` showed nothing

Usually one of: **no `up` was run**, wrong **working directory**, or **`ps` without the same `-f ...yaml`**. Always use the full `-f` path from the repo root, or `cd` to repo root and run the commands above exactly.

## Prerequisites

| Requirement | This repo |
|-------------|-----------|
| .NET SDK | Version pinned in [`global.json`](../global.json) at repository root (for example `10.0.103` with `rollForward`). Install a matching **.NET 10** SDK — see [Install .NET 10](#install-net-10-macos-and-others). |
| Docker | Required for **Docker Compose** — see **Path B** (full stack) and **Path D** (infrastructure only) below. |
| Aspire CLI | Optional. The README uses `aspire run`; if the command is missing, use **`dotnet run`** on the App Host (below). Install the CLI only if you want it—see [Microsoft’s Aspire documentation](https://learn.microsoft.com/dotnet/aspire/). |

### Install .NET 10 (macOS and others)

Pick **one** approach; then from the **repo root** run `dotnet --list-sdks` and confirm a **`10.0.x`** line appears.

**macOS — official installer (recommended)**  
Download and install the **.NET 10 SDK** `.pkg`: [https://dotnet.microsoft.com/download/dotnet/10.0](https://dotnet.microsoft.com/download/dotnet/10.0)

**macOS — Homebrew**  

```bash
brew install --cask dotnet-sdk
dotnet --list-sdks
```

**Any OS — install script**  

```bash
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 10.0
```

Follow the script output to put `DOTNET_ROOT` and `$DOTNET_ROOT` on your `PATH`. Details: [dotnet-install script](https://learn.microsoft.com/dotnet/core/tools/dotnet-install-script).

### Troubleshooting: “A compatible .NET SDK was not found”

If **every** `dotnet` command fails in this repo (including `dotnet dev-certs` and `dotnet run`), the usual cause is a **version mismatch** with [`global.json`](../global.json).

- This solution pins **.NET 10** (for example `10.0.103`). Projects target `net10.0`; you cannot substitute only an **.NET 9** SDK and expect a clean build.
- **Fix:** install [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) (same or newer patch than `global.json`, depending on `rollForward`). Then run `dotnet --list-sdks` and confirm `10.0.x` appears.
- **Do not** delete or rewrite `global.json` to 9.x unless you are intentionally porting the whole solution—out of scope for these tutorials.

Also, `dotnet dev-certs https -ep ... -p` requires a **password argument** (for example `-p password`), not `-p` alone.

## macOS

The tutorials are **OS-neutral** except where the toolchain differs. On **macOS** (Intel or Apple Silicon), pay attention to these points:

| Topic | What to do on macOS |
|--------|---------------------|
| Docker | Install [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/) (or another engine). Ensure it is running before Aspire or Compose. |
| HTTPS dev cert | Use the **macOS or Linux** block in the [README](../README.md) (`dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p <password>` then `dotnet dev-certs https --trust`). macOS will prompt **Keychain** trust—accept it so `https://localhost:5000` works. Replace `<password>` with a real password; it must match what Compose expects for the PFX (README uses `password` in examples). |
| `aspire` not found | Common if only the .NET SDK is installed. From the repo root run: `dotnet run --project src/Aspire/src/AppHost/AppHost.csproj` (same App Host as `aspire run`). |
| Paths | Examples use `~/.aspnet/https/`—on Mac that is `/Users/<you>/.aspnet/https/`. |

Everything else in this chapter (ports, gateway URLs, Grafana) is the same as on Windows or Linux.

## HTTPS dev certificate (first-time)

The main [README](../README.md) documents `dotnet dev-certs https` for Windows and macOS/Linux. Compose-based gateway and services often mount `~/.aspnet/https/aspnetapp.pfx` — follow the README so HTTPS endpoints do not fail silently.

For **Aspire / local `dotnet run`**, trusting the dev cert is usually enough:

```bash
dotnet dev-certs https --trust
```

### Troubleshooting: `ERR_SSL_PROTOCOL_ERROR` on `https://localhost:5000`

Browsers show this when the **TLS handshake fails** or the server is **not actually speaking HTTPS** on that port yet.

1. **Wait until the gateway is running** — The first `dotnet run` on the App Host can **build for several minutes**. Until **api-gateway** is up, nothing (or the wrong process) may be on port 5000. In the Aspire dashboard (`http://localhost:18888`), confirm **api-gateway** shows as running before opening the gateway URL.

2. **Trust the dev certificate** (from repo root, with .NET 10 active):

   ```bash
   dotnet dev-certs https --trust
   ```

   Approve the macOS Keychain prompt if it appears.

3. **Try HTTP first** — The gateway also listens on **HTTP** port **5001** (see [`launchSettings.json`](../src/ApiGateway/src/Properties/launchSettings.json): `https://localhost:5000;http://localhost:5001`). Open **`http://localhost:5001/`** in the browser. You should at least see the gateway app name text; [`UseHttpsRedirection`](../src/ApiGateway/src/Program.cs) may then send you to HTTPS—if that step fails, the problem is specifically HTTPS/cert on 5000.

4. **Port conflict** — Another app (or a stuck process) might be bound to **5000** with plain HTTP. From a terminal: `lsof -nP -iTCP:5000 -sTCP:LISTEN` — you want to see your **ApiGateway** / `dotnet` process, not Docker or something else. Stop conflicting stacks (e.g. a failed **Docker Compose** run) if they grabbed the port.

5. **Docker Compose build failed** — If `docker compose up --build` failed (for example gRPC `protoc` **exit code 139** on Apple Silicon), **no** gateway container is running; use **Aspire** for the gateway or fix the Docker build separately. An empty `docker compose … ps` means the stack never came up.

## Path A — Aspire (recommended for learning)

From the **repository root**, either:

```bash
aspire run
```

If `aspire` is not installed, use:

```bash
dotnet run --project src/Aspire/src/AppHost/AppHost.csproj
```

- **Aspire dashboard**: `http://localhost:18888` (see [`src/Aspire/src/AppHost/Properties/launchSettings.json`](../src/Aspire/src/AppHost/Properties/launchSettings.json), `applicationUrl`).  
- The App Host starts infrastructure and projects defined in [`src/Aspire/src/AppHost/Program.cs`](../src/Aspire/src/AppHost/Program.cs): Postgres, Mongo, Redis, EventStore, RabbitMQ, Elasticsearch, observability containers, then **identity**, **passenger**, **flight**, **booking**, **api-gateway** with fixed ports.

**Does `aspire run` / `dotnet run` on the App Host already start databases and brokers?**  
**Yes.** You do **not** need a separate “infrastructure only” Compose file for that workflow. Adding [`docker-compose.infrastructure.yaml`](../deployments/docker-compose/docker-compose.infrastructure.yaml) **at the same time** would try to bind the same ports (5432, 2113, 4317, 3000, …) and **conflict**.

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

## Path D — Infrastructure-only Docker (not with full Aspire)

Use this when you want **Postgres, EventStore, RabbitMQ, Mongo, Redis, Elasticsearch, Kibana, and the observability stack** in Docker, but you plan to run the **.NET APIs yourself** (for example `dotnet run` on each `*.Api` from the IDE, or debugging one service). This file is [`deployments/docker-compose/docker-compose.infrastructure.yaml`](../deployments/docker-compose/docker-compose.infrastructure.yaml) (project name `booking-microservices-infrastructure`).

From the **repository root**:

```bash
docker compose -f ./deployments/docker-compose/docker-compose.infrastructure.yaml up -d
```

Rebuild images if you change compose files (here everything is `image:`, not local build—so usually no `--build` needed):

```bash
docker compose -f ./deployments/docker-compose/docker-compose.infrastructure.yaml up -d --pull always
```

Stop and remove containers (data volumes may remain):

```bash
docker compose -f ./deployments/docker-compose/docker-compose.infrastructure.yaml down
```

**Do not** start Path D and **also** run the **Aspire App Host** (Path A) on the same machine: both provide the same infrastructure ports.

**Workflow summary**

| Goal | What to run |
|------|-------------|
| One command: infra + all APIs + dashboard | **Path A only** — `dotnet run --project src/Aspire/src/AppHost/AppHost.csproj` (or `aspire run`). |
| Everything in Docker including APIs | **Path B** — full [`docker-compose.yaml`](../deployments/docker-compose/docker-compose.yaml). |
| Docker infra only, APIs from `dotnet run` / IDE | **Path D** + run each `*.Api` with `appsettings` pointing at `localhost` (same defaults as repo). **Do not** start the App Host. |

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
