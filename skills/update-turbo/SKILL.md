---
name: update-turbo
description: Update installed Turbo skills to the latest version with conflict detection and exclusion support. Use when the user asks to "update turbo", "update turbo skills", "reinstall turbo", or "upgrade turbo".
---

# Update Turbo

Update installed Turbo skills to the latest version, respecting excluded skills and detecting conflicts.

## Step 1: Read Config

Read `~/.turbo/config.json` if it exists. Check for an `excludeSkills` array:

```json
{
  "excludeSkills": ["codex", "oracle"]
}
```

## Step 2: Detect Conflicts

Fetch the list of Turbo skills and check for non-symlinked skills that would be overwritten. Skip skills already in `excludeSkills`:

```bash
for skill in $(gh api repos/tobihagemann/turbo/contents/skills --jq '.[].name'); do
  # skip if already excluded
  target="$HOME/.claude/skills/$skill"
  if [ -e "$target" ] && [ ! -L "$target" ]; then
    echo "CONFLICT: $target exists and is not a symlink"
  fi
done
```

If conflicts are found, use `AskUserQuestion` to alert the user. For each conflicting skill, offer two options:
1. **Overwrite** — proceed with installation
2. **Exclude** — add the skill to `excludeSkills` in `~/.turbo/config.json` and skip it

Create `~/.turbo/config.json` if it doesn't exist. Merge the `excludeSkills` key into existing config.

## Step 3: Install

If there are excluded skills (from config or user choice), build a specific skill list:

```bash
npx skills add tobihagemann/turbo --skill 'skill1' --skill 'skill2' ... --agent claude-code -y -g
```

If no exclusions, install all:

```bash
npx skills add tobihagemann/turbo --skill '*' --agent claude-code -y -g
```

## Step 4: Clean Up Removed Skills

Compare installed skills against the current repo skill list. A skill is a Turbo skill if `~/.claude/skills/<name>` is a symlink (non-symlinked directories are user-owned and should be skipped). Check without trailing slash so `-L` works:

```bash
remote_skills=$(gh api repos/tobihagemann/turbo/contents/skills --jq '.[].name')
for skill in ~/.claude/skills/*; do
  [ -L "$skill" ] || continue
  name=$(basename "$skill")
  echo "$remote_skills" | grep -qx "$name" || echo "REMOVED: $name"
done
```

If any removed skills are found, inform the user and remove each one:

```bash
npx skills remove <skill-name> -g -y
```
