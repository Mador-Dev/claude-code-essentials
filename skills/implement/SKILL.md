---
name: implement
description: End-to-end implementation workflow for features and refactors from free-text requirements.
---

# /implement

Execute the following workflow exactly.

## 1. Understand

- Read the user's requirements.
- Inspect the codebase if needed.
- Infer obvious requirements.
- Ask clarifying questions only if required.

## 2. Create Plan

Create or replace `plan-to-implement.md` in the repository root.

Include:

- Goals
- Explicit requirements
- Inferred requirements
- Implementation phases
- Acceptance criteria
- Testing plan
- Risks / assumptions

Show the plan and wait for user approval.

## 3. Checkpoint

Run:

```
/checkpoint
```

## 4. Create Specification

Run:

```
/create-specification plan-to-implement.md
```

## 5. Validate Specification

Ensure the specification fully satisfies every explicit and inferred requirement from `plan-to-implement.md`.

Fix any gaps before continuing.

## 6. Implement

Run:

```
/goal {specs-folder} Implement this specification in its entirety.

- Do not skip anything.
- Use subagents whenever beneficial.
- Create a checkpoint after each completed phase.
- Use the Playwright skill to test every user-facing feature end-to-end.
- Fix all failing tests before continuing.
```

## 7. Final Validation

Verify:

- Every specification item is implemented.
- All tests pass.
- No planned requirement was skipped.

Summarize what was implemented, tested, and any remaining limitations.