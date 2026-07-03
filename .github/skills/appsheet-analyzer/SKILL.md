---
name: appsheet-analyzer
description: "Parse and catalog a Google AppSheet app JSON export. Use when: analyzing AppSheet JSON structure, cataloging AppSheet tables columns views actions automations expressions security AI features, inventorying AppSheet app for migration assessment, understanding AppSheet app complexity."
argument-hint: "Path to the AppSheet JSON export file"
---

# AppSheet App Analyzer

Parses a Google AppSheet application JSON export and produces a full feature inventory used as input for migration assessment.

## When to Use

- When a user provides an AppSheet JSON export file
- As Phase 1 of an AppSheet migration workflow
- When you need to understand what an AppSheet app does before proposing a migration

---

## AppSheet JSON Structure

AppSheet exports follow this top-level structure:

```
Root
├── Template
│   ├── Id, ShortName, Name, Title, Description
│   ├── Category, Department, Industry
│   ├── Presentation          ← UI theme, fonts, layout settings
│   ├── AppData
│   │   ├── DataSchemas       ← Table definitions (columns, types, keys, formulas)
│   │   ├── DataSets          ← Data source connections (Google Sheets, SQL, etc.)
│   │   ├── DataActions       ← Data-level actions (add/edit/delete row triggers)
│   │   ├── TableSlices       ← Filtered views of tables
│   │   └── ExpressionSettings← App-wide expression configuration
│   ├── Behavior
│   │   ├── Views             ← UI screens (table, gallery, detail, form, deck, map, chart, dashboard)
│   │   ├── Actions           ← User-triggered actions (navigate, add row, call URL, etc.)
│   │   ├── WorkflowRules     ← Automation rules (send email, push notification, webhook)
│   │   ├── AppBots           ← Chatbot / AI assistant configuration
│   │   ├── AppPredictiveModels← ML prediction features
│   │   ├── AppOcrModels      ← OCR / document scanning features
│   │   ├── AppEvents         ← App lifecycle events
│   │   ├── AppProcesses      ← Multi-step automation processes
│   │   ├── AuthProvider      ← Authentication method
│   │   ├── AuthRequired      ← Whether login is required
│   │   ├── AuthDomain        ← Allowed email domains
│   │   ├── UsesGeolocation   ← GPS/location features
│   │   ├── FullOfflineCaching← Offline mode configuration
│   │   └── ExternalServiceSettings ← External API integrations
│   └── UserRoles             ← Role-based access control definitions
```

---

## Procedure

### Step 1 — Load the File

Read the AppSheet JSON export file with `read_file`. If the path is not provided, ask the user for it.

### Step 2 — Extract App Metadata

From `Template`:
- App name, title, description, category, department, industry
- Date created, last modified, version
- Platform version

### Step 3 — Catalog Data Layer

From `Template.AppData.DataSchemas`:
For each table:
- Table name and key column
- Column list: name, type (Text, Number, Date, Image, File, LatLong, Enum, Ref, List, etc.), formula if any
- Record count estimates (if available)
- Whether it has ref relationships to other tables

From `Template.AppData.DataSets`:
- Data source type (Google Sheets, Cloud SQL, BigQuery, Salesforce, REST API, etc.)
- Connection details (sanitized — no credentials)

From `Template.AppData.TableSlices`:
- Slice name, parent table, filter condition

### Step 4 — Catalog UI Layer

From `Template.Behavior` (Views section):
For each view:
- View name, type (Table, Gallery, Detail, Form, Deck, Map, Chart, Calendar, Dashboard, Onboarding)
- Parent table/slice
- Sort/filter/group settings
- Conditional formatting rules
- Inline actions attached

### Step 5 — Catalog Actions

From `Template.Behavior.Actions`:
For each action:
- Action name, type (App: navigate, Data: add/edit/delete row, External: call URL/webhook, Grouped)
- Trigger type (user tap, on create, on change, on save)
- Condition expression
- Target

### Step 6 — Catalog Automations

From `Template.Behavior.WorkflowRules` and `AppProcesses`:
For each automation:
- Trigger: event type (adds, updates, deletes row, scheduled, webhook)
- Condition expression
- Actions: email, push notification, webhook, run process, set column values
- Number of steps

### Step 7 — Catalog Security

From `Template.Behavior` (auth fields) and `Template.UserRoles`:
- Auth required (yes/no)
- Auth provider (Google, Microsoft, custom)
- Auth domain restrictions
- User roles: name, table-level permissions (read/write/delete/create filter expressions)
- Row-level security expressions

### Step 8 — Catalog Advanced Features

From `Template.Behavior`:
- **AI/ML**: AppPredictiveModels (type, table, target column), AppOcrModels
- **Chatbot**: AppBots (configuration details)
- **Geolocation**: UsesGeolocation, map views
- **Offline**: FullOfflineCaching, LaunchOffline, DeltaSync, IncrementalSync
- **External integrations**: ExternalServiceSettings, GoogleMapsKey, any webhook URLs

### Step 9 — Catalog Expressions

Collect all formula expressions from:
- Column computed formulas
- Valid-if, required-if, show-if conditions
- Slice filter conditions
- Action conditions
- Security filter expressions

Classify expression complexity:
- **Simple**: CONCATENATE, IF, TODAY, NOW, UPPER, LOWER, basic arithmetic
- **Medium**: LOOKUP, SELECT, FILTER, MAXROW, MINROW, ORDERBY, IN, CONTAINS
- **Complex**: Nested SELECT/FILTER, LIST operations, ANY, ALL, COUNT, UNIQUE, multi-table lookups
- **Very Complex**: Recursive refs, deep nesting >4 levels, procedural-style logic

### Step 10 — Produce Inventory Report

Output a structured catalog:

---

## Step 11 — Validate Completeness & Challenge Gaps

JSON exports routinely omit context that changes the migration assessment. After cataloging, check for each of these gaps and challenge the user on every one found before proceeding to mapping:

| What to check | Why it matters | Question to ask |
|---------------|---------------|-----------------|
| `DataSets` lists Google Sheets or external DBs | Tells you data migration complexity | "Your app reads from [source]. Who owns that data today and is there a plan to move it to Microsoft?" |
| `FullOfflineCaching: true` or `LaunchOffline: true` | Offline is a major Power Platform constraint | "The app is configured for full offline use. How many records are expected offline per user, and is conflict resolution needed?" |
| `AppPredictiveModels` or `AppOcrModels` present | Custom AI may not map to AI Builder | "The app uses [model type]. Can you describe what it predicts / reads? Was the model trained on your own data?" |
| `WorkflowRules` with webhooks to external URLs | Hidden integrations not in the JSON | "I see webhook calls to external systems. Can you list every system this app integrates with, beyond what's in the JSON?" |
| `AuthProvider` is Google | User account migration required | "Authentication is Google. Does your organization already have Microsoft 365 / Entra ID licenses for all app users?" |
| `ExternalServiceSettings` present | Third-party connectors needed | "The app uses external services. Are any of these business-critical with SLAs or custom APIs?" |
| `AppBots` present | Copilot Studio migration needed | "There's a chatbot configured. What does it do — FAQ, guided data entry, something else?" |
| `UsesGeolocation: true` | Map provider changes (Google → Bing) | "The app uses maps/location. Does it need Google Maps specifically, or is Bing Maps acceptable?" |
| Views with `Chart` or `Calendar` type | Limited native Power Apps equivalents | "There are chart/calendar views. Are these simple visualizations or is Power BI-level analytics needed?" |
| `UserRoles` with complex filter expressions | Row-level security requires design | "User roles have security filters. Can you walk me through who sees what — is it based on email, team, location, or something else?" |

Only proceed to the mapping phase after the user has answered the critical (gap) questions.

```markdown
## AppSheet App Inventory

### Metadata
- Name: ...
- Category: ... | Department: ... | Industry: ...
- Tables: N | Views: N | Actions: N | Automations: N | Roles: N

### Data Layer
| Table | Columns | Key | Relationships | Data Source | Complexity |
|-------|---------|-----|---------------|-------------|------------|

### UI Layer  
| View | Type | Table/Slice | Features | Complexity |
|------|------|-------------|----------|------------|

### Actions
| Action | Type | Trigger | Condition | Complexity |
|--------|------|---------|-----------|------------|

### Automations
| Automation | Trigger | Steps | Actions | Complexity |
|------------|---------|-------|---------|------------|

### Security Model
- Auth: ...
- Roles: ...
- Row-level security: Yes/No

### Advanced Features
- AI/ML: ...
- Offline: ...
- Geolocation: ...
- External integrations: ...

### Expression Complexity Summary
- Simple: N | Medium: N | Complex: N | Very Complex: N
- Most complex expressions: [list top 3-5 with context]

### Overall Complexity Score
[Simple / Moderate / Complex / Very Complex]
```
