---
name: dotnet-build
description: "Build a .NET / Azure application from a migration blueprint. Use when: implementing the Pro Dev path, scaffolding an ASP.NET Core / Blazor / MAUI solution, modeling data with EF Core + Azure SQL, implementing business logic and screens, replacing AppSheet automations with backend services, building the migrated app in code."
argument-hint: "Path to spec/migration-blueprint.md"
---

# .NET Build

Implement the app described in `spec/migration-blueprint.md` using the target .NET stack named in the blueprint. Work the build backlog in dependency order with a test-first mindset.

## Inputs

- `spec/migration-blueprint.md` — chosen stack + backlog + acceptance criteria.
- `spec/app-inventory.json` — machine-readable inventory.

## Verify First

Confirm current API/SDK guidance via `migration-research-protocol` (Microsoft Learn MCP) before using a package, Azure service, or framework feature.

## Build Order

1. **Solution scaffold** — create the solution under `build/dotnet/` for the blueprint stack:
   - Web app: ASP.NET Core API + Blazor (or React) frontend.
   - Mobile/field: .NET MAUI.
   - Hybrid: Azure Functions backend + Power Apps front (coordinate with the Power Platform builder).
2. **Data** — EF Core model from the inventory; migrations against Azure SQL (or chosen store). Import existing data.
3. **Domain/logic** — translate AppSheet expressions/actions into services; keep behavior parity.
4. **UI** — screens matching AppSheet views (list/detail/form/map/dashboard).
5. **Integrations & automations** — background jobs / services replacing WorkflowRules (email, notifications, webhooks).
6. **Security** — Entra ID auth + authorization policies replicating AppSheet UserRoles / row-level rules.
7. **Advanced** — Azure AI, maps, offline sync where required.

## Output

- Solution under `build/dotnet/` that builds (`dotnet build`) and runs.
- `build/dotnet/BUILD-REPORT.md` mapping each completed backlog task ID to code produced.

## Handoff

When the backlog is complete and the solution builds, hand off to the **.NET Tester** to validate against the blueprint acceptance criteria.
