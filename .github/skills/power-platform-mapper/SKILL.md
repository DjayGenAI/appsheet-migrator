---
name: power-platform-mapper
description: "Map Google AppSheet components to Microsoft Power Platform equivalents. Use when: mapping AppSheet tables to Dataverse, mapping AppSheet views to Power Apps screens, mapping AppSheet formulas to Power Fx, mapping AppSheet automations to Power Automate, mapping AppSheet security to Dataverse security roles, translating AppSheet features to Power Platform, gap analysis AppSheet Power Platform."
argument-hint: "AppSheet inventory from appsheet-analyzer skill"
---

# Power Platform Mapper

Translates every Google AppSheet component to its Microsoft Power Platform equivalent and identifies gaps requiring workarounds or pro dev solutions.

## When to Use

- As Phase 2 of an AppSheet migration workflow
- After the `appsheet-analyzer` skill has produced the app inventory
- When you need a feature-by-feature mapping with gap analysis

---

## Mapping Reference

### Data Layer Mappings

| AppSheet Concept | Power Platform Equivalent | Notes |
|-----------------|--------------------------|-------|
| Table (Google Sheets backed) | Dataverse table (standard or custom) | Full equivalent with richer type system |
| Table (Cloud SQL backed) | Dataverse virtual table OR Azure SQL with connector | Consider data residency |
| Table (BigQuery backed) | Dataverse virtual table via custom connector or Azure Synapse | More complex |
| Table (REST API backed) | Power Apps custom connector + virtual table | |
| Column — Text | Dataverse Text column | Max length 4000 chars (single line) or unlimited (multiline) |
| Column — Number (Integer) | Dataverse Whole Number | |
| Column — Number (Decimal) | Dataverse Decimal Number or Float | |
| Column — Price | Dataverse Currency | Supports multi-currency |
| Column — Date | Dataverse Date Only | |
| Column — DateTime | Dataverse Date and Time | |
| Column — Yes/No | Dataverse Yes/No (Boolean) | |
| Column — Enum (single) | Dataverse Choice (local option set) | |
| Column — Enum (multi) | Dataverse Choices (multi-select option set) | |
| Column — Ref (to another table) | Dataverse Lookup column | 1:N relationship |
| Column — List (multi-ref) | Dataverse N:N relationship table | Requires junction table |
| Column — Image | Dataverse Image or File column | Stored in Azure Blob via Dataverse |
| Column — File | Dataverse File column | |
| Column — LatLong | Dataverse custom text column + map control OR Bing Maps | No native LatLong type |
| Column — Address | Dataverse Text + Power Apps Address input control | |
| Column — Email | Dataverse Email column | |
| Column — Phone | Dataverse Phone column | |
| Column — URL | Dataverse URL column | |
| Column — Computed (formula) | Dataverse calculated column OR Power Fx formula in canvas app | Calculated columns run server-side |
| Key column (UUID) | Dataverse Primary Key (auto-generated GUID) | |
| Table Slice (filter) | Dataverse View (server-side filter) OR Power Apps Gallery filter | |
| TableSlice (security filter) | Dataverse security role + row-level security | |

### UI Layer Mappings

| AppSheet View Type | Power Platform Equivalent | Gap Notes |
|-------------------|--------------------------|-----------|
| Table view | Power Apps Data Table control OR Gallery (vertical) | Data Table is read-only; Gallery needed for interaction |
| Gallery view | Power Apps Flexible-height Gallery | Full equivalent |
| Detail view | Power Apps Display Form | Full equivalent |
| Form view (add/edit) | Power Apps Edit Form | Full equivalent |
| Deck view | Power Apps Gallery with custom card layout | |
| Map view | Power Apps Map control (Bing Maps) | ⚠️ Replaces Google Maps; geofencing limited |
| Chart view | Power Apps Charts / Power BI embedded | Power BI recommended for complex charts |
| Calendar view | Power Apps Calendar component (community) OR SharePoint | ⚠️ No native calendar control in canvas apps |
| Dashboard view | Multiple screens + navigation OR Power BI dashboard | |
| Onboarding view | Power Apps sequence of screens with Next button | Manual implementation |
| Inline detail (nested view) | Power Apps nested Gallery | Limited nesting depth |
| Grouped view | Power Apps Gallery with GroupBy | |
| Card layout | Power Apps custom Gallery card template | |
| Scan view (QR/barcode) | Power Apps BarcodeScanner control | ✅ Full equivalent |

### Action Mappings

| AppSheet Action | Power Platform Equivalent | Gap Notes |
|----------------|--------------------------|-----------|
| App: go to view | Power Apps Navigate() function | ✅ |
| App: pop view | Power Apps Back() function | ✅ |
| Data: add row | Power Apps Patch() or SubmitForm() | ✅ |
| Data: set column values | Power Apps Patch() with specific fields | ✅ |
| Data: delete row | Power Apps Remove() | ✅ |
| Data: execute action on a set | Power Apps ForAll() with Patch/Remove | ⚠️ Delegation concerns at scale |
| External: call a URL | Power Apps Launch() function | ✅ |
| External: call a webhook | Power Automate cloud flow triggered from Power Apps | |
| External: call a REST API | Power Apps direct HTTP connector OR Power Automate | |
| Grouped action | Power Apps multiple Patch() calls in sequence | |
| Action on form save | Power Apps OnSuccess of SubmitForm | ✅ |
| Conditional action | Power Apps If() in button's OnSelect | ✅ |
| Action with confirmation prompt | Power Apps Confirm() | ✅ |
| Offline action (queue) | ⚠️ Limited — Power Apps offline sync has constraints | See offline section |

### Automation Mappings

| AppSheet Automation | Power Platform Equivalent | Gap Notes |
|--------------------|--------------------------|-----------|
| Workflow rule: when row added | Power Automate: Dataverse "When a row is added" trigger | ✅ |
| Workflow rule: when row changed | Power Automate: Dataverse "When a row is modified" trigger | ✅ |
| Workflow rule: when row deleted | Power Automate: Dataverse "When a row is deleted" trigger | ✅ |
| Workflow rule: scheduled | Power Automate: Recurrence trigger | ✅ |
| Workflow rule: webhook trigger | Power Automate: HTTP trigger | ✅ |
| Action: send email | Power Automate: Send an email (Office 365 / SendGrid) | ✅ |
| Action: send push notification | Power Automate: Push notification connector | ✅ |
| Action: call webhook | Power Automate: HTTP action | ✅ |
| Action: set column values | Power Automate: Update a row (Dataverse) | ✅ |
| Action: add row | Power Automate: Add a new row (Dataverse) | ✅ |
| Action: delete row | Power Automate: Delete a row (Dataverse) | ✅ |
| AppProcess (multi-step) | Power Automate flow with multiple steps | ✅ |
| Scheduled report | Power Automate + Power BI subscription OR custom flow | |
| SMS notification | Power Automate: Twilio connector | ⚠️ Third-party required |
| Conditional branching | Power Automate: Condition / Switch actions | ✅ |
| Loop over rows | Power Automate: Apply to each (⚠️ throttling at scale) | |

### Security Model Mappings

| AppSheet Security Feature | Power Platform Equivalent | Gap Notes |
|--------------------------|--------------------------|-----------|
| Auth required: Google | Entra ID (Azure AD) authentication | Migration of user accounts needed |
| Auth required: Microsoft | Entra ID | ✅ Direct equivalent |
| Email domain restriction | Entra ID Conditional Access policies | ✅ |
| User roles (table CRUD) | Dataverse security roles with table privileges | ✅ |
| Row-level filter by user email | Dataverse row-level security (record ownership / teams) | ⚠️ Different model — plan carefully |
| Column-level security | Dataverse field-level security profiles | ✅ |
| Read-only for some roles | Dataverse security role with Read only, no Write | ✅ |
| Visibility rules (show/hide column) | Power Apps Visible property with User().Email | ✅ |

### Expression / Formula Mappings

| AppSheet Expression | Power Fx Equivalent | Notes |
|--------------------|--------------------|-------|
| CONCATENATE(a, b) | Concatenate(a, b) or a & b | ✅ |
| IF(cond, t, f) | If(cond, t, f) | ✅ |
| TODAY() | Today() | ✅ |
| NOW() | Now() | ✅ |
| YEAR/MONTH/DAY(date) | Year()/Month()/Day() | ✅ |
| DATEDIF(d1, d2, "D") | DateDiff(d1, d2, Days) | ✅ |
| UPPER/LOWER(text) | Upper()/Lower() | ✅ |
| LEN(text) | Len(text) | ✅ |
| SUBSTITUTE(text, old, new) | Substitute(text, old, new) | ✅ |
| LEFT/RIGHT/MID | Left()/Right()/Mid() | ✅ |
| CONTAINS(text, sub) | "sub" in text or Search() | ✅ |
| IN(val, list) | val in list or CountIf()>0 | ✅ |
| ISBLANK(val) | IsBlank(val) | ✅ |
| NOT(expr) | Not(expr) or !expr | ✅ |
| AND(a, b) / OR(a, b) | And()/Or() or &&/\|\| | ✅ |
| LOOKUP(row, table, key, col) | LookUp(Table, Key=row, col) | ✅ |
| SELECT(table, filter) | Filter(Table, condition) | ✅ — delegation rules apply |
| FILTER(table, condition) | Filter(Table, condition) | ✅ — delegation limit: 500/2000 rows |
| COUNT(list) | CountRows(list) | ✅ |
| SUM(select(...)) | Sum(Filter(...), col) | ✅ — delegation applies |
| MAX/MIN(select(...)) | Max()/Min() over Filter() | ✅ |
| ORDERBY(list, col, asc) | SortByColumns(list, col, Ascending) | ✅ |
| MAXROW(table, key, filter) | LookUp(Filter(Table, cond), ...) | ⚠️ More verbose |
| ANY(list) | First(Filter(list, ...)) | ⚠️ Semantics differ |
| LIST(a, b, c) | [a, b, c] table literal | ⚠️ Limited in Power Fx |
| UNIQUE(list) | Distinct(list, col) | ✅ |
| USEREMAIL() | User().Email | ✅ |
| USERNAME() | User().FullName | ✅ |
| USERROLE() | No direct equivalent — use Security roles + Dataverse lookup | ⚠️ Requires custom role table |
| CONTEXT("View") | No direct equivalent | ❌ Architecture change needed |
| CONTEXT("Device") | Param("device") or environment checks | ⚠️ Limited |
| INDEX(list, n) | Index(list, n) | ✅ |
| SPLIT(text, delim) | Split(text, delim) | ✅ |
| ISPARTOF(a, b) | b in a or Search() | ⚠️ |
| REF_ROWS(table, refcol) | Filter(Table, RefCol=ThisItem.ID) | ✅ |
| IFS(c1,v1,c2,v2,...) | Switch() or nested If() | ✅ |
| SWITCH(val, c1,v1,...) | Switch(val, c1, v1, ...) | ✅ |
| ISNOTBLANK(val) | !IsBlank(val) | ✅ |
| FLOOR/CEILING/ROUND | RoundDown()/RoundUp()/Round() | ✅ |
| RANDBETWEEN | RandBetween() | ✅ |
| TEXT(val, format) | Text(val, format) — format codes differ | ⚠️ |
| VALUE(text) | Value(text) | ✅ |
| TIMENOW() | TimeValue(Now()) | ✅ |
| LINKTOFORM(...) | Navigate() with EditForm context | ⚠️ Different model |
| LINKTOFILTEREDVIEW(...) | Navigate() + set filter variable | ⚠️ Requires app variable management |

### Advanced Feature Mappings

| AppSheet Feature | Power Platform Equivalent | Gap Notes |
|----------------|--------------------------|-----------|
| Predictive model (custom ML) | AI Builder — custom prediction model | ⚠️ Retraining required, different platform |
| OCR model | AI Builder — Document processing / Form Recognizer | ✅ Full equivalent |
| AppBot / chatbot | Copilot Studio (Power Virtual Agents) | ✅ |
| Map / geolocation | Power Apps Map control (Bing Maps) | ⚠️ Google Maps not available |
| Full offline mode | Power Apps offline (limited) | ⚠️ Major gap — see offline analysis |
| Barcode/QR scanner | Power Apps BarcodeScanner control | ✅ |
| Signature capture | Power Apps PenInput control | ✅ |
| Image annotation | ❌ No native equivalent | Pro Dev or Azure AI Vision |
| Push notifications | Power Automate notifications connector | ✅ |
| Deep links | Power Apps parameter passing | ⚠️ Limited |
| Google Sheets data source | SharePoint Lists OR Dataverse | Migration required |
| Google Drive file storage | SharePoint / OneDrive / Azure Blob | Migration required |
| BigQuery data source | Azure Synapse / Fabric OR virtual tables | Architecture change |
| Webhook inbound | Power Automate HTTP trigger | ✅ |
| REST API integration | Power Platform custom connector | ✅ |
| SMS (via Twilio in AppSheet) | Power Automate Twilio connector | ✅ |

### Offline Mode Analysis

AppSheet has strong offline capabilities. Power Platform offline is limited:

| Capability | AppSheet | Power Apps | Gap |
|-----------|----------|-----------|-----|
| Full offline database | ✅ | ⚠️ Collections only (in-memory) | Major gap |
| Auto background sync | ✅ | ⚠️ Manual Collect() required | Significant effort |
| Conflict resolution | ✅ | ❌ Manual implementation needed | Complex |
| Offline form submit | ✅ | ✅ SaveData/LoadData | Partial |
| Offline images/files | ✅ | ⚠️ Limited | |
| Local record count | Unlimited | ~500-2000 recommended | Constraint |

**Offline score**: If the app uses `FullOfflineCaching: true` or `LaunchOffline: true`, flag this as a **High complexity / potential Pro Dev** area.

---

## Mapping Procedure

### Step 1 — Run the mapping

For each component in the AppSheet inventory:
1. Find the Power Platform equivalent from the tables above
2. Assign a mapping confidence: ✅ Full match / ⚠️ Partial match / ❌ No equivalent
3. Note any constraints (delegation, record limits, retraining required)
4. Estimate implementation effort: Low / Medium / High

### Step 2 — Validate with Microsoft Learn

For any uncertain mapping, use `mcp_microsoft_lea_microsoft_docs_search` to confirm:
- Current Power Fx function availability
- Dataverse column type constraints
- Power Automate connector support
- AI Builder model types and limitations
- Power Apps control capabilities

### Step 3 — Produce Mapping Report

Output a mapping table per layer (Data, UI, Actions, Automations, Security, Expressions, Advanced), then a gap summary:

```markdown
## Gap Summary

### Blockers (❌) — require architecture decision
- [list]

### Significant gaps (⚠️) — workaround needed
- [list]

### Migration complexity by area
| Area | Score | Key reasons |
|------|-------|-------------|

### Overall feasibility for Power Platform
[High / Medium / Low / Not Recommended]
```
