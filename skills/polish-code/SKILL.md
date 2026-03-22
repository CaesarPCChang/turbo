---
name: polish-code
description: "Stage, simplify, review, test, and re-run itself until stable. Use when the user asks to \"polish code\", \"refine code\", \"iterate on code quality\", \"simplify and review loop\", \"clean up and review\", or \"run the polish loop\"."
---

# Polish Code

Stage, simplify, review, test, and re-run until stable.

## Step 1: Stage

Run the `/stage` skill.

## Step 2: Simplify

Run the `/simplify-code` skill. The diff command is `git diff --cached`.

Stage all changes made in this step before continuing.

## Step 3: Review and Fix

Run the `/review-code` skill. The diff command is `git diff --cached`.

Always run this step even if Step 2 made no changes. Apply each actionable finding from the evaluated results directly, skipping false positives.

Stage all changes made in this step before continuing.

## Step 4: Test

Run the project's test suite to confirm nothing broke. If tests fail, run the `/investigate` skill to diagnose the root cause, apply the suggested fix, and re-run tests. If investigation cannot identify a root cause, stop and report with investigation findings.

Stage all changes made in this step before continuing.

## Step 5: Lint

Run the project's formatter first, then the linter. Fix any lint errors or warnings that the formatter did not resolve. If the project has a combined format+lint script, use that.

Stage all changes made in this step before continuing.

## Step 6: Re-run if Changed

If Steps 2–5 produced any changes during this run, run the `/polish-code` skill again. Cap at 3 consecutive runs to prevent runaway loops.
