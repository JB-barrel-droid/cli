# Itinerary Workstream Matrices

Last updated: 2026-03-11T17:35:56Z

This document keeps the itinerary feature implementation-ready while source files are blocked, with concrete acceptance checks aligned to the three required workstreams.

## Workstream 1: Schema/Setup and Migrations

### Migration Step Matrix

| Step | Change | Validation Check |
|---|---|---|
| 1 | Add `ITINERARY_COLS` in fixed 22-column order | Header row in `Itinerary` exactly matches `spec.md` order |
| 2 | Implement `ensureItineraryTab(ss)` | Running setup on a blank workbook creates `Itinerary` with frozen header + filters |
| 3 | Wire setup path | `setup` can run repeatedly without duplicate tabs or header drift |
| 4 | Wire upgrade path | Existing workbook gains missing itinerary structures without altering non-itinerary tabs |
| 5 | Add ID backfill (`Trip ID`, `Segment ID`) | Empty IDs are generated once and remain stable on rerun |
| 6 | Add chronological sort with insertion tie-break | Same-date rows sort deterministically: start, end, insertion order |

### Rollback Guardrail

- Commit checkpoint before first migration wiring change.
- Commit checkpoint after idempotent setup + upgrade validation passes.

## Workstream 2: Brief and Calendar Sync Behavior

### Routing and Policy Matrix

| Scenario | Input Shape | Expected Behavior |
|---|---|---|
| Jeremy-only itinerary row | `Traveler=Jeremy`, `Co-Travelers` empty, `Sync Target=Calendar` | Sync to Jeremy calendar only |
| Lindsay-only itinerary row | `Traveler=Lindsay`, `Co-Travelers` empty, `Sync Target=Calendar` | Sync to Lindsay calendar only |
| Shared row | `Traveler=Jeremy`, `Co-Travelers` contains Lindsay, `Sync Target=Calendar` | Sync to both calendars |
| Draft row | `Status=Draft` | Never create/update calendar event |
| Cancelled row | `Status=Cancelled`, existing `Calendar Event ID` | Delete synced event(s) and clear sync state |
| Completed row | `Status=Completed` | Keep past events; exclude from default brief |
| Brief include off | `Brief Include=No` | Row excluded from brief output |
| Shared divergence | Two calendar copies differ from sheet | Last-modified-wins reconciliation across sheet/calendar copies |

### Brief Output Expectations

- Group by `Trip ID`/`Trip Name`, then chronological segments.
- Include only traveler-relevant rows (`Traveler` or `Co-Travelers` match).
- Default filter excludes `Draft` and `Completed` rows.

## Workstream 3: Validation, Tests, Docs, and Rollback Safety

### Required Test Cases (when source is available)

| Area | Test Case | Pass Condition |
|---|---|---|
| Parsing | Missing optional times | Row loads with normalized defaults, no crash |
| Parsing | Missing IDs | IDs are generated and persisted |
| Validation | Invalid enum value (`Status`, `Sync Target`) | Rejected or normalized per validator rules |
| Briefing | Traveler relevance filter | Only matching traveler/shared rows appear |
| Briefing | Group + sort | Rows are grouped by trip and ordered chronologically |
| Sync | Jeremy/Lindsay routing | Correct calendar target(s) selected by traveler model |
| Sync | Draft/Cancelled/Completed policies | Policies applied exactly as in `spec.md` |
| Migration | Existing populated workbook upgrade | No regression in existing tabs/flows |

### Verification Commands for This Blocked Workspace

```bash
rg --files | rg '\.gs$|appsscript\.json$'
cargo --version
```

Current result: no Apps Script source files are present in `/Users/botcomp/.codex/worktrees/31a0/Google CLI`, `/Users/botcomp/.codex/worktrees/048c/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, or `/Users/botcomp/Projects/Google CLI`, and both `cargo` and `rustc` are not installed in this environment, so implementation and Rust test execution are both blocked here.

Current run branch context: `codex/itinerary-supervisor-20260311-1635` with rollback checkpoint `e9405e6`.
