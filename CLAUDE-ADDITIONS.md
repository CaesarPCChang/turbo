# CLAUDE.md Additions

Turbo-recommended additions for `~/.claude/CLAUDE.md`. Managed during [setup](SETUP.md) and [`/update-turbo`](UPDATE.md).

Each `##` section below maps to a `#` section in `~/.claude/CLAUDE.md`.

## Skill Loading

- Always use the Skill tool to invoke skills — never substitute by executing steps from memory, even if the skill was loaded earlier in this conversation, including skills invoked by other skills or by themselves
- "Already running" only means don't call a skill *in the same turn* where its `<command-name>` tag already appeared

## Pre-Planning Prep

Before writing a plan, use `AskUserQuestion` to ask whether to run `/plan-style`.

## Pre-Implementation Prep

After plan approval (ExitPlanMode) and before making edits:
1. Run `/code-style` to load code style principles
2. Read all files referenced by the user in their request
3. Read all files mentioned in the plan
4. Read similar files in the project to mirror their style
