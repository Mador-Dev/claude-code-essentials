---
name: implement-auto-improve
description: 'Run the full /implement workflow unattended: build the plan, self-review and improve it, approve it without waiting for the user, and carry the implementation through to final validation. Use when the user asks to implement something without being asked to approve the plan — e.g. "/implement-auto-improve", "just implement it, don''t ask me", "implement this end to end on your own", "no need to confirm the plan".'
---

# /implement-auto-improve

Run `/implement` exactly as written, with one change: **you approve the plan yourself instead of waiting for the user.**

Everything else — branch checks, Notion context, checkpoints, specification, implementation, screenshots, manual-steps handoff, validation, cleanup — happens unchanged.

## Step 2 becomes self-approval

In `/implement` step 2 you write the plan and wait. Here you don't wait. Instead:

1. Write `plan-to-implement.md` as `/implement` specifies.
2. Review your own plan against this checklist:
   - Does it cover every explicit requirement from the user's message?
   - Are the inferred requirements actually implied, not invented? Cut anything speculative.
   - Is it the simplest plan that works? If 5 phases could be 2, rewrite it.
   - Does each phase have a concrete acceptance criterion you can verify by running something?
   - Are independent phases marked as such, so they can run in parallel?
3. Rewrite the plan to fix what the review found.
4. Post the final plan so the user can see what you decided, state that you're proceeding without approval, and continue to step 3.

Post the plan — don't skip showing it. Unattended means the user isn't blocking you, not that they're in the dark.

## What still stops you

Auto-approval covers the plan, nothing else. Stop and ask when:

- Requirements are contradictory or missing something no reasonable assumption resolves.
- There are multiple materially different readings of the ask and picking wrong wastes the whole run.
- A simplification or refactor would drop or change existing behavior.
- The work needs a destructive or irreversible action (dropping data, force-pushing someone else's branch, deleting files you didn't create).
- External docs (Notion pages) need editing — `/implement` step 7 still requires confirmation.

When you stop, say exactly what you need and what you've already finished.
