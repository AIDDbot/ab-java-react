---
slug: health-check
tier: back
status: pending
---
# Implementation Plan — health-check — back

## Specification

Expose `GET /api/health` from the Spring Boot API. On each request the service probes database connectivity, computes process uptime from the application start time captured at boot, stamps the current server time, persists an append-only `HealthCheck` record, and returns the vitals as JSON. A reachable database yields `200 OK` with `status`/`database` = `UP`; a failed probe yields `503 Service Unavailable` with `status`/`database` = `DOWN` and never leaks a stack trace. Layering follows `HealthController` → `HealthService` → `HealthCheckRepository` → `HealthCheck`, per `docs/arch/back.arch.md`.

### Data model

`HealthCheck` JPA entity mapped to `HEALTH_CHECK`: `id` (identity), `status` (`UP`/`DOWN`), `databaseStatus` (`UP`/`DOWN`), `uptimeSeconds` (long, `>= 0`), `checkedAt` (ISO-8601 UTC). Append-only; `status` = `UP` only when `databaseStatus` = `UP`. The SQLite schema is owned by this tier: JPA (`ddl-auto: update`) materializes the table from the entity mapping — there is no separate DB/migration tier.

Response contract:

```json
{
  "status": "UP",
  "database": "UP",
  "uptime": { "seconds": 3725, "since": "2026-05-29T10:00:00Z" },
  "timestamp": "2026-05-29T11:02:05Z"
}
```

## Implementation Steps

### Step 1: Configure the SQLite datasource, dialect, and schema management

Point the API at the embedded SQLite file and let JPA own the schema, so the `HEALTH_CHECK` table (including `uptime_seconds`) is materialized from the entity mapping.

- Paths involved:
  - `back/src/main/resources/application.yml`
- [ ] Set the JDBC URL to the embedded file `data/app.db`, configure the `org.xerial:sqlite-jdbc` driver and the community SQLite Hibernate dialect, set `spring.jpa.hibernate.ddl-auto: update`, and the server port.

### Step 2: Define the `HealthCheck` JPA entity

Create the append-only domain record for a probe.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthCheck.java`
- [ ] Define a JPA `@Entity` mapped to `HEALTH_CHECK` with an auto-increment `id`, `status` and `databaseStatus` stored as text enums (`UP`/`DOWN`), `uptimeSeconds` (long, not null), and `checkedAt` (Instant/String ISO-8601), with no-arg + all-args construction.

### Step 3: Create the `HealthCheckRepository`

Provide persistence for probe records.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthCheckRepository.java`
- [ ] Define a `JpaRepository<HealthCheck, Long>` (Spring Data) for saving health-check records.

### Step 4: Define the `HealthResponse` DTO

Shape the JSON contract returned to clients, including the nested uptime object.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthResponse.java`
- [ ] Define a Java record `HealthResponse(String status, String database, Uptime uptime, String timestamp)` with a nested `Uptime(long seconds, String since)` record, serialized to the JSON contract above.

### Step 5: Capture the application start time

Make the process start instant available for uptime computation.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthService.java`
  - `back/src/main/java/dev/aiddbot/abjavareact/AbJavaReactApplication.java`
- [ ] Record the start `Instant` at boot (e.g. captured when the service bean is constructed) so uptime can be derived as the difference from the probe time.

### Step 6: Implement `HealthService` (probe, uptime, timestamp, persistence)

Assemble the vitals and persist the probe.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthService.java`
- [ ] Probe database connectivity (e.g. a lightweight repository/connection check) and set `databaseStatus`; set overall `status` = `UP` only if the DB probe succeeds, else both `DOWN`.
- [ ] Compute `uptimeSeconds` as whole seconds between the captured start time and `now`, and set `timestamp`/`checkedAt` to the current server time in ISO-8601 UTC.
- [ ] Persist one `HealthCheck` record (status, databaseStatus, uptimeSeconds, checkedAt) and return a populated `HealthResponse`.
- [ ] Catch database/probe failures internally so the method returns a `DOWN` result instead of propagating an exception.

### Step 7: Implement `HealthController` with correct status codes

Expose the endpoint and map health to HTTP status.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/health/HealthController.java`
- [ ] Add `@RestController` handling `GET /api/health` that delegates to `HealthService` and returns `200 OK` when `status` = `UP` and `503 Service Unavailable` when `status` = `DOWN`, both carrying the `HealthResponse` body.

### Step 8: Configure CORS for the SPA origin

Allow the frontend dev origin to call the API.

- Paths involved:
  - `back/src/main/java/dev/aiddbot/abjavareact/shared/`
  - `back/src/main/resources/application.yml`
- [ ] Add a CORS configuration permitting the SPA origin to issue `GET` requests to `/api/**`.

### Step 9: Provision and ignore the database file

Ensure the embedded SQLite file location exists at runtime and is never committed.

- Paths involved:
  - `back/data/`
  - `back/.gitignore`
- [ ] Ensure the `data/` directory is present/created on startup and add `data/` and `*.db` to `.gitignore` so the generated database is not committed.
