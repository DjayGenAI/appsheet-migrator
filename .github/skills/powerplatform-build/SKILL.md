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
- `spec/visual-discovery/` *(optional)* — screenshots + `ux-inventory.md` from the `appsheet-visual-discovery` skill. When present, this is the **design target** for the canvas UI: match the original app's layout, navigation pattern, theme (color/fonts), and form structure — not just its logic. Where a screenshot and the JSON export disagree, the screenshot wins for **appearance/interaction**; the export wins for **logic/expressions**.

## Required Tooling — Canvas Authoring MCP (MANDATORY)

Canvas screens **must** be authored, compiled, and validated through the **Canvas Authoring MCP** (`canvas-authoring` server, tools prefixed `canvasauthori`). Do not hand-write `.pa.yaml` blind and hope it compiles.

- **Preflight**: before building any canvas UI, confirm the MCP is connected (`connect`, `list_data_sources`, `list_controls`). If it is not available, **stop** and tell the user to install it (see setup below) — do not fall back to guessing YAML.
- **Author → compile → sync**: for every screen, use `compile_canvas` to validate Power Fx and `get_appchecker_errors` / `get_accessibility_errors` to catch issues before handing to the tester. Use `describe_control` / `describe_api` instead of guessing control properties or Power Fx signatures.

### Setup (install on the tester's machine)

The MCP is declared in `.vscode/mcp.json` as the `canvas-authoring` server. It runs via `dnx` (the .NET tool runner), so the only prerequisite is the .NET SDK that ships `dnx`:

1. Install the **.NET 10 SDK** (or later) — `dnx` ships with it. Verify with `dnx --version`.
2. Open the repo in VS Code, open `.vscode/mcp.json`, and **Start** the `canvas-authoring` server (or reload the window). First run restores `Microsoft.PowerApps.CanvasAuthoring.McpServer` from NuGet automatically (`--yes --prerelease`).
3. Confirm the `canvasauthori` tools are listed, then run `connect` to attach to the target Power Apps environment.

## Verify First

Before implementing any component, confirm the current capability/limitation via `migration-research-protocol` (Microsoft Learn MCP). Never guess connector limits, Power Fx signatures, or delegation behavior.

## Build Order

1. **Solution** — create a Dataverse solution to hold all components (use `pac` CLI where available; otherwise emit solution source under `build/powerplatform/`).
2. **Data** — Dataverse tables, columns, keys, relationships, choices from the inventory. Migrate Google Sheets/external data to Dataverse or Dataflows.
3. **UI** — canvas screens (galleries + forms) or model-driven forms/views matching AppSheet views. Author every canvas screen through the **Canvas Authoring MCP** (compile + app-check each screen); never emit unvalidated `.pa.yaml`. If `spec/visual-discovery/` exists, use its screenshots + `ux-inventory.md` as the visual target — reproduce the original layout, navigation order, theme, and form field grouping, not just the data bindings.
4. **Logic** — Power Fx for computed columns, valid-if/show-if, actions. Map AppSheet expressions per the blueprint mapping.
5. **Automation** — Power Automate cloud flows replacing WorkflowRules (email, notifications, webhooks, scheduled).
6. **Security** — Dataverse security roles + row-level rules replicating AppSheet UserRoles.
7. **Advanced** — AI Builder, Copilot Studio, maps, offline where required.

## Output

- Source/artifacts under `build/powerplatform/`.
- A `build/powerplatform/BUILD-REPORT.md` mapping each completed backlog task ID to what was built.

## Handoff

When the backlog is complete, the build is ready for the **Power Platform Tester** to validate against the blueprint acceptance criteria.
