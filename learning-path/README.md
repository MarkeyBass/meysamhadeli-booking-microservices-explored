# Learning path: booking-microservices

This folder is a **guided sequence** for learning a large microservices codebase without trying to memorize the entire repository. It assumes you are comfortable with C# basics and Git, but **not** with huge repos, microservices, or deep infrastructure.

## What you will (and will not) do

| In scope | Out of scope (for this path) |
|----------|------------------------------|
| Run the system **locally** (Aspire or Docker Compose) | Kubernetes operations and TLS/cert-manager |
| Follow **one vertical slice** at a time (API → handler → data/events) | Production hardening, full Event Sourcing theory |
| Use **observability** to confirm what you read in code | Running every optional container “just because” |
| Add a **small** feature and repeat the pattern | New microservices, large refactors |

## Order of reading

1. [00 — Mindset and navigation](./00-mindset-and-navigation.md)  
2. [01 — Environment and run](./01-environment-and-run.md)  
3. [02 — System map (one page)](./02-system-map-one-page.md)  
4. [03 — First flow: Create booking](./03-first-flow-booking-create.md)  
5. Run the flow (same doc).  
6. [04 — Observability loop](./04-observability-loop.md)  
7. [05 — Databases and data](./05-databases-and-data.md) when you need to verify persistence.  
8. [06 — Add a small feature](./06-add-a-small-feature.md)  
9. [07 — Second flow template](./07-second-flow-template.md) — copy and reuse for the next slice.

## Timeboxing (suggested)

| Session | Goal |
|---------|------|
| 1 | Finish 00–01; dashboard or gateway responds |
| 2 | Finish 02–03; you can name the main types in Create booking |
| 3 | Execute one Create booking; capture one trace |
| 4 | Land a tiny feature from 06 |

Adjust to your pace. The point is **depth on one thread**, not coverage of every service.

## Images and diagrams

- Pipeline overview: [assets/observability-pipeline.png](./assets/observability-pipeline.png) (also embedded in [04](./04-observability-loop.md)).  
- [assets/README.md](./assets/README.md) lists assets and how to add your own screenshots.

## Repo anchors (quick reference)

| Topic | Location |
|-------|-----------|
| Vertical slice example | [`src/Services/Booking/src/Booking/Booking/Features/CreatingBook/V1/CreateBooking.cs`](../src/Services/Booking/src/Booking/Booking/Features/CreatingBook/V1/CreateBooking.cs) |
| OTel registration | [`src/BuildingBlocks/OpenTelemetryCollector/Extensions.cs`](../src/BuildingBlocks/OpenTelemetryCollector/Extensions.cs) |
| Collector config | [`deployments/configs/otel-collector-config.yaml`](../deployments/configs/otel-collector-config.yaml) |
| HTTP API samples | [`booking.rest`](../booking.rest) (REST Client for VS Code) |
| Gateway (YARP) | [`src/ApiGateway/src/appsettings.json`](../src/ApiGateway/src/appsettings.json) |
| Aspire host | [`src/Aspire/src/AppHost/Program.cs`](../src/Aspire/src/AppHost/Program.cs) |

When you finish one full loop (03 → 04 → 06), **duplicate the loop** using [07](./07-second-flow-template.md) for another feature or service.
