---
name: review-pr
description: "Review a pull request by fetching PR comments and running a comprehensive code review. Use when the user asks to \"review PR\", \"review pull request\", \"review this PR\", \"check PR before merging\", or \"full PR review\"."
---

# Review PR

Fetch PR context and run a comprehensive code review.

## Step 1: Fetch PR Comments

Run the `/fetch-pr-comments` skill to get unresolved review comments.

## Step 2: Detect Base Branch

Detect the PR's base branch via `gh pr view --json baseRefName --jq '.baseRefName'`.

## Step 3: Code Review

Run the `/review-code` skill. The diff command is `git diff <base-branch>...HEAD`. Pass any unresolved PR comments as additional findings for the evaluation step.

## Rules

- If fetching PR comments fails, proceed with code review only.
