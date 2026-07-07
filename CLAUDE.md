# CLAUDE.md

Rules for working in this repo. These are MUST-follow, not suggestions.

## 1. Plan Before Implementing

- Always plan the approach before writing code.
- For large or ambiguous features, stop and ask clarifying questions first — never guess on scope.

## 2. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick one silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop, name what's confusing, and ask.

## 3. Simplicity First

Write the minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No unrequested "flexibility" or "configurability".
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

Test: "Would a senior engineer call this overcomplicated?" If yes, simplify.

## 4. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

Test: every changed line traces directly to the user's request.

## 5. Subagents & Parallelism

- For big features, identify plan steps that are independent and implement them in parallel with subagents.
- Keep sequential work inline; only parallelize when tasks don't share files or depend on each other.

## 6. Model Selection

- Use the most capable model for complex or large tasks.
- Lighter/faster models are fine for very simple, mechanical tasks (renames, formatting, boilerplate).

## 7. UI Design

- Always follow the current design system when creating or reviewing pages: @DESIGN.md
