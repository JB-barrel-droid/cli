# Itinerary Implementation Plan

Last updated: 2026-03-11 (America/Chicago)

## Overall Status
- Phase: Supervision and readiness hardening.
- Delivery state: Not implementation-ready due to missing feature sources and missing Rust toolchain.

## Active Workstreams (exactly three)

### 1) Schema/Setup/Migrations (Active)
Scope:
- Define target itinerary schema entities and migration boundaries.
- Prepare migration runbook with forward + rollback checkpoints.

Current tasks:
- Create migration checklist and failure-mode matrix.
- Identify required fixture data for migration validation.

Exit criteria:
- Migration checklist approved and rollback procedure documented.
- Validation commands identified for when toolchain is restored.

### 2) Brief + Calendar Sync Behavior (Active)
Scope:
- Define deterministic brief render contract.
- Define calendar sync idempotency contract (create/update/delete).

Current tasks:
- Enumerate edge cases: timezone shifts, DST transitions, partial itinerary edits.
- Define expected behavior for sync conflicts and stale external IDs.

Exit criteria:
- Behavior contract documented with examples.
- Verification scenarios mapped to automated tests.

### 3) Validation/Tests/Docs/Rollback Safety (Active)
Scope:
- Keep spec/plan/learnings current.
- Maintain rollback-safe commit points and run verification checklist.

Current tasks:
- Track environment blockers and unblock prerequisites.
- Maintain concise run reports and rollback hash lineage.

Exit criteria:
- Test plan ready to execute once tooling is available.
- No undocumented decisions across run artifacts.

## Blockers
1. No Personal Date Tracker itinerary implementation files found in this workspace.
2. `cargo` and `rustc` unavailable, preventing compile/lint/test execution.

## Highest-Value Next Actions
1. Restore or mount the repository/worktree containing itinerary source files.
2. Install Rust toolchain in this environment.
3. Execute migration, behavior, and validation tasks in the three workstreams above.
