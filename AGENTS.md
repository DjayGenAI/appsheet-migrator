# AGENTS.md — Repository Standards

Shared conventions for all agents, skills, and prompts in this workspace. Individual agent files should focus on their persona and orchestration and inherit the standards below rather than restating them.

## Communication Principles (standard)

- **One topic at a time** — never overwhelm the user with multiple questions in one message.
- **Show your work** — always cite the specific source (JSON field, user answer, or documentation URL) that drives each assessment point.
- **Use visual scorecards** — prefer ASCII tables for structured comparisons and decision gates.
- **Be honest about gaps** — never hand-wave a ❌ gap; always explain the workaround or why an alternative is needed.
- **Use confidence markers** — ✅ / ⚠️ / ❌ for mapping and capability confidence throughout.
- **End every message with a clear next step** so the user always knows what to do.
- **Be concise** — skip filler; lead with the answer, then the supporting detail.

## Evidence Standard

- Ground factual claims about external services (Microsoft, Google, AppSheet, etc.) in a live source before presenting them; do not rely on trained knowledge alone.
- Record verified findings to the workspace `spec/` folder so they can be reused across sessions.

## Agent Design Standard

- Keep agents **lean**: an agent file is persona + orchestration (who it is, when to call what). Put the substance in skills, not in the agent.
- Prefer **many small skills** over one large one; prefer a prompt file for a user-initiated one-shot command.
- Agents hand work to other agents via `runSubagent`; keep each agent single-purpose.

## Handoff Contract Standard

- Multi-agent work flows through a shared artifact in `spec/`, not by re-deriving context.
- The orchestrator writes `spec/migration-blueprint.md` (chosen path, feature mapping, ordered build backlog, per-task acceptance criteria) and `spec/app-inventory.json`.
- Builder agents implement the backlog into `build/<target>/` and write a `BUILD-REPORT.md` keyed to backlog task IDs.
- Tester agents verify against the blueprint acceptance criteria and write `spec/<target>-test-report.md` with a PASS/FAIL verdict and a fix list keyed to the same task IDs.
