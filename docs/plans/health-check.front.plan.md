---
slug: health-check
tier: front
status: pending
---
# Implementation Plan — health-check — front

## Specification

The React + Vite SPA fetches `GET /api/health` on load and renders the system vitals: overall status, database status, human-readable uptime (derived from `uptime.seconds`), and the current server timestamp. It shows a loading indicator while the request is in flight and a clear unhealthy/error state if the request fails or returns `status` = `DOWN`. Structure follows `docs/arch/front.arch.md` (feature folder + shared http client/types) and exposes stable `data-testid` selectors for E2E.

### Data model

Client-side `HealthResponse` type mirroring the API contract:

```ts
type HealthResponse = {
  status: "UP" | "DOWN";
  database: "UP" | "DOWN";
  uptime: { seconds: number; since: string };
  timestamp: string;
};
```

## Implementation Steps

### Step 1: Define the shared `HealthResponse` type

Mirror the API JSON contract on the client as the single source of truth.

- Paths involved:
  - `front/src/shared/types/health.ts`
- [ ] Declare the `HealthResponse` type with `status`, `database`, nested `uptime { seconds, since }`, and `timestamp`.

### Step 2: Implement the shared HTTP client

Centralize fetch with the API base URL and error handling.

- Paths involved:
  - `front/src/shared/api/httpClient.ts`
  - `front/.env` (or Vite env config)
- [ ] Implement a typed `fetch` wrapper that resolves the API base URL from Vite env, parses JSON, and throws on non-OK responses so callers can surface errors.

### Step 3: Implement the health API client

Call the endpoint through the shared client.

- Paths involved:
  - `front/src/features/health/healthApi.ts`
- [ ] Implement `getHealth(): Promise<HealthResponse>` that calls `GET /api/health` via `httpClient`.

### Step 4: Implement the `useHealth` hook

Manage fetch lifecycle and expose state to the view.

- Paths involved:
  - `front/src/features/health/useHealth.ts`
- [ ] Implement a hook that fetches on mount and exposes `{ data, loading, error }`, treating a `DOWN` response or rejected request as an error/unhealthy state.

### Step 5: Implement the `HealthStatus` component

Render the vitals and all UI states.

- Paths involved:
  - `front/src/features/health/HealthStatus.tsx`
- [ ] Render a loading indicator while `loading`; on success show overall status, database status, uptime formatted as `Xh Ym Zs` from `uptime.seconds`, and the `timestamp` in a readable form; on error/`DOWN` show a clear, non-empty unhealthy message.
- [ ] Add stable `data-testid` attributes for status, database, uptime, timestamp, loading, and error so E2E can assert them.

### Step 6: Compose the component in `App`

Render the feature at the app root.

- Paths involved:
  - `front/src/App.tsx`
  - `front/src/main.tsx`
- [ ] Render `HealthStatus` from `App`, mounted by the React root in `main.tsx`.
