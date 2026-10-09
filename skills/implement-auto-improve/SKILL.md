---
name: implement-auto-improve
description: 'Run the full /implement workflow with no human in the loop: build the plan, self-review and improve it, approve it yourself, and carry the implementation through to final validation without ever pausing for input. Use when the user asks to implement something unattended — e.g. "/implement-auto-improve", "just implement it, don''t ask me anything", "implement this end to end on your own", "no need to confirm the plan".'
---

# /implement-auto-improve

Run `/implement` end to end with **no human in the loop**. Never pause for approval, confirmation, or a clarifying question. The user is not waiting on you — they read the result when you're done.

Everything in `/implement` still happens: branch checks, Notion context, the plan, checkpoints, specification, implementation, screenshots, manual-steps handoff, validation, cleanup. What changes is that every point where `/implement` would stop and ask, you decide instead.

## Decide, don't ask

| `/implement` says | Here you |
| --- | --- |
| Step 0 — suggest creating a branch | Create it and check it out. Name it from the requirements. |
| Step 1 — "ask clarifying questions only if required" | Never ask. Pick the reading a careful engineer would, and write the assumption into the plan. |
| Step 2 — show the plan and wait for approval | Self-review, improve, post the plan, proceed. |
| Step 6 — "stop and ask" before a simplification that changes behavior | Don't make that simplification. Keep the existing contract and note what you left alone. |
| Step 7 — update docs "after the user confirms" | Update in-repo docs directly. Leave external pages (Notion) alone and list the proposed update in the report. |

## Step 2 becomes self-approval

1. Write `plan-to-implement.md` as `/implement` specifies.
2. Review your own plan against this checklist:
   - Does it cover every explicit requirement from the user's message?
   - Are the inferred requirements actually implied, not invented? Cut anything speculative.
   - Is it the simplest plan that works? If 5 phases could be 2, rewrite it.
   - Does each phase have a concrete acceptance criterion you can verify by running something?
   - Are independent phases marked as such, so they can run in parallel?
   - Is every ambiguity in the ask resolved by a stated assumption rather than left open?
3. Rewrite the plan to fix what the review found.
4. Post the final plan so the user can see what you decided, say you're proceeding, and continue to step 3.

Post the plan — don't skip showing it. Unattended means the user isn't blocking you, not that they're in the dark.

## Nothing blocks; everything surfaces

You will hit things you can't resolve or shouldn't do alone: contradictory requirements, a decision that needs product input, a credential you don't have, an irreversible action outside the ask (dropping data, force-pushing someone else's branch, deleting files you didn't create).

None of these stop the run. For each one:

- Finish every part of the task that doesn't depend on it.
- Don't take the irreversible action and don't guess at a value you can't see.
- Add it to the step 10 manual-steps checklist — what's needed, why you couldn't do it, and what you did instead.

So the run always ends with working code plus one honest list of what a human still owns. It never ends with a question and nothing built.

## Report

Close with `/implement` step 8's summary plus:

- Assumptions you made, and what you'd have asked if you could.
- Anything in the plan you cut as speculative.
- The manual-steps checklist, even if it's "no manual steps required".
