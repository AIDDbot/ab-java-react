# Architecture Decision Records (ADR) — ab-java-react

## ADR-1: Monorepo with independent tiers

- **Status**: Decided
- **Decision**: Host backend (`back/`), frontend (`front/`), and E2E (`e2e/`) tiers in a single Git repository, each independently runnable.
- **Rationale**: Simplifies sharing, versioning, and onboarding for an archetype/teaching project; keeps the full stack in one cloneable unit.
- **Consequences**: Tiers have separate build tools and dependency manifests; tooling and CI must target each tier path explicitly.

## ADR-2: Java + Spring Boot for the backend

- **Status**: Decided
- **Decision**: Build the API with Java (latest LTS) and Spring Boot, managed by Maven.
- **Rationale**: Industry-standard, well-documented stack with first-class REST, dependency injection, and testing support; ideal for a learning baseline.
- **Consequences**: JVM toolchain required; backend conventions follow Spring idioms (controllers, services, repositories).

## ADR-3: REST/JSON API style

- **Status**: Decided
- **Decision**: Expose functionality as a RESTful HTTP API returning JSON, starting with `GET /api/health`.
- **Rationale**: Simple, universally understood, and directly consumable by the React SPA and Playwright tests.
- **Consequences**: Contracts defined by HTTP verbs, paths, and JSON schemas; clients depend on stable endpoint shapes.

## ADR-4: React + Vite + TypeScript for the frontend

- **Status**: Decided
- **Decision**: Build the SPA with React and TypeScript, bundled and served by Vite.
- **Rationale**: Modern, fast dev experience; TypeScript adds type safety; React is the most common teaching/SPA target.
- **Consequences**: Node.js toolchain required; frontend communicates with the API strictly over JSON/HTTP.

## ADR-5: SQLite as the data store

- **Status**: Decided
- **Decision**: Persist data in an embedded SQLite database accessed by the backend via JDBC, using Spring Data JPA with a SQLite-compatible Hibernate dialect.
- **Rationale**: Zero-setup, file-based storage keeps the archetype runnable out of the box without external services.
- **Consequences**: Single-writer concurrency limits; not intended for high-load production; dialect must be configured explicitly since SQLite is not a default Spring/Hibernate target.

## ADR-6: Playwright (TypeScript) for end-to-end verification

- **Status**: Decided
- **Decision**: Verify the full stack with a TypeScript Playwright suite in `e2e/` that drives the SPA against a running API.
- **Rationale**: Cross-browser, reliable automation that exercises real user flows; aligns frontend and test languages.
- **Consequences**: E2E runs require both API and SPA running; tests depend on stable UI selectors and API availability.
