---
name: generate-startix-audit-report
description: 'Read-only architecture audit for the Startix codebase (an AI analyst that monitors users'' stock portfolios and generates insights/reports): maps every major business and technical flow as it exists right now, compares it against the simplest reliable best-practice design, and scores current vs ideal. Publishes the result as an artifact and ends with a prioritized P0/P1/P2 change list ready to hand to /implement. Never modifies code. Use whenever the user wants an architecture audit, sanity check, or health-check of Startix, or invokes /generate-startix-audit-report directly — this is meant to be re-run periodically as the codebase evolves, not a one-off.'
---

# /generate-startix-audit-report

A read-only architecture audit. It never edits code — only reads, reasons, and reports.

Treat every run as standing on its own: analyze the codebase exactly as it exists right now. Don't assume anything carried over from a prior audit still holds, and don't let this file's wording go stale — the point is that this same prompt produces a fresh, accurate audit no matter when it's run or how much the code has changed since the last time.

## 0. How to work

- Use read-only exploration (Explore agent, `Read`, `Grep`, `find`) across backend, frontend, database/schema, jobs/workers, queues, schedulers, integrations, and AI/LLM components. For a codebase this broad, delegate sections to parallel Explore/general-purpose subagents rather than reading everything serially.
- Every claim in the CURRENT-state sections must be evidence-based — cite the file (and line, where useful) that supports it. If you can't find evidence for how something works, say so explicitly rather than guessing.
- Do not assume the current architecture is correct, intentional, or well-documented. Verify by reading code, not by trusting comments, READMEs, or naming.
- Do not modify any code, config, or infrastructure. This is audit and redesign only.

## 1. Understand the system

Startix is an AI analyst that monitors users' stock portfolios and generates insights/reports. Identify the major flows, especially:

- User onboarding
- Authentication / account creation
- Portfolio creation/import
- Connecting or syncing brokerage/market data
- Portfolio updates
- Market-data ingestion
- Event generation and event processing
- Stock/portfolio monitoring
- AI analysis
- Alerts / notifications
- Scheduled reports
- On-demand reports / analysis
- User interactions with reports/insights
- Background jobs
- Scheduled jobs
- Failure/retry flows
- Data refresh / synchronization
- User preferences
- Subscription/billing-related flows, if present

If the actual codebase has flows beyond this list, include them too — this list is a floor, not a ceiling.

## 2. Map the CURRENT flows (FACT)

For each major flow, give a short flow diagram, e.g.:

```
User signs up
→ Create account
→ Create portfolio
→ Initial data sync
→ Generate initial analysis
→ Schedule monitoring
→ Generate events
→ AI analyzes event
→ Notify user
```

For every flow, identify: trigger, main steps, services/components involved, database state changes, async/background work, external APIs, AI/LLM calls, and output to the user. Keep it concise and scannable — this is a map, not a narrative.

## 3. Design the IDEAL architecture (RECOMMENDATION)

Independent of how it's actually built, design the simplest architecture you'd want for this kind of product. Prioritize, in order: simplicity, reliability, clear ownership of responsibilities, idempotency, observability, ease of debugging, and scalability only where actually necessary. Do not over-engineer.

For each major flow, show the ideal version. Pay particular attention to the spine of the system:

**Data → Events → Analysis → Insights → Notifications → Reports**

State explicitly what should be synchronous vs asynchronous, and where jobs/queues/schedulers should exist (and where they shouldn't).

## 4. Compare CURRENT vs IDEAL

| Flow | Current | Ideal | Problem | Recommended Change |
| ---- | ------- | ----- | ------- | ------------------- |

Focus on meaningful architectural differences, not code-level nitpicks (that's what `/code-review` and `/simplify` are for).

## 5. Best parts (FACT)

The 3–7 strongest architectural decisions currently in Startix, and briefly why each is good. Don't pad this list if fewer than 3 genuinely qualify.

## 6. Worst parts (FACT)

The 3–7 weakest architectural decisions/flows. For each: why it's problematic, what failure mode it creates, what complexity it introduces, and what you'd replace it with. Be critical — don't praise something just because it already exists, and don't manufacture weaknesses if the system is genuinely solid in an area.

## 7. Prioritized changes (RECOMMENDATION)

### P0 — Fix now
Critical architectural problems (data loss, race conditions, silent failures, security).

### P1 — Improve soon
Important but not blocking.

### P2 — Later
Nice improvements, scalability, optimization.

For each change, describe the desired flow or architecture in 1–3 sentences.

## 8. Final architecture (RECOMMENDATION)

End with a very simple high-level architecture diagram, adapted to what you actually found — e.g.:

```
User → API → Core domain → Database

Market Data → Ingestion → Events → Analysis Engine → Insights ┬→ Notifications
                                                                └→ Reports

Scheduler → Jobs → Analysis / Reports
```

## 9. Unknowns / confidence

List anything you couldn't verify with confidence (code you didn't have access to, behavior that depends on external config/secrets, ambiguous ownership) rather than presenting a guess as fact.

## 10. Publish the artifact

Load the `artifact-design` skill, then publish the audit as a single HTML artifact (title it something like "Startix Architecture Audit"). Structure it to mirror sections 1–9 above — use tables for section 4, and render the flow diagrams from sections 2, 3, and 8 as Mermaid flowcharts (Artifacts render these natively) rather than plain text arrows. Clearly visually distinguish FACT sections (2, 5, 6) from RECOMMENDATION sections (3, 7, 8) — e.g. a consistent label or accent color per type, defined in the light/dark palette per the artifact-design skill.

## 11. Hand off to implementation

After publishing, tell the user the artifact is ready and explicitly suggest running `/implement` against the P0 list to start fixing the highest-priority items — don't run `/implement` yourself, just point at it with the P0 items as the natural input.
