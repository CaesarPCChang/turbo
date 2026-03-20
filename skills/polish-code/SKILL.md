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

Reviews and fixes code style and structure: reuse opportunities, quality patterns, efficiency, and clarity. Uses multiple Claude agents scanning the diff in parallel. Does not catch correctness or logic bugs.

Stage any changes made by the simplifier.

## Step 3: Review and Fix

Run the `/review-code` skill. The diff command is `git diff --cached`.

Catches correctness and logic bugs: missing guards, security issues, incorrect conditions, unhandled edge cases. Uses a different reviewer than Step 2 and catches different problems. Always run this step even if Step 2 found nothing.

Apply each actionable finding from the evaluated results directly, skipping false positives.

Stage any changes made by the fixes.

## Step 4: Test

Run the project's test suite to confirm nothing broke. If tests fail, run the `/investigate` skill to diagnose the root cause, apply the suggested fix, and re-run tests. If investigation cannot identify a root cause, stop and report with investigation findings.

Stage any changes made by test fixes.

## Step 5: Lint

Run the project's formatter first, then the linter. Fix any lint errors or warnings that the formatter did not resolve. If the project has a combined format+lint script, use that.

Stage any changes made by the formatter, linter, or manual fixes.

## Step 6: Re-run if Changed

If any prior step produced changes, run the `/polish-code` skill again, skipping Step 1.

## Rules

- Cap at 3 consecutive runs to prevent runaway loops.
