---
name: appsheet-migration-advisor
description: "Run the interactive AppSheet migration decision tree and produce a final recommendation. Use when: scoring AppSheet migration complexity, deciding between Power Platform vs Pro Dev vs Hybrid, walking the eight complexity gates, generating an AppSheet migration scorecard, recommending a migration path, producing an AppSheet migration assessment report."
argument-hint: "AppSheet inventory from appsheet-analyzer skill or Path B interview answers"
---

# AppSheet Migration Advisor Skill

## Purpose

Run the interactive migration decision tree for an AppSheet app. Score eight complexity gates, walk the user through each one conversationally, and produce a final migration recommendation: **Power Platform (PP)**, **Pro Dev (.NET/Azure) (PD)**, or **Hybrid (HY)**.

This skill is always invoked after the app inventory is complete — either from a JSON parse or from the structured interview in Path B.

---

## Inputs

The following inputs must be available before this skill runs. If any are missing, ask for them explicitly before scoring.

| Input | Source | Ask if missing |
|-------|--------|---------------|
| Table count | JSON `AppData.DataSchemas` or interview | "How many distinct tables/entities does the app have?" |
| Expression complexity summary | appsheet-analyzer output or interview | "Are the app's formulas simple lookups and filters, or do they involve conditional branching, loops, or complex business rules?" |
| Automation count + complexity | JSON `Behavior.WorkflowRules` or interview | "How many automated workflows exist? Describe the most complex one." |
| Offline requirement | JSON `FullOfflineCaching` / `LaunchOffline` | "Does the app work offline? If so, how much data per user?" |
| Integration count + types | JSON `ExternalServiceSettings` or interview | "How many external system integrations are there?" |
| User roles / security model | JSON `UserRoles` or interview | "How is row-level access controlled — who sees what?" |
| AI / ML features | JSON `AppPredictiveModels`, `AppOcrModels` | "Does the app use AI features — predictions, OCR, image recognition?" |
| Approximate concurrent user count | Interview | "How many concurrent users will use this app?" |
| Builder team profile | Interview | "Who will build the Microsoft version — a maker/analyst or a professional developer team?" |

---

## The Eight Gates

Score each gate independently. Apply the rules below and show the user the result after each gate.

### Gate 1 — Data Complexity

**Evidence to check**: table count, relationships, key column types, use of virtual columns with complex formulas.

| Condition | Score |
|-----------|-------|
| ≤ 10 tables, mostly flat, simple lookups | Low (0) |
| 11–20 tables, some relationships, moderate computed columns | Medium (1) |
| 21–30 tables, multi-level relationships, many virtual columns | High (2) |
| > 30 tables, or deep relational hierarchies, complex key logic | Blocker (3) |
| Uses BigQuery or Cloud SQL with complex custom SQL | Blocker (3) |

**Question to confirm**:
> "I've counted [N] tables. Are there additional tables that might not be in the JSON, or any external views/queries treated as tables?"

---

### Gate 2 — Business Logic Complexity

**Evidence to check**: virtual column expressions, AppSheet expression types found, number of Complex/Very Complex expressions from the analyzer.

| Condition | Score |
|-----------|-------|
| All expressions are lookups, concatenations, simple arithmetic | Low (0) |
| Some IFS/SWITCH, moderate FILTER/SELECT expressions | Medium (1) |
| Heavy use of REF_ROWS, nested SELECT, compound FILTER, computed keys | High (2) |
| State machine logic, looping patterns (ALL, ANY over large sets), ORDERBY + SLICE combos at scale | Blocker (3) |

**Question to confirm**:
> "Are there any business rules that involve multi-step calculations, iterating over related rows, or conditional logic that spans multiple tables? Even if you can't describe them technically, a business example helps."

---

### Gate 3 — Offline Requirements

**Evidence to check**: `FullOfflineCaching`, `LaunchOffline`, `OfflineAccess` in table definitions.

| Condition | Score |
|-----------|-------|
| Online-only app | Low (0) |
| Light offline (cache read-only reference data) | Low (0) |
| Offline with basic sync (< 500 records per user, no conflicts expected) | Medium (1) |
| Offline with full CRUD and auto-sync on reconnect | High (2) |
| Full offline with conflict resolution, merge logic, or large datasets (> 5,000 records per user) | Blocker (3) |

**Power Apps offline reality check** (always share this when Gate 3 ≥ Medium):
> "I want to be transparent about Power Apps offline: it can cache data locally using the `SaveData`/`LoadData` functions, but conflict resolution is manual — you write the merge logic yourself in Power Fx. It does NOT automatically sync like AppSheet does. For [user's scenario], that means [specific implication]."

**Question to confirm**:
> "When a field user is offline and edits a record, and someone else edits the same record online — what should happen? Who wins, or is there a merge step?"

---

### Gate 4 — Integration Complexity

**Evidence to check**: `ExternalServiceSettings`, webhook URLs in `WorkflowRules`, `DataSets` pointing to external APIs.

| Condition | Score |
|-----------|-------|
| No external integrations | Low (0) |
| 1–2 integrations with standard connectors (M365, SharePoint, Exchange) | Low (0) |
| 2–4 integrations including at least one non-standard connector | Medium (1) |
| 5+ integrations, or real-time bidirectional sync with external systems | High (2) |
| Custom protocols, streaming data, sub-second latency SLA, or legacy system with no REST API | Blocker (3) |

**Question to confirm**:
> "For each external system integration: Is it read-only, write-only, or bidirectional? Is it batch (e.g., nightly) or real-time (e.g., on every save)?"

---

### Gate 5 — Security Model

**Evidence to check**: `UserRoles` definitions, filter expressions on roles, `AuthProvider`, `AuthRequired`, `AuthDomain`.

| Condition | Score |
|-----------|-------|
| No authentication, or all users see all data | Low (0) |
| Entra ID / M365 auth, basic role separation (admin vs user) | Low (0) |
| Row-level security based on user email or a single field | Medium (1) |
| Multi-dimensional RLS (team + location + role) | High (2) |
| Dynamic data masking, field-level security, or compliance requirements (HIPAA, SOC2, GDPR) | High (2) |
| Complex multi-tenant architecture with per-tenant data isolation | Blocker (3) |

**Question to confirm**:
> "Walk me through a concrete example: User A logs in — what can they see, what can they edit, and what is hidden from them? Now do the same for User B with a different role."

---

### Gate 6 — AI / Advanced Features

**Evidence to check**: `AppPredictiveModels`, `AppOcrModels`, `AppBots`, `UsesGeolocation`, barcodes in column types.

| Condition | Score |
|-----------|-------|
| No AI, no OCR, no barcode scanning, no geolocation | Low (0) |
| Basic barcode scanning or geolocation (GPS capture) | Low (0) |
| OCR (document scanning) | Medium (1) |
| Pre-built prediction models (AppSheet AutoML) | Medium (1) |
| Custom-trained ML models on proprietary data | High (2) |
| Chatbot / AppBot with custom NLP | High (2) |
| Real-time image classification at scale, or AI that AppSheet AutoML trained on >10k rows | Blocker (3) |

**AI Builder reality check** (share when Gate 6 ≥ Medium):
> "AI Builder in Power Platform covers: form processing (OCR), object detection, text classification, prediction models, and document intelligence. Custom models trained from scratch on your own data require exporting training data and rebuilding in AI Builder or Azure AI Studio. [Specific feature] maps to [specific AI Builder capability / gap]."

**Question to confirm** (if custom models found):
> "For the [model name] prediction model — what data was it trained on, and how accurate does it need to be? Do you have the training data available to retrain in AI Builder?"

---

### Gate 7 — User Scale & Performance

**Evidence to check**: User interview, `UserRoles` count, any hints about record volume in data sources.

| Condition | Score |
|-----------|-------|
| < 25 concurrent users | Low (0) |
| 25–100 concurrent users | Low (0) |
| 100–500 concurrent users | Medium (1) |
| 500–2,000 concurrent users | High (2) |
| > 2,000 concurrent users or sub-second SLA on large datasets | Blocker (3) |
| Heavy reporting / aggregation queries over > 100k rows in real time | Blocker (3) |

**Power Apps delegation warning** (always share when table > 500 rows or Gate 7 ≥ Medium):
> "Power Apps canvas apps have a delegation limit — by default, only the top 500 records are retrieved unless the data source supports delegation (Dataverse does, SharePoint partially does). For [user's scenario with N records], we need to confirm all filter/sort operations are delegable. I'll validate against the Microsoft docs."

**Question to confirm**:
> "What's the largest table in your app — approximately how many rows today, and how many in 2 years? Do users need to search or filter across all rows?"

---

### Gate 8 — Builder Team Profile

**Evidence to check**: User interview.

| Condition | Score |
|-----------|-------|
| Business analyst / power user / maker with no coding experience | Low → strongly favors PP |
| Mix of makers and developers | Neutral (0) |
| Small team of professional developers | Medium (1) → consider PD if other gates are also ≥ Medium |
| Large team of professional developers with DevOps/CI-CD requirements | High (2) → consider PD |
| Organization mandates code review, versioning, and testing pipelines | High (2) → PD strongly preferred |

**Note**: This gate adjusts the recommendation but cannot be a Blocker on its own.

---

## Scoring and Recommendation

### Calculate the Total Score

After all 8 gates, tally the scores:

```
Gate 1: Data Complexity           __ / 3
Gate 2: Business Logic            __ / 3
Gate 3: Offline                   __ / 3
Gate 4: Integration               __ / 3
Gate 5: Security                  __ / 3
Gate 6: AI / Advanced Features    __ / 3
Gate 7: Scale & Performance       __ / 3
Gate 8: Builder Team              __ / 2
─────────────────────────────────────────
Total Score:                      __ / 23
```

### Present the Scorecard

Display a visual scorecard to the user before announcing the recommendation:

```
╔══════════════════════════════════════════════════════════╗
║         APPSHEET MIGRATION SCORECARD                     ║
╠═══════════════════════╦══════════╦══════════════════════╣
║ Gate                  ║ Rating   ║ Key Finding          ║
╠═══════════════════════╬══════════╬══════════════════════╣
║ 1. Data Complexity    ║ [rating] ║ [1-line finding]     ║
║ 2. Business Logic     ║ [rating] ║ [1-line finding]     ║
║ 3. Offline            ║ [rating] ║ [1-line finding]     ║
║ 4. Integrations       ║ [rating] ║ [1-line finding]     ║
║ 5. Security           ║ [rating] ║ [1-line finding]     ║
║ 6. AI / Advanced      ║ [rating] ║ [1-line finding]     ║
║ 7. Scale / Perf       ║ [rating] ║ [1-line finding]     ║
║ 8. Builder Team       ║ [rating] ║ [1-line finding]     ║
╠═══════════════════════╬══════════╬══════════════════════╣
║ TOTAL                 ║ xx / 23  ║ Recommendation: PP / HY / PD ║
╚═══════════════════════╩══════════╩══════════════════════╝
```

### Apply the Recommendation Rules

**Power Platform (PP)** — recommend when:
- Total score ≤ 6 AND
- No gate is Blocker AND
- Gate 3 (Offline) ≤ Medium AND
- Gate 7 (Scale) ≤ Medium

**Hybrid (HY)** — recommend when:
- Total score 7–12 AND
- At most ONE Blocker (on Gate 4 or Gate 6 only — these can be handled by Azure Functions) AND
- Gates 1, 2, 3 are all ≤ High

**Pro Dev (PD)** — recommend when:
- Total score > 12, OR
- Any Blocker on Gates 1, 2, 3, 5, or 7, OR
- Two or more Blockers on any gates, OR
- Gate 8 = High and total score > 8

---

## Announcing the Recommendation

### Always confirm before the full output

> "Based on our analysis, the data points strongly toward [Power Platform / a Hybrid approach / a Pro Dev rebuild]. Before I generate the full plan, does that direction make sense to you, or is there anything in the scoring above that you'd like me to revisit?"

Wait for confirmation. If the user disputes a gate score, revisit it with new information, adjust, and re-score before continuing.

---

## Delivering the Recommendation

### Power Platform Plan

Structure:

**Architecture Overview (text diagram)**
```
[Power Apps Canvas App]
    │
    ├─ [Dataverse] ──── [Security Roles]
    │
    ├─ [Power Automate Flows]
    │       ├─ [Approval Flow]
    │       ├─ [Notification Flow]
    │       └─ [Integration Connector(s)]
    │
    ├─ [AI Builder] (if needed)
    │
    └─ [Copilot Studio] (if AppBot existed)
```

**Migration Phases**

| Phase | Milestone | Duration | Key Actions |
|-------|-----------|----------|-------------|
| 1. Foundation | Data in Dataverse | Week 1–2 | Export from Google Sheets → import to Dataverse, define column types, set up security roles |
| 2. Core App | MVP screens in Power Apps | Week 3–5 | Build main galleries, detail screens, and forms; replicate key formulas in Power Fx |
| 3. Automations | Flows in Power Automate | Week 6–7 | Rebuild WorkflowRules as cloud flows; test approval logic |
| 4. Security | Roles + RLS | Week 8 | Apply Dataverse security roles matching AppSheet UserRoles |
| 5. Advanced | AI, maps, integrations | Week 9–10 | AI Builder models, Bing Maps, external connectors |
| 6. Test & Go-Live | User acceptance | Week 11–12 | Parallel run, user training, cutover |

**Licensing recommendation**: List required licenses (Power Apps per user / per app, Dataverse, premium connectors, AI Builder credits).

**Top 3 risks** with mitigations.

---

### Pro Dev Plan

Structure:

**Why Power Platform won't work** — one paragraph citing the specific Blocker gates and what they mean for the user.

**Recommended Stack Decision**

Present the options and recommend:

| Scenario | Stack |
|----------|-------|
| Heavy data + complex API + enterprise scale | ASP.NET Core Web API + React frontend + Azure SQL + Azure AD B2C |
| Mobile-first field app needing native device features | .NET MAUI + Azure Mobile Apps SDK + Azure SQL |
| Mostly forms/workflows but too complex for low-code | Power Apps Model-Driven App + ASP.NET Core Azure Functions backend |
| Complex offline with sync + enterprise integrations | .NET MAUI or PWA (React) + Azure Cosmos DB (offline-sync) + Azure API Management |

**Migration Phases** (tailored to the chosen stack — 6–8 phases with effort sizing).

**What Power Platform CAN still handle** — list any subset of features that belong in the low-code layer of a hybrid.

**Effort estimate** (T-shirt sizes: XS/S/M/L/XL per phase, and total).

---

### Hybrid Plan

Structure:

**Split Architecture**

```
[Power Apps Canvas App] ──── [Custom Connector]
                                      │
                              [Azure Functions .NET]
                                      │
                              [Azure SQL / Dataverse]
```

**What goes low-code vs pro-code**

| Feature | Low-Code (Power Platform) | Pro-Code (Azure) |
|---------|--------------------------|-----------------|
| [Feature A] | ✅ Canvas app screen | — |
| [Feature B] | ✅ Power Automate flow | — |
| [Feature C] | — | ✅ Azure Function (complex logic) |
| [Feature D] | — | ✅ .NET service (real-time integration) |

**Integration pattern**: "The Power Apps canvas app calls the Azure Function via a custom connector registered in the Power Platform admin center. Auth flows through Entra ID using delegated permissions."

**Phase plan** combining both tracks.

---

## Output: Final Migration Assessment Report

After delivering the plan, always produce a written artifact the user can save:

```markdown
# AppSheet Migration Assessment Report
**App Name**: [app name]
**Assessment Date**: [date]
**Analyst**: AppSheet Migration Specialist (GitHub Copilot)

## Executive Summary
[1 paragraph: what the app does, what was analyzed, what the recommendation is and why]

## App Inventory Summary
| Component | Count | Complexity |
|-----------|-------|------------|
| Tables | N | Low/Med/High |
| Views | N | Low/Med/High |
| Automations | N | Low/Med/High |
| Expressions | N | Low/Med/High |
| Integrations | N | Low/Med/High |
| AI Features | N | Low/Med/High |

## Migration Scorecard
[Paste the scorecard table]

## Recommendation: [PP / HY / PD]
[Full recommendation text from above]

## Migration Plan
[Phase table]

## Risks & Mitigations
[Top 5 risks]

## Next Steps
1. [First concrete action the user/team should take]
2. [Second concrete action]
3. [Third concrete action]
```

---

## Guardrails

- Never finalize the recommendation without user confirmation of the scorecard
- If a gate score is disputed, re-examine with the new information and re-score
- If critical inputs are still missing at the end of the interview, state explicitly: "I cannot confidently score Gate [N] without knowing [X]. My estimate is [score], but please confirm before we proceed."
- Always validate Power Platform capabilities with Microsoft Learn MCP before citing them as supported or unsupported
- Never understate a risk — if there is genuine uncertainty, say "this is a risk area I'd want to prototype and test before committing"
