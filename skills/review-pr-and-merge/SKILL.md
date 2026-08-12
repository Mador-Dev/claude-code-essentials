---
name: review-pr-and-merge
description: 'End-to-end PR review that runs the branch locally, drives the main user flows in a real browser with Playwright, and merges if nothing is broken. Use this whenever the user asks to review, check, verify, test, approve, or merge a pull request — given a PR number, PR link, branch name, or the issue the PR implements (e.g. "review PR 412", "can you check this PR and merge it", "does #88 break anything?", "look at github.com/org/repo/pull/57"). Also use it when someone asks whether a PR is safe to ship or whether it broke existing flows. The bias is toward merging: only production-breaking problems block a merge, and those get a PR comment plus an email to the author.'
---

# Review PR and merge

The job: prove the PR does what it was asked to do and doesn't break anything that already works — then get out of the way and merge it.

This skill exists because the default review instinct is too strict. Style nits, missing tests, and "I would have done it differently" are not reasons to hold up a merge. Only things that break production are.

## The bar

Merge when the PR broadly does what the issue asked and the main flows still work.

Block only for problems that would break production:

- A core user flow throws, hangs, or renders an error state that wasn't there before
- Build fails, app doesn't boot, or a route 500s / white-screens
- Data loss or corruption — wrong writes, deletes without guard, broken migration
- Secrets, API keys, or tokens committed, or auth/permission checks removed
- An API contract other code depends on changed without callers updated
- An unhandled promise rejection or uncaught exception on a happy path
- Something so far from the requirement that shipping it is worse than not shipping

Never block for: naming, formatting, file layout, missing tests, missing types, TODO comments, a slightly different approach than you'd take, minor a11y gaps, or console warnings that don't break a flow. If it's genuinely worth mentioning but not blocking, put it in the merge summary as a note — not as a PR comment.

Never merge blind. If the app can't be run or the flows can't be exercised, don't guess — report exactly what stopped you and let the user decide. A merge that wasn't verified is worse than no merge.

## Workflow

### 1. Resolve the PR

Accept a PR number, PR URL, branch name, or an issue reference — resolve all of them to one PR:

```bash
gh pr view <number-or-url> --json number,title,body,author,headRefName,baseRefName,url,files,additions,deletions,state,mergeable
```

```bash
gh pr list --search "<issue-number-or-text>" --json number,title,url
```

If `gh` isn't installed or authenticated, fall back to `git` plus the repo's remote; if neither works, say so and stop. If more than one PR matches an issue, list them and ask which.

### 2. Understand what was asked

Read, in this order, and stop when the requirement is clear:

1. The PR description and its linked issue (`gh issue view <n>`)
2. Notion — search for the ticket, spec, or feature page (see "Notion as second brain")
3. The diff itself (`gh pr diff <n>`)

Write down the requirement as 1–3 concrete, checkable statements. These are what you verify in step 5. If the requirement can't be pinned down at all, ask the user rather than inventing acceptance criteria.

### 3. Run it locally

```bash
gh pr checkout <n>
```

Or, without `gh`:

```bash
git fetch origin pull/<n>/head:pr-<n> && git checkout pr-<n>
```

Install and start using whatever the repo actually uses — check `package.json` scripts, lockfile, and any README/Makefile before assuming. Typical:

```bash
npm ci
```

```bash
npm run build
```

```bash
npm run dev
```

A failing build is already a block. Background the dev server and capture the port from its output.

Notes that save time: private registries and base images may be configured in `.npmrc` / `Dockerfile` — respect them. If the app needs env vars, look for `.env.example` and ask for anything secret rather than stubbing auth. If it needs a backend or DB that isn't available, say so instead of mocking your way to a green result — a fake pass is the one outcome that makes this skill useless.

### 4. Drive the main flows with Playwright

Pick the flows to test from, in order: existing e2e specs in the repo, the app's primary routes and nav, prior review notes in Notion, and whatever the diff touches. Aim for 3–6 flows — the ones that would cause a real incident if they broke, plus every flow the PR touches directly.

```bash
npx playwright install chromium
```

Write a throwaway script in a temp dir (not in the repo — don't add files to someone's PR branch):

```js
// /tmp/pr-<n>-check.mjs
import { chromium } from 'playwright';

const errors = [];
const browser = await chromium.launch();
const page = await browser.newPage();
page.on('console', m => m.type() === 'error' && errors.push(`console: ${m.text()}`));
page.on('pageerror', e => errors.push(`pageerror: ${e.message}`));
page.on('response', r => r.status() >= 400 && errors.push(`http ${r.status()}: ${r.url()}`));

// one block per flow: navigate, interact, assert something real is on screen
await page.goto('http://localhost:5173/');
await page.getByRole('button', { name: /add position/i }).click();
await page.waitForSelector('[data-testid="position-form"]');

await browser.close();
console.log(errors.length ? errors.join('\n') : 'clean');
```

What matters when judging the run:

- Assert on visible outcomes, not just that the page loaded — a blank div with no errors is still a broken flow. Screenshot each flow (`page.screenshot`) and actually look at it.
- Console errors and 4xx/5xx are signal, not proof. Compare against `main`: if the same error already happens on the base branch, the PR didn't cause it and it isn't a block.
- Flaky selectors and timeouts are your bug, not the PR's. Retry once with a better wait before concluding anything is broken.
- If a flow fails, isolate it — re-run just that flow on `main` to confirm the PR is responsible.

### 5. Decide

Check each requirement statement from step 2 against what you saw, then apply "The bar" above. Two outcomes only.

### 6a. Clean → merge

```bash
gh pr merge <n> --squash --delete-branch
```

Match the repo's convention if it differs. If merge is blocked by branch protection or a required check, don't fight it — report what's blocking and stop.

Then log the review to Notion (see "Notion as second brain") and give the user a short summary: what the PR does, what you exercised, what you found, that it's merged. Include non-blocking observations here as notes.

### 6b. Production-breaking problem → comment, email, don't merge

PR comments — only where crucially needed, one per distinct fundamental problem. Don't leave a comment for anything you wouldn't block on.

```bash
gh pr comment <n> --body "..."
```

```bash
gh pr review <n> --comment --body "..."
```

Each comment says: what breaks, the exact reproduction (flow, step, observed result), and why it's production-breaking. Be specific and factual — no hedging, no lecture, no list of unrelated nits.

Email the author — find their address in Notion (see below), match on GitHub handle or name. Keep it to a few sentences: what's wrong, what it breaks, the PR link.

Send it if a send-capable mail tool is available. The Gmail connector can only create drafts — in that case create the draft with `create_draft` and tell the user plainly that it's sitting in Drafts ready to send, since a draft they don't know about is the same as no email.

```
Subject: PR #<n> — blocking issue in <short description>

Hi <name>,

I reviewed PR #<n> (<title>) and hit a problem that would break production:

<1–3 sentences: what breaks and how to reproduce>

Holding the merge until that's resolved. PR: <url>

Thanks,
<sender>
```

If no email address can be found for the author, skip the email, say so, and rely on the PR comment.

## Notion as second brain

Search Notion before reviewing and write to it after. Two uses:

**Read** — `notion-search` for the issue key, feature name, or PR title to find specs, acceptance criteria, and prior review notes on the same area (past reviews often record which flows are fragile and which console errors are pre-existing — that context prevents false blocks). Team member emails live in Notion too: search for the team/contacts page or directory database and pull the author's address from there.

**Write** — after every review, append an entry to the PR review log. Look for an existing "PR Reviews" page or database first (`notion-search`); create one only if it genuinely doesn't exist. Keep entries short and uniform so they're useful to skim later:

```
PR #<n> — <title>  [<date>]
Author: <name> · Status: merged / blocked
Requirement: <one line>
Flows tested: <list>
Result: <one line — clean, or what broke>
Notes: <pre-existing errors, fragile flows, anything future reviews should know>
```

## Environment fallbacks

This skill should degrade gracefully rather than fail:

- No `gh` → use `git` and the GitHub API via `curl` with a token if available; otherwise report and stop
- No Playwright / no browser → say the flows couldn't be exercised and don't merge on assumption. Running the repo's own e2e or unit tests is a partial substitute worth reporting, but state clearly that browser verification didn't happen
- No Notion connector → do the review anyway, note that context and the review log were skipped
- No mail tool → post the PR comment and tell the user to notify the author

## Style

Be concise. The user wants an outcome, not a report: what you ran, what you found, what you did. A clean merge should take a short paragraph. Don't pad a passing review with speculative concerns — if it merged, say it merged.
