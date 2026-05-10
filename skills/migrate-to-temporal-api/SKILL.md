---
name: migrate-to-temporal-api
description: Guides migration from Date and date-library wrappers to the native Temporal API, including best practices, do's and don'ts, and a phased plan for complex integrations.
---

# Migrate to Temporal API

Use this skill when a project currently relies on `Date` and/or date wrappers (`moment`, `dayjs`, `date-fns`, `luxon`, custom wrappers) and needs a safe migration to native `Temporal`.

## Outcomes

- Inventory all date/time usage and classify by domain concern.
- Migrate incrementally with backwards-compatible adapters.
- Avoid timezone, DST, and serialization regressions.
- Establish final conventions centered on `Temporal` types.

## Migration Principles

1. **Model intent first**
   - Use `Temporal.Instant` for machine timestamps.
   - Use `Temporal.ZonedDateTime` for user-facing date-time in a specific zone.
   - Use `Temporal.PlainDate` / `PlainTime` / `PlainDateTime` for calendar concepts without timezone.
2. **Make time zones explicit**
   - Never rely on implicit local timezone behavior.
   - Carry IANA zone IDs (`Europe/Berlin`, `America/New_York`) through boundaries.
3. **Prefer immutable transformations**
   - Replace mutation-heavy flows with chained immutable `Temporal` operations.
4. **Migrate at system boundaries first**
   - Parsing, persistence, API contracts, and scheduling boundaries should become Temporal-safe before deep refactors.
5. **Dual-read/dual-write during transition**
   - In complex systems, support both old and new formats temporarily with feature flags and telemetry.

## Do's and Don'ts

### Do

- Do define a small shared time module with canonical helpers and type aliases.
- Do standardize serialization formats per type:
  - `Instant` as ISO 8601 with `Z`.
  - `ZonedDateTime` with bracketed zone (`[Europe/Berlin]`) when needed.
  - `PlainDate` as `YYYY-MM-DD`.
- Do normalize external inputs (API/UI/DB) immediately after parsing.
- Do write regression tests for DST boundaries, month-end math, leap years, and locale formatting.
- Do keep arithmetic domain-appropriate (`date.add({ days: 1 })` vs timestamp math).

### Don't

- Don't convert everything to epoch milliseconds unless strictly required.
- Don't mix `Date` and `Temporal` in domain logic without explicit adapters.
- Don't use server-local timezone assumptions in business rules.
- Don't migrate all modules in one large refactor if the system is business-critical.
- Don't silently change API payload formats without versioning or compatibility shims.

## Phased Migration Path (Complex Integrations)

1. **Discovery & Risk Mapping**
   - Catalog all date/time call sites, grouped by: parsing, formatting, arithmetic, persistence, scheduling, external APIs.
   - Mark high-risk surfaces: billing windows, SLAs, cron-like jobs, compliance reports.

2. **Target Architecture**
   - Define canonical Temporal types for each domain entity and API boundary.
   - Document timezone ownership (user profile, tenant config, event source, system default).

3. **Compatibility Layer**
   - Introduce adapter utilities:
     - `Date -> Temporal.Instant` and `Temporal.Instant -> Date` only at boundaries.
     - Wrapper-library object -> Temporal equivalent mappings.
   - Add lint/code-review guardrails to prevent new wrapper usage.

4. **Boundary-First Refactor**
   - Update API serializers/deserializers, DB mappers, and queue/event schemas.
   - Implement dual-read/dual-write where version skew exists.

5. **Domain Refactor by Vertical Slice**
   - Migrate one workflow at a time (e.g., booking, invoicing, notifications).
   - Keep old and new implementations side-by-side behind flags until parity is proven.

6. **Validation & Observability**
   - Snapshot-compare old vs new outputs in production-like traffic.
   - Add metrics for parse failures, timezone fallbacks, and drift anomalies.

7. **Cutover & Cleanup**
   - Remove deprecated wrappers and adapter paths.
   - Enforce Temporal-only conventions via linting, architecture checks, and docs.

## Practical Mapping Cheatsheet

- `Date.now()` -> `Temporal.Now.instant()`
- `new Date(iso)` -> `Temporal.Instant.from(iso)` (if timestamp) or `Temporal.PlainDate/PlainDateTime.from(...)` (if calendar input)
- Date formatting via locale -> `temporalValue.toLocaleString(locale, options)`
- Manual timezone math -> `instant.toZonedDateTimeISO(timeZone)` and operate there

## Delivery Checklist for the Agent

- Confirm runtime support or polyfill strategy for `Temporal`.
- Produce a migration matrix: old type/helper -> target Temporal type.
- Provide a staged PR plan with rollback points.
- Add/adjust tests for timezone and calendar edge cases.
- Define explicit deprecation and removal timeline for legacy wrappers.
- If Obra Superpowers are installed, hand over migration execution using `/writing-plans` after preparing the migration plan and risk map.
