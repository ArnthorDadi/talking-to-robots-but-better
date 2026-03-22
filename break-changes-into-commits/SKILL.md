---
name: break-changes-into-commits
description: Analyze git changes and group them into logical commits. Use when user wants to split changes into multiple commits or understand how to best organize their changes.
---

When asked to break changes into commits or organize git changes, follow this three-phase workflow.

## Phase 1: Create & Present Plan

### 1.1 Gather all changes

Run these commands in parallel:

- `git status --porcelain` - all changed files (staged + unstaged)
- `git diff --name-only` - unstaged changes
- `git diff --cached --name-only` - staged changes
- `git diff` - unstaged change content
- `git diff --cached` - staged change content
- `git log --oneline -10` - recent commits for style reference

### 1.2 Analyze and group files

Apply these heuristics to group files into commits:

**Grouping rules (apply in priority order):**

1. **Test files** always group with their source (e.g., `foo.test.ts` with `foo.ts`)
2. **Configuration files** related to a feature group together (e.g., `package.json` changes with new dependencies)
3. **Directory-based grouping** - files in the same directory with related purpose
4. **Change type grouping** - group by what the change does:
    - New feature (feat) → group related implementation
    - Bug fix (fix) → group the fix with its tests
    - Refactor → group all refactored files
    - Documentation → group all doc changes
    - Dependencies → group all dependency changes
5. **Atomic commits** - if changes are independent and affect different concerns, keep them separate

**Be decisive:** If a file could reasonably belong to multiple commits, pick the most logical one based on primary
purpose.

### 1.3 Present the plan

Display the grouping with reasoning for each group:

```
Found X files changed. Proposed grouping:

Group 1: "commit message" [REASON]
  Files: [file1.ts, file2.ts]

Group 2: "commit message" [REASON]
  Files: [file3.css, file4.scss]

... (remaining groups)
```

### 1.4 Get plan approval

Ask: "Does this plan make sense?"

**If Yes** → proceed to Phase 2

**If No** → accept modifications:

- Merge groups: "merge group 2 and 3"
- Move files: "move file X to group 1"
- Split group: "split group 2 into two"
- Remove group
- Add new group
- Reorder groups

Re-display plan with changes, re-ask until approved.

---

## Phase 2: Commit Loop

For each group in order:

### 2.1 Stage the files

Stage all files in the current group.

### 2.2 Get commit message

Call the `suggest-commit` skill with the list of files in this group.

### 2.3 Present for approval

Show the user:

- Files to be committed
- Proposed commit message
- Reasoning for this grouping

Ask: "Should I commit this?"

### 2.4 Handle response

**If Yes** → commit with the message, move to next group

**If No** → ask what to do:

- **Discard**: Unstage and remove from plan (user will handle manually)
- **Hold**: Unstage and mark as "pending" - will be addressed in Phase 3
- **Reassign**: Move files to another existing group (re-display that group's plan)
- **Edit message**: Let user write message directly, then re-ask "Should I commit this?"
- **Alternative message**: Call suggest-commit again (optionally with hint), present alternatives, let user pick, then
  re-ask

Loop back to 2.3 after any modification.

---

## Phase 3: Handle Remaining Files

After all groups processed, check for any remaining uncommitted/ungrouped files.

### 3.1 Acknowledge remaining files

List any files that weren't committed and ask: "What should I do with these remaining files?"

**If user sees no connection** → offer to create a catch-all group, then proceed through Phase 2 for that group

**If user says not ready** → acknowledge and exit. The files remain unstaged for them to handle later.

---

## Summary

After all commits, display:

```
Created X commits:

[short-hash] "commit message"
[short-hash] "commit message"
...

Y file(s) remaining (if any)
```
