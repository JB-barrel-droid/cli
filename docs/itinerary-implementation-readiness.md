# Itinerary Implementation Readiness

Last updated: 2026-03-10T16:01:24Z

This document keeps the itinerary feature implementation-ready while Apps Script source files are unavailable in this workspace.

Detailed stream acceptance matrices live in `docs/itinerary-workstream-matrices.md`.

## Active Workstreams (Exactly Three)

1. Schema/setup and migrations
2. Brief and calendar sync behavior
3. Validation, tests, docs, and rollback safety

Current assignment while source is blocked:

1. Stream 1 owns migration-order locking and setup idempotency acceptance checks.
2. Stream 2 owns brief/sync decision tables and traveler-routing policy proofs.
3. Stream 3 owns blocker verification, rollback checkpoints, and test-command readiness.

## Stream 1: Schema/Setup + Migrations

Execution order once Apps Script source is available:

1. Add `ITINERARY_COLS` with the full v1 schema from `spec.md`.
2. Implement `ensureItineraryTab(ss)`:
   - create tab if missing
   - write headers
   - freeze header row
   - set filters and default formatting
   - apply data validation lists
3. Wire setup path to call `ensureItineraryTab(ss)`.
4. Wire upgrade/migration path to create/populate missing itinerary structures without changing existing tab behavior.
5. Add stable opaque ID generation for `Trip ID` and `Segment ID` when missing.
6. Add chronological sort routine:
   - `Start Date`
   - `Start Time`
   - `End Date`
   - `End Time`
   - insertion-order tie-break

Milestone gate:

- Existing tabs still initialize unchanged.
- Empty `Itinerary` tab does not break setup or summary generation.

## Stream 2: Brief + Calendar Sync Behavior

Execution order once Apps Script source is available:

1. Add itinerary row loader (`getItineraryData(ss)`) with normalization for traveler, status, dates/times, and sync fields.
2. Extend brief builder:
   - traveler relevance via `Traveler` + `Co-Travelers`
   - `Brief Include = Yes`
   - exclude `Draft` and `Completed` by default
   - group by trip, then chronological rows
3. Add traveler-to-calendar routing:
   - Jeremy-only rows -> Jeremy calendar
   - Lindsay-only rows -> Lindsay calendar
   - shared rows -> dual calendar sync
4. Add sync policy:
   - `Draft`: never sync
   - `Cancelled`: delete synced events
   - `Completed`: keep past events
5. Add shared divergence handling with last-modified-wins.

Milestone gate:

- Sheet-to-calendar and calendar-to-sheet update loops preserve ownership rules.
- Shared events do not regress into single-calendar behavior.

## Stream 3: Validation/Tests/Docs + Rollback Safety

Execution order once Apps Script source is available:

1. Add/refresh validation rules for required fields and enum controls.
2. Add tests for:
   - itinerary row parsing and defaults
   - brief filtering/grouping
   - traveler routing and shared-row dual sync
   - cancelled/completed/draft sync behavior
   - migration safety on pre-existing sheets
3. Update `spec.md`, `plan.md`, `learnings.md` when implementation deviates from current assumptions.
4. Keep rollback points:
   - checkpoint commit before migration wiring
   - checkpoint commit before sync reconciliation changes
   - checkpoint commit after each validated milestone

Milestone gate:

- Tests pass for new logic and no regression in existing summary/sync paths.
- Latest safe rollback commit is documented in run output and memory.

## Blocker Status

Current blocker remains active: no `*.gs` or `appsscript.json` source files are present in:

- `/Users/botcomp/.codex/worktrees/0788/Google CLI`
- `/Users/botcomp/.codex/worktrees/5e89/Google CLI`
- `/Users/botcomp/Projects/Google CLI`

No code-level itinerary implementation can proceed until script source is present in this workspace.

Current execution branch: `codex/itinerary-supervisor-20260310-0788` (restored baseline at `a883410`).
