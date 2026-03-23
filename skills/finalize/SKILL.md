---
name: finalize
description: "Run the post-implementation quality assurance workflow including tests, code polishing, review, and commit. Use when the user asks to \"finalize implementation\", \"finalize changes\", \"wrap up implementation\", \"finish up\", \"ready to commit\", \"run QA workflow\", or \"ship it\"."
---

# Finalize Implementation

Post-implementation QA workflow: tests, code polishing, commit, and self-improvement.

## Task Tracking

At the start, use `TaskCreate` to create a task for each phase:

1. Polish code
2. Self-improve
3. Commit and PR

## Phase 1: Polish Code

Run the `/polish-code` skill for the current changes.

## Phase 2: Self-Improve

Run the `/self-improve` skill for the current session.

## Phase 3: Commit and PR

### Step 1: Determine Intent

Detect the repository's default branch via `gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'`. Check the current branch name and whether a PR already exists for it using `gh pr view`.

Use `AskUserQuestion` to ask the user how to proceed. Present the options based on the current state:

- **On a feature branch with an existing PR** — commit and push; or commit, push, and update the PR; or commit only
- **On a feature branch without a PR** — commit and push; or commit, push, and create a PR; or commit only
- **On the default branch** — commit and push; or create a feature branch, commit, push, and create a PR; or commit only
- **Abort** — leave changes staged, do not commit

### Step 2: Branch (if Needed)

If the user wants a PR and the current branch is the default branch:

1. Suggest a branch name based on the changes and use `AskUserQuestion` to confirm or adjust
2. Create and switch to the new branch: `git checkout -b <branch-name>`

### Step 3: Check for Unstaged Changes

Run `git status` to check for unstaged changes. If any exist, stage them. This catches files modified by auto-formatters that were not re-staged.

### Step 4: Commit

Run the `/commit-staged` skill.

If the commit fails due to a pre-commit hook (formatter, linter), fix the issues — or run the project's format/lint script to auto-fix — then **re-stage all modified files** before retrying. Pre-commit hooks may modify files in the working tree without updating the staging area.

### Step 5: Push and Create or Update PR (if Requested)

- **Push only** — push (do not create or update a PR)
- **Create PR** — push with `-u` and run the `/create-pr` skill
- **Update PR** — push and run the `/update-pr` skill
- **Skip** — end the workflow (do not push)

### Step 6: Resolve PR Comments

Use `AskUserQuestion` to ask if the user wants to wait for automated reviewers to finish and resolve comments.

- **Skip** — end the workflow
- **Resolve comments** — run the `/resolve-pr-comments` skill

## Rules

- Diff size, number of files changed, passing tests, or perceived user urgency are not reasons to skip a phase. Each phase does work beyond what those signals cover.
- Never stage or commit files containing secrets (`.env`, credentials, API keys). Warn if detected.
- Do not present diffs to the user — the user reviews diffs in an external git client. Use `git diff` internally as needed.
- If a non-test step fails (polish, review), stop and report the failure. Do not skip ahead.
