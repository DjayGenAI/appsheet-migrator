---
name: migration-research-protocol
description: "Enforce evidence-based verification for every Microsoft/AppSheet/Google capability claim, handle MCP availability, and persist findings to a spec folder. Use when: verifying a Power Platform or Azure capability, confirming an AppSheet or Google feature, asserting feature equivalence, Microsoft Learn MCP not available, checking whether a comparable Google service exists, recording migration research findings, building a reusable spec folder."
argument-hint: "The capability, limitation, or feature-mapping claim to verify"
---

# Migration Research Protocol (MANDATORY)

Do NOT rely on internal/trained knowledge alone for any factual claim about a Microsoft service capability, a Microsoft service limitation, an AppSheet feature, a Google service, or the equivalence between an AppSheet/Google feature and a Microsoft feature. Every such claim must be grounded in a live source before you present it.

## Verification Workflow (every factual claim)

1. **Check `spec/` first** — if a recent verified finding already covers the topic, reuse it and tell the user you're drawing on the stored spec instead of re-querying.
2. **Validate the Microsoft side** with the Microsoft Learn MCP (`mcp_microsoft_lea_microsoft_docs_search`, then `mcp_microsoft_lea_microsoft_docs_fetch` for depth). This is required — never skip it.
3. **Validate the Google / AppSheet side** — if a Google MCP is available, use it. Otherwise perform a **web search** (`fetch_webpage` against official Google/AppSheet documentation) to confirm the source feature behaves as assumed, and to check whether a comparable Google-side service exists that changes the recommendation.
4. **Cross-check equivalence** — only after confirming both sides may you assert that feature X in AppSheet maps to feature Y in Microsoft. State confidence (✅ / ⚠️ / ❌) and cite the source.
5. **Record the finding** to the `spec/` folder (see below).

When a live check changes your assessment, tell the user explicitly:

> "I just checked the current Microsoft documentation on [topic] and found [finding]. This changes my earlier assessment of [area] — [updated assessment]. I've recorded this in `spec/migration-research.md`."

---

## MCP Availability Enforcement

- Before the first factual claim in a session, **confirm the Microsoft Learn MCP is reachable** by issuing a real query. If it fails or is unavailable:
  - **Tell the user explicitly**: "The Microsoft Learn MCP is not available / not responding. I cannot verify Microsoft capabilities without it."
  - **Help re-establish the connection**: tell the user to check that the Microsoft Learn MCP server is configured and running (VS Code: check the MCP server list / restart the server), and offer to retry once they confirm.
  - **Do not fabricate** verified claims while the MCP is down. Proceed only with clearly-labeled unverified assumptions, and revisit them once the MCP is restored.
- If a **Google MCP is not available**, say so, then fall back to a web search for the Google/AppSheet side rather than guessing.

---

## Persist Findings to a Spec Folder

Every verified finding (Microsoft Learn result, web search result, or feature-mapping decision) MUST be recorded to a `spec/` folder in the workspace so it can be reused as a reference:

- Create the folder `spec/` at the workspace root if it does not exist.
- Append findings to `spec/migration-research.md` (never overwrite prior findings). For each entry record: the topic/question, the source (Microsoft Learn URL or web URL), the date, the verified finding, and how it affects the recommendation.
- Optionally maintain `spec/feature-mapping.md` for the running AppSheet→Microsoft mapping table.
- If the user has not confirmed a workspace location, ask where the `spec/` folder should live before writing.

### Suggested entry format for `spec/migration-research.md`

```markdown
## [Topic / question]
- **Date:** YYYY-MM-DD
- **Microsoft source:** <Microsoft Learn URL>
- **Google/AppSheet source:** <URL>
- **Finding:** <verified fact>
- **Confidence:** ✅ / ⚠️ / ❌
- **Impact on recommendation:** <how this changes the assessment>
```
