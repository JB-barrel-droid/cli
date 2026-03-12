# Personal Date Tracker Itinerary Feature Spec

## Status
- State: Blocked for implementation in this workspace.
- Last reviewed: 2026-03-12 (America/Chicago) / 2026-03-12T23:50:45Z.
- Blocking reasons:
  - No existing Personal Date Tracker itinerary source files are present in this repository.
  - Rust toolchain is unavailable in this environment (`cargo`/`rustc` not installed), so compile/test verification cannot run.

## Feature Goal
Deliver an itinerary workflow that converts date-plan inputs into a structured itinerary record, keeps a user-facing brief in sync, and synchronizes calendar events safely and reversibly.

## Functional Requirements
1. Schema/setup and migrations:
- Introduce itinerary domain schema with explicit versioning.
- Add forward migration and tested rollback migration.
- Ensure idempotent setup for fresh and partially configured states.

2. Brief generation and calendar sync:
- Generate a deterministic itinerary brief from canonical itinerary data.
- Sync calendar entries from itinerary segments with stable external IDs.
- Support update and delete flows without duplicating calendar events.

3. Validation, docs, and rollback safety:
- Validate user inputs (dates, times, timezone, location, attendee/resource names).
- Add branch and rollback procedures for all risky operations.
- Provide tests for happy paths and rejection paths.

## Non-Functional Requirements
- Revertability: every milestone captured by a standalone commit on a `codex/*` branch.
- Safety: reject malformed or ambiguous temporal inputs.
- Traceability: each implementation decision recorded in `learnings.md`.

## Acceptance Criteria
- `plan.md` contains exactly three active workstreams.
- Each workstream has concrete deliverables and validation steps.
- Blockers and next actions are explicit and time-stamped.
- Latest safe rollback commit hash is recorded every run.
