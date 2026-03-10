# Itinerary Source-Unblock Handoff

Last updated: 2026-03-10T21:02:15Z

This checklist is the execution handoff for the first run where Apps Script source files (`*.gs`, `appsscript.json`) become available.

## Stream 1: Schema/Setup and Migrations

1. Locate constants + setup section in Apps Script source.
2. Add `ITINERARY_COLS` using exact `spec.md` V1 order (22 columns).
3. Add `ensureItineraryTab(ss)` with:
   - idempotent tab creation
   - header write/freeze/filter
   - validations for `Segment Type`, `Status`, `Sync Owner`, `Sync Target`, `Brief Include`
4. Wire `ensureItineraryTab(ss)` into setup and migration paths.
5. Add ID backfill for missing `Trip ID` and `Segment ID`.
6. Add chronological sorting with insertion-order tie-break.

Validation gate:

- Re-running setup does not modify existing non-itinerary tabs.
- Re-running migration does not regenerate stable IDs.

## Stream 2: Brief and Calendar Sync Behavior

1. Add `getItineraryData(ss)` parser and normalization helpers.
2. Extend brief builder to include itinerary rows:
   - traveler relevance (`Traveler` + `Co-Travelers`)
   - `Brief Include = Yes`
   - hide `Draft` and `Completed` by default
   - group by trip and sort chronologically
3. Add itinerary calendar target resolver:
   - Jeremy-only -> Jeremy calendar ID
   - Lindsay-only -> Lindsay calendar ID
   - Shared row -> both calendar IDs
4. Enforce sync policies:
   - `Draft` never syncs
   - `Cancelled` deletes synced event(s)
   - `Completed` keeps prior event(s)
5. Apply last-modified-wins reconciliation for shared divergence.

Validation gate:

- Shared row creates/updates both calendar copies only when both names are present.

## Stream 3: Validation, Tests, Docs, and Rollback Safety

1. Add/refresh tests for:
   - schema defaults and ID generation
   - brief inclusion/exclusion rules
   - Jeremy/Lindsay routing and shared sync
   - draft/cancelled/completed transitions
   - migration idempotency and regression safety
2. Run validation commands after each milestone:

```bash
cargo clippy -- -D warnings
cargo test
```

3. Keep rollback-safe commits:
   - before migration wiring
   - before sync reconciliation changes
   - after each milestone passes verification
4. Sync decision changes back into `spec.md`, `plan.md`, and `learnings.md`.

Validation gate:

- New tests cover modified logic paths and existing workflows show no regressions.

## Blocker Verification Command

```bash
rg --files -g '*.gs' -g 'appsscript.json'
```

Expected until unblocked: no output.
