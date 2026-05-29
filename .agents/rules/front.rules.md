---
description: Front coding conventions for ab-java-react
globs: "front/src/**/*.{ts,tsx}"
---
# Front Conventions — ab-java-react

## Summary

TypeScript + React 19 (Vite) tier. Function components only, with hooks for state/effects and a thin typed API layer for I/O. Conventions follow the Airbnb React/TypeScript style guide and the React docs: small presentational components, custom hooks for data fetching, strict typing, and no `any`.

---

## Naming

### Folders
- **Pattern**: feature-based (`features/{feature}`), `shared/` for cross-cutting
- **Casing**: kebab-case for folders; feature names singular
- **Examples**: `features/health`, `shared/api`

### Files
- **Component**: `PascalCase.tsx` — `HealthStatus.tsx`
- **Hook**: `useCamelCase.ts` — `useHealth.ts`
- **Service/API**: `camelCaseApi.ts` — `healthApi.ts`
- **Type module**: `camelCase.ts` — `health.ts`

### Language Elements
- **Components**: PascalCase — `HealthStatus`
- **Interfaces / Types**: PascalCase (prefer `type`; no `I` prefix) — `HealthResponse`
- **Functions / Hooks**: camelCase (`use` prefix for hooks) — `useHealth`
- **Variables / Fields**: camelCase — `isLoading`
- **Constants**: UPPER_SNAKE for module-level constants — `API_BASE_URL`
- **Enums**: avoid `enum`; use union literal types — `type Status = 'UP' | 'DOWN'`

---

## Artifact Roles

### Type

**Dominant pattern**: shared `type` mirroring an API contract; single source of truth, no classes.

**Canonical example**:
```ts
export type HealthResponse = {
  status: 'UP' | 'DOWN';
  database: 'UP' | 'DOWN';
  checkedAt: string;
};
```

**Anti-pattern**:
- Using `any` or redeclaring the shape per component — import the shared type.

### Service (API client)

**Dominant pattern**: pure async function that calls the HTTP client and returns typed data; no React imports.

**Canonical example**:
```ts
import { httpClient } from '../../shared/api/httpClient';
import type { HealthResponse } from '../../shared/types/health';

export async function getHealth(): Promise<HealthResponse> {
  return httpClient.get<HealthResponse>('/api/health');
}
```

**Anti-pattern**:
- Calling `fetch` directly inside components (untyped, untestable, scatters base-URL logic).

### Hook

**Dominant pattern**: custom hook owning fetch lifecycle (loading/error/data); returns a typed state object.

**Canonical example**:
```ts
export function useHealth() {
  const [data, setData] = useState<HealthResponse | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    getHealth().then(setData).catch(setError).finally(() => setIsLoading(false));
  }, []);

  return { data, error, isLoading } as const;
}
```

**Anti-pattern**:
- Putting fetch logic and JSX in the same component — separate data (hook) from view.

### Component

**Dominant pattern**: small, typed function component; props via a `type`; render states explicitly; `data-testid` for E2E hooks.

**Canonical example**:
```tsx
export function HealthStatus() {
  const { data, error, isLoading } = useHealth();
  if (isLoading) return <p data-testid="health-loading">Loading…</p>;
  if (error) return <p data-testid="health-error" role="alert">Unavailable</p>;
  return <p data-testid="health-status">{data?.status}</p>;
}
```

**Anti-pattern**:
- Class components, default exports, and inline business logic in JSX.

---

## Wiring and Dependencies

Components import hooks; hooks import services; services import the shared `httpClient`. No upward imports (shared never imports from features). Prefer named exports. See `front.arch.md` — Dependencies between features.

## Error Handling

The `httpClient` throws on non-2xx; hooks catch and expose an `error` state; components render an accessible error (`role="alert"`). Never swallow errors silently.

---

## Testing

### Infrastructure
- **Framework**: Vitest + React Testing Library
- **Runner**: `npm run test`
- **Placement**: colocated next to source
- **File naming**: `*.test.tsx` / `*.test.ts`

### Patterns
- **Setup**: render with RTL; mock the API service module (not `fetch`)
- **Mocking**: `vi.mock('./healthApi')`; assert via accessible queries/`data-testid`
- **Assertions**: `@testing-library/jest-dom` matchers

### Canonical Test Example

```tsx
vi.mock('./healthApi', () => ({ getHealth: vi.fn().mockResolvedValue({ status: 'UP', database: 'UP', checkedAt: '' }) }));

test('renders UP status', async () => {
  render(<HealthStatus />);
  expect(await screen.findByTestId('health-status')).toHaveTextContent('UP');
});
```

### Coverage by Artifact Role

| Role | Tested | Scope |
|------|--------|-------|
| Component | Yes | loading / error / success |
| Hook | Yes | resolves + rejects |
| Service | Yes | maps response |
| Type | No | compile-time only |

### What NOT to Test
- Third-party libraries, styling details, and implementation internals — test behavior via the DOM.

---

## Known Deviations

- No deviations yet — greenfield baseline.
