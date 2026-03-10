# Itinerary Source Unblock Checklist

Use this checklist as soon as the Personal Date Tracker Apps Script source is available.

## Required Source Artifacts

- Bound script manifest: `appsscript.json`
- Script files containing setup, upgrade, brief, and calendar sync logic (`*.gs`)

## Placement

- Place source files under a dedicated directory in this repo (for example: `personal-date-tracker/`).
- Preserve original filenames to keep function references stable.

## Immediate Workstream Assignment

1. Schema/setup and migrations
- Add `ITINERARY_COLS` and `ensureItineraryTab(ss)`.
- Wire setup + upgrade flows.

2. Brief and calendar sync behavior
- Add itinerary loading and brief grouping/filtering.
- Add Jeremy/Lindsay routing and shared-row dual sync.

3. Validation/tests/docs/rollback safety
- Add validation + regression tests for itinerary parsing/sync rules.
- Commit checkpoint before migration wiring, before sync reconciliation changes, and after each passing test milestone.

## Verification Order

1. Setup/upgrade safety on existing sheets.
2. Empty-itinerary brief behavior.
3. Traveler-specific brief filtering.
4. Sheet-to-calendar and calendar-to-sheet sync routing.
5. Status policies (`Draft` no sync, `Cancelled` delete, `Completed` keep past events).
6. Shared-row conflict resolution (last-modified-wins).

## Run Exit Criteria

- Three workstreams remain active with no unassigned scope.
- Latest rollback-safe commit hash is recorded in run summary and automation memory.
- `spec.md`, `plan.md`, and `learnings.md` reflect any implementation decision changes.
