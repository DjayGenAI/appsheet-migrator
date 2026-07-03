---
name: appsheet-migration-interview
description: "Conduct a structured, one-topic-at-a-time discovery interview to inventory an AppSheet app when no JSON export is available. Use when: no AppSheet JSON export provided, gathering AppSheet app requirements by interview, building a migration inventory from user answers, AppSheet discovery questionnaire, interviewing a customer about their AppSheet app."
argument-hint: "None — run interactively when no JSON export exists"
---

# AppSheet Migration Interview

Conduct a structured discovery interview to build a migration inventory equivalent to what the `appsheet-analyzer` skill produces from a JSON export. Use this when the user has **no AppSheet JSON export**.

## Core Rule: One Topic at a Time

Never dump all questions at once. Ask one topic, wait for the answer, ask a focused follow-up if needed, then move to the next topic. Confirm your understanding before moving on.

---

## Interview Topics (in order)

### B1 — App Overview (ask first)
> "Let's start with the basics. In 2-3 sentences, what does this app do, who uses it, and how often?"

After response, ask a follow-up if needed, then move to B2.

### B2 — Data
> "How many different data entities (tables) does the app work with? For example: orders, customers, inspections, products..."
> "Where does that data live today — Google Sheets, a database, an API, or something else?"

### B3 — Users and Scale
> "Roughly how many users will use the Microsoft version of this app?"
> "Do they need to work offline — no internet connection — and if so, how much data do they need offline?"

### B4 — Screens and Interactions
> "What are the main screens in the app? (e.g., list of records, detail view, a form to add/edit, a map, a dashboard...)"
> "Are there any barcode scanning, image capture, signature, or camera features?"

### B5 — Automations
> "Does the app send emails, notifications, or trigger actions automatically when data changes? If so, describe the most complex one."

### B6 — Integrations
> "Does the app connect to any external systems — SAP, Salesforce, custom APIs, SMS, or anything else outside AppSheet?"

### B7 — Security
> "How is access controlled? Can all users see all data, or does each user see only their own records / their team's records?"
> "Does your organization already use Microsoft 365 / Azure?"

### B8 — Special Features
> "Does the app use any AI features — like scanning documents, predicting values, or a chatbot?"
> "Any features that felt like AppSheet 'magic' that you're worried won't exist in Microsoft tools?"

### B9 — Team & Constraints
> "Who will build the Microsoft version — a business analyst / maker, or a professional developer team?"
> "Is there a timeline or budget constraint I should factor in?"

---

## After the Interview

Synthesize the answers into a structured inventory equivalent to the `appsheet-analyzer` output (data layer, UI layer, actions, automations, security, advanced features, expression complexity). Then hand off to the `appsheet-migration-advisor` skill to run the decision tree.
