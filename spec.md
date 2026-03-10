# Itinerary Tab Feature Spec

## Objective

Add a new spreadsheet tab and supporting Apps Script behavior for trip itinerary details so trip-related information can be stored in a structured, time-series format and included in daily or trip-related briefs.

The feature must work for any traveler, not just one person, and must support common travel itinerary elements such as:

- flights and other travel segments
- hotels and lodging
- event venues and meeting locations
- meals and reservations
- transportation between locations
- free-form trip notes that still fit into a structured model

## Problem

The current Personal Date Tracker system is strong for date-based tasks and events, but trip planning data is more granular than the existing item model. A trip usually contains many timestamped segments across several days, locations, and categories. That information needs to be:

- structured enough to summarize cleanly
- flexible enough to cover different trip types
- tied to one or more travelers
- easy to query for a brief or digest

## Goals

- Add an `Itinerary` tab with a schema built for sequential travel segments.
- Support multiple travelers on the same trip.
- Ensure trips where Jeremy and Lindsay travel together work cleanly end-to-end.
- Support multiple trips over time with durable trip IDs.
- Model itinerary rows as time-based entries rather than generic notes.
- Make it easy to generate a trip brief from structured rows.
- Keep the design flexible enough to handle business travel, family trips, and mixed-purpose travel.
- Support bidirectional Google Calendar sync for itinerary rows that are explicitly marked for calendar sync.
- Support bidirectional Google Tasks sync for `Dates` task rows to one accountable owner at a time.

## Initial Delivery Scope

The first implementation slice should include:

- `Itinerary` tab creation and schema
- itinerary brief integration
- itinerary event calendar sync

`Dates` task sync changes remain in scope overall, but should follow after the initial itinerary slice is working.

## Non-Goals

- Full travel booking automation in the first pass.
- Automatic ingestion from airline, hotel, or calendar vendors in the first pass.
- Perfect support for every travel edge case before launch.
- Replacing the existing `Dates` tab or existing task/event model.

## Primary Use Cases

1. A traveler adds a trip with flights, hotel check-in/check-out, meeting locations, meals, and transportation.
2. Multiple people share one trip, but each itinerary row can specify the relevant traveler or group.
3. A daily brief or trip brief summarizes the next relevant itinerary segments in chronological order.
4. A traveler reviews a trip in one place without needing unstructured notes spread across tabs.
5. Jeremy and Lindsay travel together and shared itinerary rows correctly appear in briefs and sync to the right calendars.

## Proposed Tab

Tab name:

- `Itinerary`

Optional future tab:

- `Trip Settings`

The first version should stay within a single `Itinerary` tab unless configuration complexity forces a separate settings tab.

## Data Model

Each row should represent one itinerary segment or trip-related timebox.

Recommended columns:

| Column | Purpose |
|---|---|
| `Trip ID` | Stable ID for grouping rows belonging to the same trip |
| `Trip Name` | Human-readable trip label |
| `Traveler` | Primary traveler name or traveler key |
| `Co-Travelers` | Optional comma-separated additional travelers |
| `Segment ID` | Stable row-level unique ID |
| `Segment Type` | `Flight`, `Hotel`, `Train`, `Car`, `Event`, `Meal`, `Ground Transport`, `Free Time`, `Note`, `Other` |
| `Start Date` | Local calendar date for the segment start |
| `Start Time` | Optional local start time |
| `End Date` | Optional end date |
| `End Time` | Optional end time |
| `Time Zone` | Canonical timezone for the segment |
| `Origin` | Starting place or city |
| `Destination` | Ending place or city |
| `Venue / Property` | Hotel, restaurant, office, stadium, venue, airport, etc. |
| `Address / Details` | Structured location details or directions |
| `Confirmation / Reference` | Booking code, flight number, reservation number, meeting code |
| `Status` | `Draft`, `Planned`, `Confirmed`, `Cancelled`, `Completed` |
| `Sync Owner` | Which person owns the sync target, initially `Jeremy` or `Lindsay` |
| `Sync Target` | `None` or `Calendar` |
| `Calendar Event ID` | External Google Calendar event ID when synced |
| `Brief Include` | `Yes` or `No` |
| `Notes` | Additional context |

### Exact V1 Column Order

The implementation should use this exact column order for the `Itinerary` tab:

1. `Trip ID`
2. `Trip Name`
3. `Traveler`
4. `Co-Travelers`
5. `Segment ID`
6. `Segment Type`
7. `Start Date`
8. `Start Time`
9. `End Date`
10. `End Time`
11. `Time Zone`
12. `Origin`
13. `Destination`
14. `Venue / Property`
15. `Address / Details`
16. `Confirmation / Reference`
17. `Status`
18. `Sync Owner`
19. `Sync Target`
20. `Calendar Event ID`
21. `Brief Include`
22. `Notes`

### Why this shape

- `Trip ID` groups a multi-day trip.
- `Segment ID` prevents fragile row-based references.
- `Traveler` and `Co-Travelers` make the feature work for any person or shared trip.
- `Segment Type` keeps the model flexible without forcing separate tabs for flights, hotels, and meals.
- V1 should keep one unified itinerary table with no type-specific extra columns.
- V1 location modeling should stay lightweight with `Venue / Property` plus `Address / Details`.
- `Sync Owner` decouples row relevance from external ownership so a shared trip can still sync to the right person.
- `Calendar Event ID` preserves bidirectional calendar sync state cleanly.
- `Brief Include` allows operational rows to exist without cluttering briefs.
- V1 should remain a flat table with trip metadata repeated on each relevant row rather than adding trip header rows.

## Time-Series Rules

- Rows must sort by `Start Date`, then `Start Time`, then `End Date`, then `End Time`.
- All brief generation should use chronological ordering.
- Segments without time should still be allowed and should sort after timed items on the same date unless a stronger product rule is chosen.
- Multi-day rows such as hotel stays should retain both start and end fields.
- Hotel stays should be represented as one multi-day row, not separate check-in and check-out rows.
- If two rows have the same effective timestamp, tie-break by insertion order so later-added rows sort after earlier-added rows.

## Traveler Model

The feature should be identity-aware, consistent with the existing `Identities` tab:

- `Traveler` should store the person's display name, but resolve through `Identities` to the underlying email and sync targets.
- `Co-Travelers` should support multiple names or emails.
- Shared travel should use one row with `Traveler` as the primary person and the other participant listed in `Co-Travelers`.
- Brief generation should filter rows relevant to the requested traveler.
- Shared segments should be reusable instead of copied for every traveler unless later automation requires denormalization.

## Brief Integration

The new tab should support inclusion in briefs using structured filtering:

- upcoming itinerary rows for a specific traveler
- rows within the full active trip by default
- rows flagged with `Brief Include = Yes`
- rows with meaningful status such as `Planned` or `Confirmed`
- `Draft` rows should be hidden from briefs by default
- `Completed` rows should be hidden from briefs by default

The brief output should group by trip first, then list segments chronologically.

Example brief grouping:

1. Trip header
2. Date sections
3. Chronological entries with segment type, time, place, and confirmation details

## Bidirectional Sync Requirements

### Google Calendar

- Itinerary rows must be able to sync bidirectionally with Google Calendar when explicitly marked for calendar sync.
- Sync must support separate targets for Jeremy and Lindsay.
- Both Jeremy's and Lindsay's calendars should be connected.
- Calendar routing should use fixed calendar IDs stored in Settings.
- Calendar-originated changes should update the sheet.
- Sheet-originated changes should create or update the correct calendar event.
- Deleted calendar events should not silently orphan sheet rows.
- When the same shared event diverges across calendars or sheet state, last modified timestamp should win.
- `Draft` itinerary rows should never sync to calendar until they move to a non-draft status.
- `Cancelled` itinerary rows should remove any synced calendar events.
- `Completed` itinerary rows should leave past synced calendar events in place.

### Google Tasks

- `Dates` task rows must be able to sync bidirectionally with Google Tasks.
- Each task should sync to one accountable owner at a time.
- Secondary involvement can be tracked in the sheet, but should not create a second Google Task copy in v1.
- Task routing should use fixed Google Task list IDs stored in Settings.
- Because Google Tasks is user/list based rather than truly co-owned, a single-owner model should be the default for execution accountability.
- Task-originated changes should update the sheet.
- Sheet-originated changes should create or update the correct owner's Google Task.
- Completed task state should reconcile cleanly in both directions.

### Sync Ownership

- The primary person on the row is the owner of the event.
- If only one person's name is present, sync only to that person's calendar.
- If both Jeremy and Lindsay are present on the row, the event should sync to both calendars.
- For tasks in `Dates`, one person should be the explicit accountable owner for Google Tasks sync.
- For tasks in `Dates`, secondary involvement should remain in the existing `Co-Owners` field.
- Initial implementation should explicitly support Jeremy and Lindsay rather than a generic unlimited-account sync framework.
- The longer-term design should remain extensible to more identities later.

## Validation Rules

- `Trip ID` and `Segment ID` must be generated if missing.
- `Trip ID` should be an auto-generated opaque ID.
- `Segment ID` should be an auto-generated opaque ID.
- `Segment Type` should use data validation.
- `Status` should use data validation.
- New itinerary rows should default `Status` to `Planned`.
- `Traveler` should ideally validate against `Identities`.
- `Sync Owner` should use data validation and initially allow `Jeremy` and `Lindsay`.
- New itinerary rows should default `Sync Owner` to the primary `Traveler`.
- `Sync Target` should use data validation.
- `Sync Target` for itinerary rows should allow only `None` or `Calendar`.
- `Sync Target` should default to `None` for non-calendar itinerary items unless explicitly marked for event sync.
- Flights should not sync to calendar by default; they should sync only when explicitly marked with `Sync Target = Calendar`.
- Meals should be brief-only by default and should sync to calendar only when explicitly marked.
- Any itinerary segment type may sync to calendar if explicitly marked with `Sync Target = Calendar`.
- `Brief Include` should use `Yes` or `No`.
- New itinerary rows should default `Brief Include` to `Yes`.
- Start date is required.
- Trip name is required.
- Segment type is required.

## Apps Script Changes

Minimum code work expected:

- add `ITINERARY_COLS` constants
- add `ensureItineraryTab(ss)` setup helper
- include tab creation and formatting in `setup` and upgrade paths
- add itinerary-aware read helper such as `getItineraryData(ss)`
- add brief builder support for itinerary rows
- add traveler-to-calendar resolution for calendar-synced itinerary rows
- add `Dates` task owner-to-task-list resolution for Jeremy and Lindsay
- add bidirectional sync support for calendar-synced itinerary rows
- add sorting / validation reset logic for the new tab

## Live Script Touchpoints

The current bound Apps Script already has patterns that the implementation should extend rather than replace:

- `setup`
- upgrade / migration helpers such as `ensureIdentitiesTab`, `ensureOrdersTab`, and related setup functions
- calendar helpers such as `createCalendarEvent`, `updateCalendarEvent`, `deleteCalendarEvent`, `syncCalendarToSheet`, and `syncBidirectional`
- brief / email builders that already separate per-person relevance

The itinerary implementation should follow those existing patterns:

- add `ensureItineraryTab(ss)`
- add `getItineraryData(ss)`
- add itinerary-to-calendar sync helpers that parallel the existing calendar helpers
- extend the current summary/brief builder rather than creating a separate briefing system

## Required Settings Additions

The current Settings model should be extended with fixed IDs for external sync routing:

- `Jeremy Calendar ID`
- `Lindsay Calendar ID`
- `Jeremy Task List ID`
- `Lindsay Task List ID`

These should live in the existing Settings tab rather than a new configuration tab for v1.

## Implementation Notes

- `Sync Target` should default to `None`.
- `Sync Owner` should default to the primary `Traveler`.
- Shared itinerary rows should be one row, not duplicated rows.
- Shared calendar sync should write to both configured calendars only when both Jeremy and Lindsay are present on the row.
- Non-calendar items should remain brief-only unless explicitly opted into calendar sync.

## Backward Compatibility

- Existing tabs and workflows must continue to work unchanged.
- Existing daily summary behavior should not break if the `Itinerary` tab is empty.
- Upgrade logic should create the tab safely for current users.
- Existing current calendar sync should not regress while the system moves from one shared-calendar assumption to Jeremy/Lindsay-specific sync targets.
- Shared event conflict resolution should use last-modified-wins semantics.
- Existing `Dates` task behavior should not regress while task sync moves toward explicit single-owner accountability.

## Acceptance Criteria

- A new `Itinerary` tab can be created by setup/upgrade.
- Users can enter structured itinerary rows for different travelers and trips.
- The rows sort cleanly as a time series.
- Trip rows can be filtered per traveler.
- Brief generation can include itinerary rows in chronological order.
- Calendar-synced itinerary rows can sync bidirectionally with the correct Jeremy or Lindsay calendar.
- Shared calendar-synced itinerary rows with both Jeremy and Lindsay present can sync to both calendars.
- Shared trips for Jeremy and Lindsay work without requiring duplicate manual entry of the same itinerary segment.
- `Dates` task rows can sync bidirectionally to one owner's Google Tasks while preserving accountability in the sheet.
- No existing `Dates`, `Orders`, `Identities`, or Calendar workflows regress.

## Open Questions

- None for v1 scope.
