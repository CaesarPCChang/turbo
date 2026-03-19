---
name: tidy-memory
description: "Audit and clean up a project's auto-memory when it grows too large. Extracts inline content into separate files, removes stale or code-derivable entries, compresses verbose entries, and rebuilds the MEMORY.md index. Use when the user asks to \"tidy memory\", \"clean up memory\", \"memory is too large\", \"memory cleanup\", \"trim memory\", \"shrink memory\", \"memory too big\", \"reduce memory size\", \"organize memory\", \"memory audit\", or \"compact memory\"."
---

# Tidy Memory

Restructure an overgrown memory directory so MEMORY.md returns to a pure index under 200 lines.

## Step 1: Locate and Measure

Find the memory directory. If the user specifies a project, list `~/.claude/projects/` and match the directory whose name encodes the project path (e.g., `-Users-<username>-Developer-project-name` for `/Users/<username>/Developer/project-name`). Otherwise use the current project's memory directory.

Measure:
- Read `MEMORY.md` and count lines
- List all `.md` files in the directory (excluding `MEMORY.md`)
- Read each memory file

Report the current state: total MEMORY.md lines, number of separate memory files, and whether MEMORY.md contains inline content (content beyond short pointer entries).

## Step 2: Classify Every Entry

Walk through MEMORY.md and each memory file. For every entry (a bullet point, a section, or a file), assign one of these classifications:

| Classification | Criteria | Action |
|---|---|---|
| **Extract** | Inline content in MEMORY.md that should be a separate memory file | Move to its own file, leave a pointer |
| **Stale** | Information now derivable from the current codebase — file paths, API signatures, project structure, architecture that can be read from code | Remove |
| **Verbose** | Valid information but uses more tokens than necessary — explanations Claude already knows, redundant examples, excessive detail | Compress |
| **Duplicate** | Same information appears in multiple places — within MEMORY.md, across memory files, or already in CLAUDE.md | Deduplicate |
| **Merge** | Multiple small entries on the same topic that would be better as one consolidated memory file | Combine into a single file |
| **Keep** | Valid, concise, non-derivable content already in a separate file | No change |
| **Orphaned** | Memory file not referenced from MEMORY.md | Add pointer to index or remove if stale |

### Staleness Heuristics

An entry is likely stale when it:
- Documents which files or functions exist (readable from code)
- Lists switch sites or exhaustive update locations (findable via grep)
- Describes architecture that's visible from the module/directory structure
- Records a bug fix that's already in the codebase
- Documents API signatures or method names (readable from source)

An entry is NOT stale when it:
- Records a non-obvious gotcha or workaround (e.g., compiler bug, surprising framework behavior)
- Captures a decision rationale not visible in code
- Stores credentials or external references
- Documents cross-cutting conventions that aren't enforced by tooling
- Records user preferences or feedback

### Compression Heuristics

An entry can be compressed when it:
- Explains what a concept is (Claude already knows what PDFs, async streams, etc. are)
- Includes lengthy workaround steps when a one-liner would suffice
- Repeats the same pattern across multiple bullets (consolidate into a single rule)
- Contains code examples that restate what the text already says

## Step 3: Present the Cleanup Plan

Show the user a table before making any changes:

```
| # | Entry/Section | Classification | Action | Est. Lines Saved |
|---|---------------|----------------|--------|------------------|
| 1 | <Section A> (8 lines) | Extract | Move to section_a.md | 7 (from index) |
| 2 | <Section B> (3 lines) | Stale | Remove — derivable from source code | 3 |
| 3 | <Section C> (22 lines) | Extract + Compress | Move to section_c.md, compress verbose entries | 20 (from index) |
| ...| | | | |
```

Include totals:
- Current MEMORY.md lines
- Projected MEMORY.md lines after cleanup
- Number of new memory files to create
- Number of entries to remove

Use `AskUserQuestion` to let the user approve, reject, or modify individual items. Repeat until the user is satisfied.

## Step 4: Execute the Cleanup

Apply approved changes in this order:

1. **Create new memory files** for extracted content. Each file must have proper frontmatter:
   ```markdown
   ---
   name: {{descriptive name}}
   description: {{one-line description for relevance matching}}
   type: {{user|feedback|project|reference}}
   ---

   {{content}}
   ```

   Type guide: **user** = role, preferences, knowledge. **feedback** = corrections and confirmed approaches. **project** = ongoing work, goals, decisions. **reference** = pointers to external systems.

2. **Compress** verbose entries in both new and existing files. Write the compressed version, not a summary. Every fact must survive compression.

3. **Remove** stale and duplicate entries.

4. **Fix orphaned files** — add missing pointers to MEMORY.md or remove stale orphans.

5. **Rebuild MEMORY.md** as a pure index: one line per memory file with a brief description, organized by topic. No inline content.

## Rules

- Never discard information the user hasn't approved removing. When in doubt, classify as **Keep**.
- Credentials, external references, and user preferences are always **Keep** regardless of age.
- Compression must preserve every fact. Reword for brevity, don't summarize or generalize.
- The rebuilt MEMORY.md must stay under 200 lines (the truncation threshold).
- Memory file names should be descriptive and use snake_case (e.g., `swift_compiler_bugs.md`, `test_patterns.md`).
