# Current Learnings

## Confirmed System Context

- The spreadsheet `Personal Date Tracker` is readable and active.
- The bound Apps Script project is also named `Personal Date Tracker`.
- The Apps Script `parentId` matches the spreadsheet ID, so the script is definitely bound to that sheet.
- The script is large and already supports substantial functionality across dates, identities, orders, calendar sync, email delivery, and Gmail reply processing.

## Current Spreadsheet Shape

Confirmed existing tabs include at least:

- `Dates`
- `Identities`
- `Escalation Settings`
- `Orders`

The workbook already uses dedicated tabs for distinct operational domains, which supports adding `Itinerary` as a first-class tab rather than overloading `Dates`.

## Current Script Characteristics

- The script uses explicit column maps for each domain.
- The script has setup and upgrade patterns already, including functions like `setup`, `ensureIdentitiesTab`, `ensureOrdersTab`, and related migration helpers.
- The system already builds daily summaries / briefs and has logic for per-person relevance.
- The system already uses Google Calendar, Gmail, Google Tasks, and a web app deployment.
- The current calendar sync implementation is real and bidirectional, but it is centered on one configured calendar target rather than explicit per-person calendar ownership.
- The script references task-related behavior and enables the Google Tasks advanced service, but task sync does not appear clearly implemented at the same maturity level as calendar sync.
- Google Tasks is organized around per-user task lists. It does not present a true multi-owner task model for this use case.

## Product Implications

- The existing architecture favors a tab-per-domain approach with explicit column constants.
- The itinerary feature should follow that pattern instead of being embedded as a loose extension of `Dates`.
- Because the system already has identity-aware summaries, itinerary data should integrate through traveler-based filtering rather than a single-owner assumption.
- Jeremy and Lindsay should be treated as first-class sync owners for the next feature slice, not just names in the `Identities` tab.
- Trip tasks should stay in `Dates`; the new `Itinerary` tab should focus on time-series trip segments and event sync.
- Single-owner task sync is more logical than mirrored task copies because it preserves accountability and avoids reconciliation drift.

## Design Learnings

- Trips are not well-modeled as one row per event in the current `Dates` schema because travel introduces sequential segments, multi-day stays, and mixed row types.
- A single normalized itinerary table is more flexible than separate tabs for flights, hotels, and meals at this stage.
- `Trip ID` plus `Segment ID` is likely the right base model for future-proofing imports, edits, and brief generation.

## Technical Learnings

- Apps Script API access is now enabled and usable from this machine.
- `gws` does not expose Apps Script directly in this repo build, but the stored OAuth credentials can still be used to call `script.googleapis.com`.
- Future implementation work can be reviewed against the live bound script rather than guessed from partial context.
- The current script contains `syncCalendarToSheet`, `syncBidirectional`, `createCalendarEvent`, `updateCalendarEvent`, and `deleteCalendarEvent`, which gives a concrete foundation for expanding sync behavior.
- The current script appears to default calendar-originated sheet items to the first identity or configured calendar rather than a strict Jeremy/Lindsay ownership map.

## Assumptions For Now

- `Itinerary` should be a new tab, not a replacement for `Dates`.
- Manual entry is acceptable in v1.
- Bidirectional Google Calendar and Google Tasks sync for Jeremy and Lindsay is now a core requirement, not follow-on polish.
- Flexibility for multiple travelers is a hard requirement, not a nice-to-have.
- Both Jeremy's and Lindsay's calendars should be connected.
- Jeremy and Lindsay calendar routing should use fixed calendar IDs stored in Settings.
- Jeremy and Lindsay task routing should use fixed Google Task list IDs stored in Settings.
- The primary person listed on an event row is the owner for sync purposes.
- Itinerary briefs should default to the full active trip window.
- V1 should use a flat `Itinerary` table with trip metadata repeated per row, not header rows.
- Hotel stays should be one multi-day row with start and end fields.
- Traveler rows should store display names and resolve through `Identities` for email and sync routing.
- Shared travel between Jeremy and Lindsay is a first-class requirement and should not require duplicate manual itinerary rows.
- Shared itinerary authoring should use one row with a primary `Traveler` and additional people in `Co-Travelers`.
- Non-calendar itinerary items should default to `Sync Target = None` unless explicitly treated as events.
- Flights should only sync to calendar when explicitly opted in.
- Meals should be brief-only by default and sync only when explicitly opted in.
- V1 location structure should stay lightweight with `Venue / Property` and `Address / Details`.
- V1 should keep one unified itinerary table with no type-specific extra columns.
- The first implementation slice should focus on `Itinerary` tab creation, brief integration, and itinerary calendar sync before `Dates` task sync changes.
- Any itinerary segment type may opt into calendar sync explicitly; calendar sync is not limited to `Event` rows.
- Itinerary status values should be `Draft`, `Planned`, `Confirmed`, `Cancelled`, and `Completed`.
- `Draft` itinerary rows should be hidden from briefs by default.
- `Draft` itinerary rows should not sync to calendar until they move out of `Draft`.
- `Cancelled` itinerary rows should remove any synced calendar events.
- `Completed` itinerary rows should be hidden from briefs by default.
- `Completed` itinerary rows should leave past synced calendar events in place.
- When itinerary timestamps tie, later-added rows should sort after earlier-added rows.
- `Trip ID` should be an auto-generated opaque ID.
- `Segment ID` should be an auto-generated opaque ID.
- New itinerary rows should default `Brief Include` to `Yes`.
- New itinerary rows should default `Sync Owner` to the primary `Traveler`.
- New itinerary rows should default `Status` to `Planned`.
- Itinerary `Sync Target` should allow only `None` or `Calendar`.
- If the other person's name is not present, the event should not sync to that person's calendar.
- If both Jeremy and Lindsay are present, the event should sync to both calendars.
- If shared calendar copies diverge, last modified timestamp should win.
- Tasks should live in `Dates`.
- For tasks, one person should be the accountable Google Tasks owner.
- Secondary involvement for tasks should remain sheet-level context rather than a second Google Task copy.
- The existing `Co-Owners` field should remain the sheet-level place for secondary task involvement.
- V1 needs four fixed external routing settings: `Jeremy Calendar ID`, `Lindsay Calendar ID`, `Jeremy Task List ID`, and `Lindsay Task List ID`.
- The `Itinerary` tab should use 22 columns in a fixed order so script constants and validations stay stable.

## Open Questions To Resolve During Build

- None for v1 scope at this time.

## Run Log

- 2026-03-09: Re-synced `spec.md`, `plan.md`, and `learnings.md` into this worktree branch after they were absent from the repository snapshot in this workspace.
- 2026-03-09: Reconfirmed blocker that no Personal Date Tracker Apps Script source files (`*.gs`, `appsscript.json`) are present in this worktree, which still prevents direct implementation edits.
- 2026-03-09: Kept exactly three active workstreams by assigning source-unblocked implementation sequencing to stream 1, sync/brief behavior rules to stream 2, and verification + rollback protection to stream 3.
- 2026-03-10: Added `docs/itinerary-workstream-matrices.md` with concrete migration, sync-policy, and test acceptance matrices to keep three parallel workstreams implementation-ready while source remains blocked.
- 2026-03-10: Reconfirmed blocker persists in this workspace: no Personal Date Tracker Apps Script source files (`*.gs`, `appsscript.json`) are present.
- 2026-03-10: Verified environment constraint: `cargo` is unavailable in this runtime, so Rust test execution cannot run here.
- 2026-03-10: Reconfirmed blocker in the active worktree `/Users/botcomp/.codex/worktrees/d3ea/Google CLI` and corrected stale blocker-path documentation in `docs/itinerary-implementation-readiness.md`.
- 2026-03-10: Added `docs/itinerary-unblock-handoff.md` with an exact first-unblocked execution sequence across the three streams to minimize time-to-implementation once Apps Script source files are added.
- 2026-03-10: Created and switched to dedicated branch `codex/itinerary-supervisor-20260310-d` from checkpoint commit `46d1292` to preserve rollback-safe implementation continuity in this worktree.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/5e89/Google CLI` and `/Users/botcomp/Projects/Google CLI`: no Personal Date Tracker Apps Script source files (`*.gs`, `appsscript.json`) are present in either location.
- 2026-03-10: Refreshed stream-control docs (`plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`) to keep exactly three active workstreams implementation-ready under the current blocked constraints.
- 2026-03-10: Created and switched to dedicated branch `codex/itinerary-supervisor-20260310-e` from checkpoint `0fe9620` to continue without detached-HEAD risk.
- 2026-03-10: Revalidated blocker again in `/Users/botcomp/.codex/worktrees/5e89/Google CLI` and `/Users/botcomp/Projects/Google CLI`; both still contain no `*.gs` or `appsscript.json`, so implementation remains source-blocked.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` timestamps and branch context for this run.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-f` in this workspace to remove detached-HEAD risk before continuing supervision.
- 2026-03-10: Restored itinerary control artifacts (`spec.md`, `plan.md`, `learnings.md`, and `docs/itinerary-*`) from `codex/itinerary-supervisor-20260310-e` because this worktree snapshot did not include them.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/470d/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; all still contain no `*.gs` or `appsscript.json`, so implementation remains source-blocked.
- 2026-03-10: Reconfirmed runtime tooling blocker: `cargo` is unavailable in this environment (`command not found`), so Rust lint/test commands cannot run here.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-h` from detached `HEAD` (`5e7d120`) and restored itinerary control docs by cherry-picking `fccff80` as checkpoint `210fb6c`.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/8518/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed stream-control docs (`plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, `docs/itinerary-unblock-handoff.md`) for current workspace and branch context while keeping exactly three active workstreams.
- 2026-03-10: Reconfirmed runtime tooling blocker: `cargo` remains unavailable (`command not found`), so Rust lint/test commands cannot be executed in this environment.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-2` from detached `HEAD` (`5e7d120`) for this run.
- 2026-03-10: Restored missing itinerary control artifacts (`spec.md`, `.changeset/itinerary-readiness-matrices.md`, and `docs/itinerary-source-unblock-checklist.md`) by pulling them from checkpoint `210fb6c` so this workspace is once again aligned with the itinerary supervision baseline.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/1c4d/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed stream-control docs (`plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, `docs/itinerary-unblock-handoff.md`) with current branch and blocker-path context while keeping exactly three active workstreams.
- 2026-03-10: Reconfirmed runtime tooling blocker: `cargo` remains unavailable (`command not found`), so Rust lint/test commands cannot be executed in this environment.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-f170` from detached `HEAD` (`5e7d120`) and restored itinerary supervision baseline artifacts from checkpoint `b007451` as local rollback-safe commit `4340cff`.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/f170/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` for the f170 run context while keeping exactly three active workstreams.
- 2026-03-10: Reconfirmed runtime tooling blocker: `cargo` remains unavailable (`command not found`), so Rust lint/test commands cannot be executed in this environment.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-1700` from detached `HEAD` (`5e7d120`) to preserve rollback-safe continuity for this workspace.
- 2026-03-10: Restored itinerary control docs by cherry-picking checkpoint `b880c598` as local commit `8c3f2f2`, then restored missing `spec.md`, `.changeset/itinerary-readiness-matrices.md`, and `docs/itinerary-source-unblock-checklist.md` from `b007451` as checkpoint `a2c5eb5`.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/048c/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` for the 1700 run context while keeping exactly three active workstreams; reconfirmed tooling blocker that `cargo` is unavailable.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-1800` from inherited supervisor baseline `3cfc8e7` because `codex/itinerary-supervisor-20260310-1700` is attached to another worktree and could not be checked out here.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/ea26/Google CLI`, `/Users/botcomp/.codex/worktrees/048c/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` for the 1800 run context while keeping exactly three active workstreams; reconfirmed tooling blocker that `cargo` is unavailable in this environment.
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-1900` from `codex/itinerary-supervisor-20260310-1800` to continue with rollback-safe branch isolation in the `ba1f` worktree.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/ba1f/Google CLI`, `/Users/botcomp/.codex/worktrees/048c/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` for the 1900 run context while keeping exactly three active workstreams; reconfirmed runtime tooling blocker that `cargo` is unavailable (`command not found`).
- 2026-03-10: Created dedicated branch `codex/itinerary-supervisor-20260310-2000` from inherited supervisor checkpoint `75c85b1` because `codex/itinerary-supervisor-20260310-1900` is attached to another worktree and cannot be checked out here.
- 2026-03-10: Revalidated blocker in `/Users/botcomp/.codex/worktrees/3c0c/Google CLI`, `/Users/botcomp/.codex/worktrees/048c/Google CLI`, `/Users/botcomp/.codex/worktrees/5e89/Google CLI`, and `/Users/botcomp/Projects/Google CLI`; no `*.gs` or `appsscript.json` were found in any location.
- 2026-03-10: Refreshed `plan.md`, `docs/itinerary-implementation-readiness.md`, `docs/itinerary-workstream-matrices.md`, and `docs/itinerary-unblock-handoff.md` for the 2000 run context while keeping exactly three active workstreams; reconfirmed runtime tooling blocker that `cargo` is unavailable (`command not found`).
