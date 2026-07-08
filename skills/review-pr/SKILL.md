---
name: review-pr
description: Review a git pull request or local diff for correctness bugs, security issues, and code quality before merge. Use when asked to review a PR, review a diff, review code changes, do a code review, or check changes before committing/merging.
license: MIT
compatibility: opencode
metadata:
  category: code-review
---

## What I do

Review the code changes in a PR or local diff and report concrete, actionable findings — not a style lecture. Priority order: correctness bugs > security issues > missing test coverage > readability/simplification.

## When to use me

- User asks to "review this PR", "review my diff", "check this branch before merge"
- User pastes a PR URL/number or says "review PR #123"
- User wants feedback before opening or merging a PR

Not for: auditing an entire codebase from scratch — that's a much bigger task. Stick to the diff plus enough surrounding context to judge it correctly.

## Step 1 — Determine scope

Figure out exactly what to review:

- PR by number/URL → `gh pr view <n>` and `gh pr diff <n>`
- Local branch vs base → `git diff <base>...HEAD`
- Uncommitted changes → `git diff` / `git diff --staged`

If it's ambiguous (no PR number given, multiple candidate branches, unclear base), ask the user instead of guessing.

## Step 2 — Gather full context

Never judge a diff hunk in isolation:

- Read each changed file in full, not just the `+`/`-` lines — bugs often hide just outside the visible hunk (e.g. a removed null check whose caller wasn't touched).
- Grep for other call sites of changed functions/classes to catch breaking changes to signatures or behavior.
- Read the PR description/commit messages for stated intent, then verify the diff actually delivers that — mismatches between stated intent and actual change are a common source of real bugs.

## Step 3 — Find issues

Work through the categories in [references/checklist.md](references/checklist.md). For every candidate finding, before reporting it:

- State the concrete failure scenario: specific input/state → wrong output, crash, or vulnerability. If you can't state one, it isn't a real finding — drop it or downgrade to a nit.
- Verify by re-reading the actual code and its callers, not by pattern-matching on how the code "looks."

Rank findings:
- **Blocking** — bug, security hole, data loss, breaks a caller
- **Should-fix** — real logic risk, or a missing test for a reachable edge case
- **Nit** — naming/style only worth a one-line mention, never a blocker

## Step 4 — Report

```
## Summary
<1-3 sentences: overall risk, is this safe to merge>

## Blocking
- file.ts:42 — <what's wrong> → <concrete failure scenario> → <suggested fix>

## Should-fix
- ...

## Nits
- ...

## Test coverage
<gaps in tests for the new/changed behavior, if any>
```

Omit a section entirely if it has zero findings — don't write "none found."

## Step 5 — Posting (only if asked)

If the user wants the review posted to the PR (`gh pr comment`, or inline comments via `gh api repos/<owner>/<repo>/pulls/<n>/comments`), confirm the target PR and the exact comment content with the user first. Posting is visible to others on the team and not easily undone.

## Guardrails

- Don't flag issues outside the diff unless the diff directly causes them (e.g. a caller broken by a signature change).
- Don't repeat what a linter/formatter/type-checker/CI already catches automatically.
- Don't invent hypothetical edge cases that aren't reachable through a real input or call path — ground every finding in an actual scenario.
- If the diff is large, say so upfront and propose reviewing it in logical chunks (by file or module) instead of skimming everything at once.
