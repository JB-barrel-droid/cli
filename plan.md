# Itinerary Tab Implementation Plan

## Active Workstreams (2026-03-11 17:35Z Run)

1. Schema/setup and migrations
   - Keep the v1 22-column schema locked in `spec.md`.
   - Keep migration sequence explicit so setup/upgrade edits can be applied in one pass once source is present.
   - Maintain step-by-step migration acceptance checks in `docs/itinerary-workstream-matrices.md`.
   - Keep an implementation patch-order handoff in `docs/itinerary-unblock-handoff.md` so source onboarding is immediate.
2. Brief and calendar sync behavior
   - Keep traveler relevance and dual-calendar routing rules implementation-ready.
   - Keep status-driven sync policy (`Draft`, `Cancelled`, `Completed`) explicitly ordered for reconciliation.
   - Maintain sync-routing and policy decision table in `docs/itinerary-workstream-matrices.md`.
   - Keep concrete helper-function insertion points ready for direct Apps Script patching once files appear.
3. Validation, tests, docs, and rollback safety
   - Maintain verification matrix and rollback checkpoints while source is blocked.
   - Preserve branch hygiene and checkpoint commits with clear rollback hashes each run.
   - Keep blocked-workspace verification commands and expected outcomes current.
   - Re-verify source blocker state every run in this workspace and comparison locations:
     `/Users/botcomp/.codex/worktrees/048c/Google CLI`,
     `/Users/botcomp/.codex/worktrees/5e89/Google CLI`,
     and `/Users/botcomp/Projects/Google CLI`.

Current branch checkpoint chain for rollback-safe continuation:

- `8970b03` (latest inherited supervisor checkpoint from 14:36Z run)
- `f6a8829` (latest inherited supervisor checkpoint from 03:01Z run)
- `16b53f7` (latest inherited supervisor checkpoint from 02:00Z run)
- `fb9d689` (latest inherited supervisor checkpoint from 23:00Z run)
- `bb81c16` (restored itinerary supervisor artifacts in this workspace)
- `d4d05a3` (latest inherited supervisor checkpoint from 22:00Z run)
- `dab945e` (latest inherited supervisor checkpoint from 21:00Z run)
- `528be03` (latest inherited supervisor checkpoint from 20:00Z run)
- `e9405e6` (latest inherited supervisor checkpoint from 16:35Z run)
- current branch `codex/itinerary-supervisor-20260311-1635` (branched from rollback-safe checkpoint `e9405e6`)

## Phase 1: Design Lock

- Confirm the final `Itinerary` column set.
- Use a flat `Itinerary` table with trip metadata repeated per row.
- Store display names in `Traveler` and resolve through `Identities` to email and sync metadata.
- Use full active trip as the default itinerary brief window.
- Use fixed Settings-stored calendar IDs for Jeremy and Lindsay.
- Use fixed Settings-stored Google Task list IDs for Jeremy and Lindsay.
- Keep trip tasks in `Dates`, not `Itinerary`.

## Phase 2: Spreadsheet Schema

- Add a new `Itinerary` tab creation helper.
- Add header row, frozen row, filters, data validation, and default formatting.
- Add stable IDs for `Trip ID` and `Segment ID`.
- Add sorting behavior for chronological ordering.
- Use the exact 22-column `Itinerary` order defined in `spec.md`.

## Phase 3: Script Integration

- Add `ITINERARY_COLS` constant definitions.
- Add `ensureItineraryTab(ss)` and wire it into `setup` and upgrade flows.
- Add `getItineraryData(ss)` parser.
- Add shared normalization helpers for traveler names, dates, times, and status.
- Add sync-target resolution helpers for Jeremy and Lindsay.
- Extend Settings parsing and setup to include four fixed external IDs:
  `Jeremy Calendar ID`, `Lindsay Calendar ID`, `Jeremy Task List ID`, `Lindsay Task List ID`.
- Add logging where itinerary parsing or validation can fail.
- Reuse the existing calendar helper pattern instead of introducing a second unrelated sync architecture.

## Phase 4: Bidirectional Sync

- Add traveler-specific Google Calendar target resolution for Jeremy and Lindsay.
- Use the primary person on the row as the default owner calendar.
- If both Jeremy and Lindsay are present on a row, support dual-calendar sync for any itinerary row explicitly marked for calendar sync.
- Add traveler-specific Google Tasks target resolution for Jeremy and Lindsay.
- If a `Dates` task row is assigned to Jeremy or Lindsay, sync it only to that person's task system.
- Add itinerary row sync state fields for calendar event IDs.
- Add `Dates`-side task sync state fields for a single external task ID plus ownership metadata.
- Add bidirectional reconciliation rules for sheet-to-Google and Google-to-sheet changes.
- Use last-modified-wins conflict handling for shared calendar event divergence.

## Phase 5: Brief Integration

- Extend the current brief-building path to include itinerary segments.
- Filter by traveler relevance and `Brief Include`.
- Group entries by trip and sort chronologically.
- Format hotel, travel, meal, and event segments differently enough to be readable without making the brief noisy.
- Verify shared-trip rows can appear correctly for both Jeremy and Lindsay without duplicate authoring.

## Phase 6: Quality and Safety

- Verify existing tabs still load correctly.
- Verify setup/upgrade is safe on an already-populated tracker.
- Verify empty itinerary state does not break briefs.
- Verify invalid or partial rows degrade gracefully.
- Verify Jeremy events do not sync into Lindsay targets and vice versa.
- Verify shared Jeremy/Lindsay events sync to both calendars only when both are present.
- Verify shared event divergence resolves by latest modification timestamp.
- Verify task completion sync works in both directions.
- Verify tasks sync only to the accountable owner and do not create duplicate task copies.

## Phase 7: Optional Follow-On Work

- Email reply parsing support for itinerary additions.
- Import helpers from confirmation emails.
- Trip-level summary generation.
- Traveler-specific trip filters in the web app.

## Proposed Delivery Order

1. Add schema constants and tab setup.
2. Add upgrade support.
3. Add row parsing helpers.
4. Add Jeremy/Lindsay sync mapping for Calendar and Tasks.
5. Add bidirectional sync behavior.
6. Add brief integration.
7. Add polish, validation, and tests.

## Suggested First Implementation Slice

The best first slice is:

- create `Itinerary` tab
- support manual row entry
- read rows into Apps Script
- support `Sync Owner` and `Sync Target`
- include itinerary rows in briefs
- prove itinerary event calendar sync end-to-end for Jeremy and Lindsay
- add the four new Settings keys and validate they resolve correctly

This keeps the first release concrete while validating the highest-value itinerary workflow first. `Dates` task sync changes should follow after this slice is stable.

## Risks

- The current project already has substantial complexity and many tabs, so a weak schema choice will create cleanup work later.
- Time zones and partial times can create sorting bugs.
- Traveler identity matching may become brittle if names and emails are mixed inconsistently.
- Brief formatting can become noisy if itinerary rows are not grouped well.
- Dual-account sync logic can create duplication, drift, or wrong-owner writes if owner mapping is underspecified.
- Google Tasks reconciliation may be more brittle than calendar sync because task lists and status transitions are less rich than calendar events.
- Google Tasks does not provide true co-ownership, so a single-owner task model is safer and easier to reconcile.

## Definition of Done

- `setup` or upgrade creates the `Itinerary` tab safely.
- Users can enter itinerary rows with no manual script editing.
- Jeremy and Lindsay each have clear sync targets for calendar events and tasks.
- Event and task rows reconcile in both directions with Google.
- Briefs can include itinerary details for the right traveler.
- Existing reminders, dates, orders, and calendar sync still function.
