---
name: powerplatform-validation
description: "Validate a Power Platform build against the migration blueprint acceptance criteria. Use when: testing a migrated Power Platform app, verifying Dataverse tables and relationships, checking canvas/model-driven screens, testing Power Automate flows, running solution checker, confirming security roles, producing a Power Platform test verdict."
argument-hint: "Path to spec/migration-blueprint.md and build/powerplatform/"
---

# Power Platform Validation

Verify that the Power Platform build satisfies every acceptance criterion in `spec/migration-blueprint.md`. Produce a clear pass/fail verdict with evidence.

## Inputs

- `spec/migration-blueprint.md` — acceptance criteria + backlog IDs.
- `build/powerplatform/` — the build artifacts and `BUILD-REPORT.md`.

## Checks

1. **Solution health** — run Power Platform Solution Checker (via `pac solution check` where available); report critical/high issues.
2. **Data** — every blueprint table/column/relationship exists with correct types and keys.
3. **UI** — each mapped screen exists and binds to the right table; required fields and views present.
4. **Logic** — Power Fx behaves per acceptance criteria (spot-check computed columns, valid-if, actions).
5. **Automation** — each flow triggers and completes; test the most complex flow end-to-end.
6. **Security** — roles enforce the intended row-level visibility.
7. **Delegation & limits** — confirm no unexpected delegation warnings on large tables (verify via `migration-research-protocol`).

## Verdict

Write `spec/powerplatform-test-report.md`:
- Per acceptance criterion: ✅ pass / ❌ fail / ⚠️ partial, with evidence.
- Overall verdict: PASS / FAIL.
- If FAIL: a concrete, prioritized fix list keyed to backlog task IDs, for the builder to act on.

Return the verdict and the fix list so the builder can iterate.
