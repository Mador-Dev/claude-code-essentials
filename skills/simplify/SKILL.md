---
name: simplify
description: 'Simplify code and repo hygiene without changing behavior: cut unnecessary lines, remove unused files, strip non-essential comments, and prune non-essential docs/.md files. Default scope is committed changes not yet on the base branch (a PR-shaped diff) — pass "all" / "entire project" / "whole repo" to widen scope to the full repository. Always proposes a plan and waits for explicit approval before touching anything, then verifies nothing broke. Use when the user asks to simplify, clean up, or trim down code, or invokes /simplify directly.'
---

# /simplify

The job: make the codebase smaller and easier to read without changing what it does. This is a quality pass, not a bug hunt (use `/code-review` for correctness) and not a feature change (never alter behavior, inputs, outputs, or error handling to "simplify").

Every change must pass one test: **would a teammate approve this as a pure readability/size win, with zero functional risk?** If a candidate change is even slightly ambiguous about behavior, drop it from the plan instead of guessing.

## 0. Determine scope

- **Default (no args, or args don't say otherwise):** only *committed* changes not yet on the base branch — i.e. the same diff a PR review would see.
  ```bash
  git fetch origin
  BASE=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
  BASE=${BASE:-main}
  git diff --name-only "$(git merge-base HEAD "origin/$BASE")"...HEAD
  ```
  If HEAD has no divergence from the base (you're on the base branch itself, or everything's already merged), fall back to just the last commit: `git diff --name-only HEAD~1..HEAD`.
  Uncommitted working-tree changes are out of scope — mention them to the user but don't touch them.

- **Explicit full-project mode:** only when the user's request clearly says so (e.g. "simplify the entire project", "whole repo", "everything", or `/simplify all`). Then scope is every source file in the repo, not just the diff.

State which scope you're using before doing anything else.

## 1. Inventory candidates

Within scope, look for four categories. Don't apply anything yet — just build a list.

### a. Code simplification (the main job)

Apply [Chesterton's Fence](https://en.wikipedia.org/wiki/Wikipedia:Chesterton%27s_fence) first: for anything you're about to touch, understand why it's there (check `git blame`, callers, tests) before deciding the reason no longer applies.

Scan for concrete, not vague, signals:

| Pattern | Simplify to |
|---|---|
| Deep nesting (3+ levels) | Guard clauses / extracted helper |
| Long functions doing several things | Split into focused, named functions |
| Nested ternaries | if/else chain or lookup table |
| Boolean flag params (`doThing(true, false)`) | Options object or separate functions |
| Repeated inline conditionals | One named predicate function |
| Manual loops building an array/map | `filter`/`map`/comprehension equivalent |
| Duplicated 5+ line blocks | Extracted shared function |
| Dead code (unreachable branches, unused vars, commented-out blocks) | Removed, after confirming it's truly unused |
| Wrapper/abstraction that adds no value (factory-for-a-factory, single-strategy strategy pattern) | Inlined |
| Redundant type assertions/casts | Removed |

Rules that keep this safe:
- Preserve behavior exactly — same inputs/outputs, same errors, same side effects and ordering. If unsure, don't make the change.
- Match existing project conventions (see [CLAUDE.md](../../CLAUDE.md)); simplification that fights the codebase's own style is churn, not simplification.
- Fewer lines is not the goal — comprehension speed is. Don't collapse two clear functions into one dense one just to cut lines.
- Never remove or weaken error handling to make code "cleaner."
- Don't touch code outside the declared scope, even if you notice something nearby.

### b. Unused files

Flag a file as removable only if, within scope:
- Nothing imports/requires/references it anywhere in the repo (check by filename and by module path, including dynamic imports and config-driven references — build configs, route tables, CI YAML, `package.json` scripts).
- It isn't a well-known convention file tooling expects even without a code reference (e.g. `.gitignore`, `.env.example`, license/config files).

If a file only *might* be unused, list it as a flagged-but-uncertain item in the plan rather than assuming.

### c. Non-essential comments

Remove comments that restate what the code already says (`// increment counter` above `count++`). Keep comments that carry a "why" the code can't express: a workaround for a specific bug, a non-obvious constraint, a reason a simpler-looking approach was rejected. When in doubt, keep it.

### d. Non-essential docs / `.md` files

Protected by default — never propose removing these unless the user names them explicitly: `README.md`, `CLAUDE.md`, `LICENSE`, `CONTRIBUTING.md`, and any doc referenced by tooling (CI config, `package.json`, build scripts).

Candidates for removal: stale planning notes, superseded design docs, duplicate/outdated docs nothing links to, scratch `.md` files left over from prior work — anything not required for the system to build, run, or be understood by a newcomer. Confirm nothing (code, CI, other docs) links to a file before flagging it.

## 2. Build the simplification plan

Produce a plain-text plan (in chat, not a file) listing every candidate change as a table or list:

```
File                          Action                              Reason                    Est. impact
src/foo/bar.ts                Extract guard clauses (L40-78)       3-level nesting           -12 lines
src/utils/oldHelper.ts        Delete (unused)                      No references found       -34 lines
notes/2024-planning.md        Delete                               Stale, unlinked           -1 file
src/api/client.ts             Remove restating comment (L12)       Adds no info              -1 line
```

Include a short summary line: files touched, files deleted, estimated total line reduction.

## 3. Get approval

Show the plan and stop. Do not modify anything until the user explicitly approves. If they want changes to the plan (exclude an item, widen/narrow scope), revise and re-show it before proceeding.

## 4. Apply, incrementally

For each approved item, one at a time:
1. Make the change.
2. Run the project's existing test suite / build / typecheck / lint (whatever the repo actually has — check `package.json` scripts or equivalent before assuming).
3. If it passes, move to the next item. If it fails, revert that one change and report it instead of guessing at a fix that changes behavior.

Never batch multiple unrelated simplifications into one untested step — if something breaks, you need to know which change caused it.

## 5. Verify nothing broke

Before reporting done, confirm:

- [ ] Existing tests pass without modification
- [ ] Build/typecheck succeeds with no new warnings
- [ ] Linter/formatter passes
- [ ] No error handling was removed or weakened
- [ ] No behavior changed — only expression of the same behavior
- [ ] The diff is clean: only approved items, nothing unrelated snuck in

If any deleted file or doc turns out to be referenced somewhere you missed, restore it and drop it from the summary instead of leaving the repo broken.

## 6. Summarize

Report concisely: what was removed/simplified, actual line/file delta, anything flagged in step 1 but deliberately left out of the plan (and why), and confirmation that verification passed.
