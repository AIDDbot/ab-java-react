# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

### Changed

### Fixed

### Removed

## [0.1.0] - 2026-05-29

### Added

- Health Check feature reporting system vitals across all tiers (`health-check`, see `docs/specs/health-check.spec.md`):
  - `back`: `GET /api/health` Spring Boot endpoint returning overall status, database status, process uptime, and server timestamp; `200 OK` when healthy and `503 Service Unavailable` when the database probe fails (never leaking a stack trace). Each probe is persisted as an append-only `HealthCheck` record in SQLite.
  - `front`: React SPA that fetches the probe on load and renders the vitals with explicit loading and error/unhealthy states.
  - `e2e`: Playwright suite driving the SPA against the live API, covering the healthy, loading, down, and unreachable scenarios.
