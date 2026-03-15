# Prompt Plan Reviewer Guidelines

Review a prompt plan against its source spec to verify that the plan is a complete, gap-free decomposition.

The core principle: a prompt plan is a "broken down spec." Following the full prompt chain should implement the entire spec with nothing missing and nothing disconnected.

## Inputs

- The prompt plan file
- The source spec (path listed in the prompt plan's `Source:` field)

Read both files in full before starting the review.

## Review Dimensions

### 1. Spec Coverage (No Gaps)

For every requirement, feature, acceptance criterion, and constraint in the spec, verify it appears in at least one prompt.

Process:
1. Extract a numbered list of discrete requirements from the spec
2. For each requirement, identify which prompt(s) cover it
3. Flag any requirement not assigned to a prompt

Output a coverage matrix:

```
| # | Spec Requirement (short summary) | Prompt(s) | Status    |
|---|----------------------------------|-----------|-----------|
| 1 | User authentication via OAuth     | Prompt 2  | Covered   |
| 2 | Rate limiting on API endpoints    | —         | GAP       |
```

### 2. Wiring (Outputs Feed Inputs)

Each prompt produces artifacts (files, modules, endpoints, types) that later prompts may depend on. Verify the chain is connected.

Check for:
- **Dead ends** — A prompt creates a module, API, or capability that no subsequent prompt consumes or integrates. This produces dead code.
- **Missing prerequisites** — A prompt assumes an artifact exists that no prior prompt creates.
- **Implicit dependencies** — A prompt's `Depends on` field omits a prompt it actually relies on.

Output a wiring report:

```
| Prompt | Produces              | Consumed by | Issue?              |
|--------|-----------------------|-------------|---------------------|
| 1      | DB schema, models     | 2, 3        | —                   |
| 2      | Auth middleware        | —           | Dead end: no consumer |
| 3      | API endpoints         | 4           | —                   |
```

### 3. Completeness (Chain Implements the Spec)

Walk through the prompts in order and simulate what exists after each one completes. After the final prompt, verify that:
- Every spec feature is reachable from the project's entry points
- No component is orphaned (created but never wired into the system)
- The system described in the spec would be functional, not just a collection of parts

### 4. No Duplication

Verify each spec requirement is assigned to exactly one prompt. Duplication causes conflicting implementations. If two prompts intentionally touch the same area (one creates, the other extends), confirm they have clear boundaries.

### 5. Cross-Reference Accuracy

If prompts reference external resources (other codebases, documentation, APIs, search terms), spot-check that:
- Referenced projects or docs actually exist and are relevant
- Suggested search terms would find useful code
- The most useful reference is called out as primary

## Output Format

### Coverage Summary
- Total spec requirements extracted: N
- Covered: N
- Gaps: N (list each)

### Wiring Summary
- Dead ends: N (list each with prompt number and artifact)
- Missing prerequisites: N (list each)
- Implicit dependencies: N (list each)

### Completeness Assessment
Walk-through narrative: after all prompts execute, does the system match the spec? Call out any functional gaps.

### Issues by Severity

#### Critical
Issues that would cause the implementation to miss spec requirements or produce a broken system.

#### Major
Issues that would cause dead code, redundant work, or unclear boundaries between prompts.

#### Minor
Ordering improvements, wording clarity, or minor dependency refinements.

### Overall Rating
Pass / Needs Revision / Needs Major Revision

### Priority Recommendations
Top 3 fixes ordered by impact.
