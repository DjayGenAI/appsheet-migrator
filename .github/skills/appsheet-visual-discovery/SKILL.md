---
name: appsheet-visual-discovery
description: "Capture the visual and interaction design of a LIVE AppSheet app by driving it with the Playwright MCP. Use when: the JSON export lacks UX fidelity, capturing AppSheet screens/screenshots, walking an AppSheet app's navigation, inventorying views/forms/actions visually, producing a design spec for the canvas app rebuild, complementing the AppSheet JSON analysis with real UI evidence."
argument-hint: "The running AppSheet app URL (e.g. https://www.appsheet.com/start/<app-id>) and the role/test account to use"
---

# AppSheet Visual Discovery (Playwright MCP)

Capture the **visual and interaction design** of a running AppSheet app by driving it with the **Playwright MCP** (`playwright` server, tools prefixed `mcp_playwright_browser_`). This is the source-side counterpart to the Canvas Authoring MCP on the target side.

## Why this exists

Google exposes **no** API for an AppSheet app's UX/definition (verified: the public AppSheet API is data-CRUD + admin/monitoring only). The JSON export captures the *logic* (tables, expressions, view configs) but **not** what the app actually looks like or feels like to use. Playwright captures the *rendered* app — layout, theme, navigation flow, form structure, conditional formatting — so the rebuild can match the original design.

- The JSON export answers **"how does it work?"**
- Visual discovery answers **"what does it look like and how do you move through it?"**

Both are needed for a faithful rebuild. This skill produces the **design** half.

## When to Use

- **Optional, opt-in** — only when the user has a running app, can log in, and wants design fidelity.
- Best run **after** `appsheet-analyzer` (so you know the view names to expect) and **before** the blueprint (so the design spec feeds the build).
- Skip entirely if the user only wants a logic/functional migration, or has no access to a running instance.

## Consent & Login Gate (MANDATORY)

Before touching the browser:

1. **Ask for opt-in** — confirm the user wants to use the Playwright MCP for visual discovery. If they decline, stop and note that the rebuild will rely on the JSON export + interview only (lower design fidelity).
2. **Set expectations about login** — tell the user plainly:
   > "AppSheet apps sit behind Google sign-in. I'll open the app in a browser window, then **pause and hand it to you** so you can log in yourself. I never see or handle your credentials. Once you're in and looking at the app's home screen, tell me to continue."
3. **Never handle credentials** — do not type passwords, request them, or automate the Google login. The interactive login is always the user's action.
4. **Ask which role/account** — conditional formatting, security filters, and visible rows depend on the logged-in user's role. Ask which role/test account best represents the app so coverage is representative, and record it in the spec.

## Preflight

- Confirm the `playwright` MCP server is connected (its `mcp_playwright_browser_*` tools are listed). If not, tell the user to start the `playwright` server in `.vscode/mcp.json` (it runs via `npx @playwright/mcp@latest`, so Node.js is the only prerequisite) — do not proceed without it.
- Ask for the app URL (typically `https://www.appsheet.com/start/<app-id>`).

## Procedure

### Step 1 — Open & authenticate

1. `browser_navigate` to the app URL.
2. **Pause** and hand the browser to the user for Google login (per the login gate above).
3. When the user confirms they are logged in and on the app home screen, continue.

### Step 2 — Capture the entry state

1. `browser_snapshot` — capture the accessibility tree (roles, labels, text) of the home view. This is structured, not just pixels.
2. `browser_take_screenshot` — save to `spec/visual-discovery/00-home.png`.
3. Identify the primary navigation (bottom nav bar, hamburger menu, tabs) from the snapshot.

### Step 3 — Walk every view systematically

For each top-level view (cross-reference the view list from `appsheet-analyzer` so nothing is missed):

1. `browser_click` the nav entry for the view (use `browser_wait_for` to let it render).
2. `browser_snapshot` + `browser_take_screenshot` → `spec/visual-discovery/<nn>-<view-name>.png`.
3. Record: view type (table / deck / gallery / detail / form / map / chart / dashboard), layout, which columns/fields are shown, sort/group, and any visible action buttons and their placement.

### Step 4 — Traverse into detail, form, and actions

For representative rows:

1. Open a **detail** view (`browser_click` a row) → snapshot + screenshot.
2. Open an **Add/Edit form** → snapshot + screenshot; record field order, grouping, required markers, and input types.
3. Enumerate **action buttons** (row actions, detail actions, view actions) — capture label, icon, and position. Do **not** trigger destructive actions (delete, submit that writes data) unless the user explicitly asks; prefer read-only navigation.

### Step 5 — Capture design tokens & dynamic behavior

- Note theme: primary color, light/dark, fonts, iconography, header style.
- Capture any **conditional formatting** (color-coded rows, highlighted cells), charts, and maps as screenshots — these only exist in the rendered app.
- Note anything role-dependent you can see, and flag what your account **cannot** see (so coverage gaps are honest).

## Output — write to `spec/`

Per the handoff-contract standard, all evidence goes to the shared `spec/` folder:

- `spec/visual-discovery/*.png` — one screenshot per view / detail / form.
- `spec/visual-discovery/ux-inventory.md` — the design catalog:
  - App-level design tokens (theme color, fonts, nav pattern).
  - The logged-in role/account used and its coverage limits.
  - A per-view table: **view name** (matched to the JSON export) · view type · layout · fields/columns shown · actions · nav position · screenshot filename.
  - A **navigation map** (which view leads to which — a small Mermaid `flowchart` is ideal).
  - An honest **gaps** section: views/states not captured and why.

## Handoff — feeds the build (design purpose)

This spec is a **design source of truth for the canvas app rebuild**:

- The **`migration-blueprint`** skill references `spec/visual-discovery/` so design intent is part of the contract.
- The **`powerplatform-build`** skill uses the screenshots + `ux-inventory.md` as the **visual target** when authoring canvas screens through the Canvas Authoring MCP — matching layout, navigation, theme, and form structure to the original, not just replicating logic.
- Where a screenshot and the JSON export disagree, the screenshot wins for **appearance/interaction**; the JSON export wins for **logic/expressions**.

## Guardrails

- Read-only by default — navigate and observe; never write data or fire destructive actions without explicit user approval.
- Credentials are always the user's to enter; the agent never sees them.
- If the app requires MFA or a session times out, pause and hand the browser back to the user.
- Cite the screenshot/snapshot behind every design claim in the inventory (per AGENTS.md evidence standard).

## Communication

Follow the shared conventions in [AGENTS.md](../../../AGENTS.md): one topic at a time, cite the specific screenshot/snapshot behind each design point, use ✅ / ⚠️ / ❌ confidence markers, be honest about coverage gaps, and end every message with a clear next step.
