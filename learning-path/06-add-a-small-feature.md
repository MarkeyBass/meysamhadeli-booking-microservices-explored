# 06 — Add a small feature

You have read one slice and seen it in telemetry. Now **change** something small so muscle memory forms: edit → build → run → verify → trace.

## Rules of thumb

- **Touch one vertical slice** folder (same pattern as `Features/CreatingBook/V1`).  
- **Prefer validation or mapping** before you change domain invariants or EventStore schemas.  
- **Always** run `dotnet build` on the solution or at least the affected service test project.  
- **Re-run** the observability loop from [04](./04-observability-loop.md) and confirm a new span or log line reflects your change.

## Suggested first features (pick one)

### A) Stronger validation on Create booking

In [`CreateBookingValidator`](../src/Services/Booking/src/Booking/Booking/Features/CreatingBook/V1/CreateBooking.cs), add a rule on `Description` (max length, not empty, forbidden characters).  

**Verify**: call the API with an invalid body → `400` with a clear validation problem.  

**Trace**: handler should not run for failed validation; you may see shorter spans — that is expected.

### B) Response metadata

Add a custom header on the Create booking endpoint (for example `X-Booking-Version: 1`) in the same endpoint class.  

**Verify**: inspect response headers in REST Client or curl.  

**Trace**: unchanged business logic; use this if you want a gentler first edit.

### C) Logging breadcrumb

Add a **structured** log line at the start of `CreateBookingCommandHandler` with `BookingId` and `FlightId` (no secrets).  

**Verify**: see the line in Aspire logs or Loki Explore in Grafana.

## Checklist (copy into your notes)

- [ ] I know which **files** I will edit before I start.  
- [ ] I ran **tests** (`dotnet test` filtered to Booking if available) or added a small test if the repo pattern is clear.  
- [ ] I called the API and got the expected **status code**.  
- [ ] I saw **telemetry** (log or span) that proves the new path executed.  
- [ ] I can **explain** the change in one sentence to another developer.

## What to avoid as “first feature”

- New microservice or new database  
- Changing JWT policies or Identity clients without reading Duende docs  
- Large cross-service refactors

## Screenshot placeholder

<!-- ![Tests green](./assets/dotnet-test-booking.png) -->

## Next step

Copy [07 — Second flow template](./07-second-flow-template.md) to your notes repo or duplicate the file for a **second** slice (Flight create, Passenger register, and so on).
