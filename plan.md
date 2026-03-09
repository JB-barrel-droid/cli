# Itinerary Tab Implementation Plan

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

## Phase 3: Script Integration

- Add `ITINERARY_COLS` constant definitions.
- Add `ensureItineraryTab(ss)` and wire it into `setup` and upgrade flows.
- Add `getItineraryData(ss)` parser.
- Add shared normalization helpers for traveler names, dates, times, and status.
- Add sync-target resolution helpers for Jeremy and Lindsay.
- Add logging where itinerary parsing or validation can fail.

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

## Active Workstreams (2026-03-09 Run)

1. Schema/setup + migrations
- Owner: supervisor run in `codex/itinerary-supervisor-20260309-8d55`
- Current task: keep schema specification and migration checkpoints ready while waiting for the actual Apps Script source tree.
- Next concrete step when unblocked: implement `ITINERARY_COLS`, `ensureItineraryTab(ss)`, and setup/upgrade wiring in script source.

2. Brief + calendar sync behavior
- Owner: supervisor run in `codex/itinerary-supervisor-20260309-8d55`
- Current task: keep brief and calendar sync requirements locked and traceable to acceptance criteria.
- Next concrete step when unblocked: add itinerary row brief grouping and Jeremy/Lindsay calendar routing with dual-sync support for shared rows.

3. Validation/tests/docs + rollback safety
- Owner: supervisor run in `codex/itinerary-supervisor-20260309-8d55`
- Current task: persist run-by-run blocker and rollback metadata; keep docs synchronized across worktrees.
- Next concrete step when unblocked: add validation/test coverage for status/sync rules and run script-level verification.

## Current Blocker (2026-03-09T10:01:15Z)

- Repository still does not include the Personal Date Tracker Apps Script implementation files needed for coding (`.gs` and `appsscript.json` not present in this worktree or sibling checkout).
- Workstreams remain active with highest-value preparatory tasks, but feature code changes cannot proceed until source is available.
