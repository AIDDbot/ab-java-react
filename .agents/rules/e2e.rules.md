---
description: E2E coding conventions for ab-java-react
globs: "e2e/**/*.ts"
---
# E2E Conventions — ab-java-react

## Summary

TypeScript + Playwright Test tier. Specs are user-centric and isolated, using web-first assertions and resilient locators (`getByRole`, `getByTestId`) over CSS/XPath. Page Objects encapsulate selectors. Conventions follow the official Playwright best-practices guide: test user-visible behavior, never use hard waits, keep tests independent.

---

## Naming

### Folders
- **Pattern**: feature-based — `tests/` for specs, `pages/` for Page Objects
- **Casing**: kebab-case for folders, lowercase spec files
- **Examples**: `tests`, `pages`

### Files
- **Spec**: `{flow}.spec.ts` — `health.spec.ts`
- **Page Object**: `{Page}Page.ts` — `HealthPage.ts`
- **Config**: `playwright.config.ts`

### Language Elements
- **Classes (Page Objects)**: PascalCase — `HealthPage`
- **Types**: PascalCase — `HealthFixtures`
- **Functions / Methods**: camelCase — `expectStatusUp`
- **Variables**: camelCase — `healthPage`
- **Constants**: UPPER_SNAKE — `BASE_URL`
- **Test ids**: kebab-case strings — `health-status`

---

## Artifact Roles

### Page Object

**Dominant pattern**: class wrapping a `Page`; exposes intent-revealing actions and locators, no assertions inside (return locators, assert in specs).

**Canonical example**:
```ts
import { type Page, type Locator } from '@playwright/test';

export class HealthPage {
  readonly status: Locator;
  constructor(private readonly page: Page) {
    this.status = page.getByTestId('health-status');
  }
  async goto() { await this.page.goto('/'); }
}
```

**Anti-pattern**:
- Hard-coded CSS/XPath selectors and `expect` calls buried inside Page Objects.

### Spec

**Dominant pattern**: `test.describe` grouping; one user flow per `test`; web-first assertions that auto-retry; independent (no shared mutable state).

**Canonical example**:
```ts
import { test, expect } from '@playwright/test';
import { HealthPage } from '../pages/HealthPage';

test.describe('Health check', () => {
  test('shows system status UP', async ({ page }) => {
    const health = new HealthPage(page);
    await health.goto();
    await expect(health.status).toHaveText('UP');
  });
});
```

**Anti-pattern**:
- `page.waitForTimeout(...)` / arbitrary sleeps, and tests that depend on a previous test's state.

### Config

**Dominant pattern**: central `playwright.config.ts` with `baseURL`, browser projects, retries on CI, and a `webServer` block to boot the app under test.

**Canonical example**:
```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: { baseURL: 'http://localhost:5173', trace: 'on-first-retry' },
  webServer: { command: 'npm --prefix ../front run dev', url: 'http://localhost:5173', reuseExistingServer: !process.env.CI },
  projects: [{ name: 'chromium', use: { ...devices['Desktop Chrome'] } }],
});
```

**Anti-pattern**:
- Absolute URLs in specs instead of `baseURL`; manually starting servers outside `webServer`.

## Error Handling

Rely on Playwright's auto-waiting and web-first assertions (`expect(locator).toHaveText`) rather than try/catch. Capture `trace`, `screenshot`, and `video` on first retry for diagnosis; let failures surface — do not catch and swallow.

---

## Testing

### Infrastructure
- **Framework**: `@playwright/test` (latest)
- **Runner**: `npx playwright test`
- **Placement**: specs under `tests/`, Page Objects under `pages/`
- **File naming**: `*.spec.ts`

### Patterns
- **Setup**: built-in `page` fixture; create Page Objects per test; use `test.beforeEach` for shared navigation
- **Mocking**: prefer real backend; use `page.route` only to simulate failure/edge cases
- **Assertions**: web-first `expect(locator)` matchers (auto-retrying)

### Canonical Test Example

```ts
test.beforeEach(async ({ page }) => { await new HealthPage(page).goto(); });

test('database status is UP', async ({ page }) => {
  await expect(page.getByTestId('health-status')).toHaveText('UP');
});
```

### Coverage by Artifact Role

| Role | Tested | Scope |
|------|--------|-------|
| Spec | Yes | critical user flows (happy path + key errors) |
| Page Object | No | exercised indirectly via specs |
| Config | No | infrastructure |

### What NOT to Test
- Backend units already covered by `back` tests; exhaustive field-level validation — keep E2E to user-visible flows.

---

## Known Deviations

- No deviations yet — greenfield baseline.
