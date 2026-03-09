# Itinerary Tab Implementation Plan

## Active Workstreams (2026-03-09T23:01:18Z run)

1. Schema/setup and migrations (active): Keep the tab schema and setup migration plan implementation-ready by maintaining the locked 22-column contract and setup/upgrade touchpoint checklist while Apps Script sources are unavailable in this worktree.
2. Brief and calendar sync behavior (active): Keep the brief and calendar-sync behavior contract implementation-ready by preserving the reconciliation/default rules and explicit Jeremy/Lindsay routing requirements for immediate coding once script files are present.
3. Validation, tests, docs, and rollback safety (active): Validate blockers, keep spec/plan/learnings in sync, and maintain rollback-safe checkpoint commits on a dedicated `codex/` branch after each verified milestone.

## Current Run Status

- Status: blocked on implementation because no Personal Date Tracker Apps Script files (`*.gs`, `appsscript.json`) are present in this repository snapshot.
- Reassignment to keep three active streams: all three streams are running as specification hardening and verification streams so coding can begin immediately when the script source becomes available.
- Dedicated branch: `codex/itinerary-supervisor-20260309-d`.
- Latest safe rollback point for this run: `227ebabbd1b30216f0ebc169b216dc45e2a91398`.

## Run Verification (2026-03-09T23:01:18Z)

- Re-checked repository for implementation targets: `find . -type f \( -name '*.gs' -o -name 'appsscript.json' \)` returned no files.
- Confirmed branch safety baseline with `git rev-parse HEAD` before edits.

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
