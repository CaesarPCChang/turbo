---
name: polish-code
description: "Stage, format, lint, test, simplify, review, smoke test, and re-run itself until stable. Use when the user asks to \"polish code\", \"refine code\", \"iterate on code quality\", \"simplify and review loop\", \"clean up, test, and review loop\", or \"run the polish loop\"."
---

# Polish Code

Stage, clean up, simplify, review, smoke test, and re-run until stable.

## Task Tracking

At the start, use `TaskCreate` to create a task for each step:

1. Stage
2. Deterministic cleanup
3. Simplify code
4. Review code
5. Evaluate findings
6. Apply findings
7. Smoke test

## Step 1: Stage

Run the `/stage` skill.

## Step 2: Deterministic Cleanup

Run the project's formatter first, then the linter. Fix any lint errors or warnings that the formatter did not resolve. If the project has a combined format+lint script, use that.

Run the project's test suite to confirm nothing is broken. If tests fail, run the `/investigate` skill to diagnose the root cause, apply the suggested fix, and re-run tests. If investigation cannot identify a root cause, stop and report with investigation findings.

Stage all changes made in this step before continuing.

## Step 3: Simplify Code

Run the `/simplify-code` skill. The diff command is `git diff --cached`.

Stage all changes made in this step before continuing.

## Step 4: Review Code

Run the `/review-code` skill. The diff command is `git diff --cached`.

Always run this step even if Step 3 made no changes.

## Step 5: Evaluate Findings

Run the `/evaluate-findings` skill on the results from Steps 3 and 4.

If zero actionable findings survive evaluation, skip to Step 7.

## Step 6: Apply Findings

Run the `/apply-findings` skill on the evaluated results.

Stage all changes made in this step before continuing.

## Step 7: Smoke Test

Use the Agent tool to run the `/smoke-test` skill in a subagent (`model: "opus"`, do not set `run_in_background`). Pass the diff command (`git diff --cached`) to the subagent.

If any test fails, fix the issues and stage the fixes.

## Step 8: Re-run if Changed

Check whether any file was edited during Steps 3-7. Any edit counts, regardless of how small or mechanical it seems. A one-line doc comment fix, a renamed variable, a reformatted import, a smoke test fix — all trigger a re-run.

If any file was edited, re-run `/polish-code` via skill invocation (use the Skill tool) scoped to only the files modified in the previous iteration. Use `git diff --cached -- <file1> <file2> ...` as the diff command for `/simplify-code` and `/review-code`. Smoke test scope remains unchanged (full feature scope, not file-narrowed). Cap at 3 total iterations (the initial run plus up to 2 re-runs) to prevent runaway loops.

## Rules

- Every step must run. No change is "self-evidently correct" or "mechanical" enough to skip review. Simplify findings do not substitute for review. Passing tests do not substitute for lint. Each step catches different issues.
