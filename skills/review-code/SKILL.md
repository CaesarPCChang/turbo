---
name: review-code
description: Run AI code review, evaluate findings, apply fixes, and verify. Launches `/code-review` and `/peer-review` in parallel, then evaluates and applies actionable findings. Use when the user asks to "review code", "code review", "review my changes", "find bugs in my changes", or wants a reviewed and evaluated set of findings.
---

# Review Code

Run AI code review, evaluate findings, apply fixes, and verify.

## Step 1: Determine the Diff Target

Determine what to review based on context:

- **Uncommitted changes**: `--uncommitted`
- **Against a base branch**: `--base <branch>`
- **Specific commit**: `--commit <sha>`

Default to reviewing against the repository's default branch (detect via `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`). If the caller specifies a different target, use that.

## Step 2: Run Two Reviews in Parallel

The diff target from Step 1 determines what each reviewer analyzes.

### Review A: Code Review

Launch a background agent (`model: "opus"`, `run_in_background: true`) that runs the `/code-review` skill with the diff target from Step 1.

### Review B: Peer Review

Run the `/peer-review` skill directly (via the Skill tool) with the same diff target. This runs in the main thread while Review A works in the background.

## Step 3: Evaluate Findings

Combine findings from both reviewers and run the `/evaluate-findings` skill on the merged set.

If there are additional findings to include (e.g., PR comments passed in by the caller), combine them before evaluation.

If zero actionable findings survive evaluation, report that the code looks clean and stop.

## Step 4: Apply Fixes

Run the diff command to get the current changes, then apply each actionable finding directly, skipping false positives. Only edit files — do not stage, build, or test.

## Step 5: Simplify Fixes

Run the `/simplify-code` skill. The diff command is `git diff` (the fixes are unstaged).

## Step 6: Test and Lint

1. Run the test suite to confirm nothing broke
2. If tests fail, run the `/investigate` skill to diagnose the root cause, apply the suggested fix, and re-run tests. If investigation cannot identify a root cause, stop and report with investigation findings.
3. Run the project's linter/formatter to ensure clean output

## Rules

- If either reviewer is unavailable or returns malformed output, proceed with findings from the other reviewer alone.
- Process findings in file order to minimize context switching.
