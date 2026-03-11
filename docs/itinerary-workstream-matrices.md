# Itinerary Workstream Matrices

## Workstream 1: Schema/Setup/Migrations
- Deliverable: migration checklist with rollback mapping.
- Validation: migration dry-run + rollback simulation tests.
- Risk: data loss or non-idempotent setup.

## Workstream 2: Brief + Calendar Sync
- Deliverable: deterministic brief/sync behavior contract.
- Validation: scenario tests for create/update/delete + DST/timezone edges.
- Risk: duplicate events or stale external IDs.

## Workstream 3: Validation/Tests/Docs/Rollback
- Deliverable: coverage checklist, docs sync, rollback hash per run.
- Validation: lint/tests + run report consistency.
- Risk: untracked decisions and unsafe rollback points.
