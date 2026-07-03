---
name: dotnet-validation
description: "Validate a .NET build against the migration blueprint acceptance criteria. Use when: testing a migrated .NET app, running unit/integration/E2E tests, verifying the build compiles and starts, checking EF Core data model and migrations, testing API endpoints and UI flows, producing a .NET test verdict."
argument-hint: "Path to spec/migration-blueprint.md and build/dotnet/"
---

# .NET Validation

Verify the .NET build satisfies every acceptance criterion in `spec/migration-blueprint.md`. Produce a clear pass/fail verdict with evidence.

## Inputs

- `spec/migration-blueprint.md` — acceptance criteria + backlog IDs.
- `build/dotnet/` — the solution and `BUILD-REPORT.md`.

## Checks

1. **Build** — `dotnet build` succeeds with no errors; report warnings.
2. **Unit tests** — run `dotnet test`; each acceptance criterion has at least one test. Add missing tests.
3. **Startup** — the app starts and health endpoint responds.
4. **Data** — EF Core migrations apply; model matches the inventory (tables, keys, relationships).
5. **API/logic** — integration tests cover endpoints and business rules; verify behavior parity with AppSheet expressions.
6. **UI / E2E** — for web apps, run a Playwright smoke test of the main flows (list → detail → create/edit → save).
7. **Security** — auth required where expected; row-level authorization enforced.

## Verdict

Write `spec/dotnet-test-report.md`:
- Per acceptance criterion: ✅ pass / ❌ fail / ⚠️ partial, with evidence (test names, output).
- Overall verdict: PASS / FAIL.
- If FAIL: a concrete, prioritized fix list keyed to backlog task IDs, for the builder to act on.

Return the verdict and the fix list so the builder can iterate.
