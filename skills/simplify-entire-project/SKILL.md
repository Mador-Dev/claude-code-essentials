---
name: simplify-entire-project
description: 'Same as /simplify, but always scoped to the entire repository instead of just new committed changes. Use when the user asks to simplify, clean up, or trim down the whole project/repo, or invokes /simplify-entire-project directly.'
---

# /simplify-entire-project

Run the [`/simplify`](../simplify/SKILL.md) workflow exactly, with one change: skip step 0's scope detection and go straight to **full-project mode** — every source file in the repo is in scope, not just the diff against the base branch.

Everything else is unchanged: build the candidate inventory (unused files, non-essential comments, non-essential docs, code simplification), present the plan, wait for explicit approval, apply approved changes incrementally with tests/build/lint after each, then verify and summarize.
