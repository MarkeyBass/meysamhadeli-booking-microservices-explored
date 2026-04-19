# 07 — Second flow template

Use this as a **repeatable worksheet**. Duplicate the section for each new flow you learn.

---

## Flow name

<!-- e.g. Reserve seat, Complete passenger registration -->

## Entry points

| Type | Value |
|------|--------|
| HTTP method and path (gateway) | |
| HTTP method and path (direct service) | |
| Sample in `booking.rest` | |

## Authorization

| Audience / scope needed | How to obtain token |
|-------------------------|---------------------|
| | |

## Code map (top-down)

| Order | File path | Responsibility |
|-------|-----------|----------------|
| 1 | | Endpoint / route |
| 2 | | Validator or request DTO |
| 3 | | Handler / consumer |
| 4 | | External dependency (gRPC, DB, bus) |

## Run checklist

- [ ] Dependencies running ([01](./01-environment-and-run.md))  
- [ ] Token acquired with correct **scope**  
- [ ] Request succeeds (or fails in an **expected** way)

## Observability checklist ([04](./04-observability-loop.md))

- [ ] Saw trace in **Aspire** or **Tempo/Jaeger**  
- [ ] Named at least **three** spans and mapped them to code  
- [ ] Optional: Prometheus query that moves when you load-test

### Screenshot slots

<!-- ![Flow2 trace](./assets/flow2-trace.png) -->

## Data persistence ([05](./05-databases-and-data.md))

| Store | What I checked |
|-------|----------------|
| | |

## Mini feature for this flow

**Idea**:

**Files touched**:

**Proof it works**:

---

## After two flows

You now have a **personal algorithm**:

1. Narrow scope  
2. Read slice  
3. Run  
4. Observe  
5. Optionally persist-check  
6. Small feature  
7. Duplicate this template

Return to the [learning-path README](./README.md) if you want to adjust scope or timeboxing.
