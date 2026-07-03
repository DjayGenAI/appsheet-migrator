---
name: powerplatform-build
description: "Build a Power Platform app (Dataverse + Canvas/Model-driven Power Apps + Power Automate) from a migration blueprint. Use when: implementing the Power Platform path, scaffolding a Dataverse solution, creating tables and columns, authoring Power Fx, building canvas screens, converting AppSheet WorkflowRules to Power Automate flows, building the migrated app low-code."
argument-hint: "Path to spec/migration-blueprint.md"
---

# Power Platform Build

Implement the app described in `spec/migration-blueprint.md` on the Power Platform. Work the build backlog in dependency order and produce source artifacts that can be imported into a Dataverse environment.

## Inputs

- `spec/migration-blueprint.md` — backlog + acceptance criteria (source of truth).
- `spec/app-inventory.json` — machine-readable inventory.

## Verify First

Before implementing any component, confirm the current capability/limitation via `migration-research-protocol` (Microsoft Learn MCP). Never guess connector limits, Power Fx signatures, or delegation behavior.

## Build Order

1. **Solution** — create a Dataverse solution to hold all components (use `pac` CLI where available; otherwise emit solution source under `build/powerplatform/`).
2. **Data** — Dataverse tables, columns, keys, relationships, choices from the inventory. Migrate Google Sheets/external data to Dataverse or Dataflows.
3. **UI** — canvas screens (galleries + forms) or model-driven forms/views matching AppSheet views.
4. **Logic** — Power Fx for computed columns, valid-if/show-if, actions. Map AppSheet expressions per the blueprint mapping.
5. **Automation** — Power Automate cloud flows replacing WorkflowRules (email, notifications, webhooks, scheduled).
6. **Security** — Dataverse security roles + row-level rules replicating AppSheet UserRoles.
7. **Advanced** — AI Builder, Copilot Studio, maps, offline where required.

## Output

- Source/artifacts under `build/powerplatform/`.
- A `build/powerplatform/BUILD-REPORT.md` mapping each completed backlog task ID to what was built.

## Handoff

When the backlog is complete, the build is ready for the **Power Platform Tester** to validate against the blueprint acceptance criteria.
