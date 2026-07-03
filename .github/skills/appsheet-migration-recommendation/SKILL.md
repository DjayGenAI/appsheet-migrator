---
name: appsheet-migration-recommendation
description: "Produce the final migration recommendation and phased plan after the decision-tree scorecard is known. Use when: recommending Power Platform vs Pro Dev vs Hybrid, building an AppSheet migration phase plan, choosing a .NET/Azure stack for a migrated AppSheet app, writing the target architecture and licensing needs, delivering a hybrid split architecture."
argument-hint: "The migration scorecard / gate results from appsheet-migration-advisor"
---

# AppSheet Migration Recommendation

Turn the decision-tree scorecard (from `appsheet-migration-advisor`) into a concrete recommendation and phased plan. Choose exactly one path based on the gate scores.

> Before asserting any capability, limitation, or feature equivalence, follow the `migration-research-protocol` skill (verify with Microsoft Learn MCP + Google/web, record to `spec/`).

---

## Path PP: Power Platform

**Recommend when** the scorecard shows Low-to-Medium across all gates with no Blockers.

Deliver:
1. **Target architecture diagram** (text-based): Power Apps canvas app → Dataverse → Power Automate + connectors
2. **Phase plan**:
   - Phase 1: Data migration (Google Sheets → Dataverse / SharePoint)
   - Phase 2: Core screens (main tables as canvas app with galleries + forms)
   - Phase 3: Automations (WorkflowRules → Power Automate cloud flows)
   - Phase 4: Security (Dataverse security roles replicating AppSheet user roles)
   - Phase 5: Advanced features (AI Builder, Copilot Studio, maps)
3. **Licensing needs** (per user / per app / premium connector requirements)
4. **Top 3 risks and mitigations**

---

## Path PD: Pro Dev (.NET / Azure)

**Recommend when** any gate scores Blocker, or when 3+ gates score High.

Deliver:
1. **Why Power Platform won't work** — one paragraph, plain language, referencing specific evidence
2. **Recommended stack** — choose the right one:
   - **Heavy data processing, complex logic**: ASP.NET Core API + React/Blazor frontend + Azure SQL
   - **Mobile-first field app**: .NET MAUI (cross-platform native) + Azure SQL + Azure API Management
   - **Mostly forms + workflows but too complex for low-code**: Power Apps model-driven + .NET Azure Function backend (hybrid)
   - **Fully custom**: Azure Container Apps + .NET API + React PWA
3. **What Power Platform CAN handle** — identify any subset of features that stay low-code
4. **Migration phases** tailored to the chosen stack
5. **Effort estimate** (T-shirt sizing: S/M/L/XL) per phase

---

## Path HY: Hybrid

**Recommend when** most gates are Low-Medium but 1-2 gates are High (not Blocker).

Deliver:
1. **Split architecture**: which parts go Power Platform, which parts go Azure Functions / .NET
2. **Integration pattern**: Power Apps calling Azure Functions via custom connector
3. **Decision rationale** per split point
4. **Phase plan** combining both tracks

---

## Output

Present the recommendation with cited evidence for each claim, then record the final recommendation and rationale to `spec/migration-research.md`.
