---
name: migration-blueprint
description: "Author the handoff contract that downstream builder and tester agents consume. Use when: producing the migration blueprint after path validation, writing the build backlog and acceptance criteria, packaging the app inventory + chosen path + feature mapping for handover to a builder agent, defining the spec that a Power Platform or .NET builder will implement."
argument-hint: "The validated recommendation + app inventory"
---

# Migration Blueprint (Handoff Contract)

The single artifact that lets the orchestrator hand work to a builder agent automatically. Produced **after** the user validates the recommended path. Written to `spec/migration-blueprint.md` (+ `spec/app-inventory.json`).

## When to Use

- After `appsheet-migration-recommendation` and user sign-off on the path.
- Before invoking any builder agent — builders and testers read this contract, not the raw AppSheet JSON.

## Required Contents of `spec/migration-blueprint.md`

1. **App summary** — name, purpose, users, scale.
2. **Chosen path** — `PowerPlatform` | `ProDev` | `Hybrid`, with one-line rationale and the target stack (e.g., Canvas + Dataverse + Power Automate, or ASP.NET Core + Blazor + Azure SQL).
3. **Feature mapping table** — each AppSheet feature → target component, with confidence (✅ / ⚠️ / ❌) and source citation (from `spec/migration-research.md`).
4. **Build backlog** — ordered, ID'd tasks grouped by phase (data → UI → logic → security → advanced). Each task has: `ID`, `title`, `depends-on`, `target component`.
5. **Acceptance criteria** — per backlog task, a testable statement the tester agent will verify (Given/When/Then style).
6. **Open risks & assumptions**.

## Companion File: `spec/app-inventory.json`

The structured inventory from `appsheet-analyzer` (tables, columns, views, actions, automations, security, expressions) so builders have machine-readable source-of-truth.

## Rule

Every claim in the blueprint about a target capability must already be verified per `migration-research-protocol`. Do not invent mappings here — reference the recorded findings.
