# Code Style & Editing Discipline for SpecFarm2

## Overview
This document establishes editing rules for SpecFarm2 agents (OpenCode, Copilot, etc.) to prevent wholesale file rewrites and encourage surgical, patch-first edits. **Always prioritize minimal, targeted changes over full-file replacements.**

---

## Editing Rules (MANDATORY)

1. **Never rewrite an entire file unless:**
   - Creating a brand new file, OR
   - A complete refactor is **explicitly requested** in the task description

2. **For any modification to existing files:**
   - Output changes **exclusively as a unified diff** or **search/replace block**
   - Include 3 lines of context (before/after) around each change
   - Prefer small, targeted hunks (5–15 lines max per change block)

3. **Exploration before editing:**
   - Always use read tools (`view`, `grep`, `bash cat`) to examine the file first
   - Never propose edits without seeing the current state
   - Search for related functions/imports before making changes

4. **Diff formatting:**
   - Use proper unified diff syntax with `@@` markers:
     ```
     @@@ -line_start,count +line_start,count @@
     context_line
     -removed_line
     +added_line
     context_line
     ```
   - Do NOT include placeholders like `// ... rest of file unchanged`
   - Do NOT output complete file content in search/replace unless creating new file

5. **Tool usage (SpecFarm2):**
   - **Preferred**: `edit` tool with targeted search/replace blocks
   - **Acceptable**: Unified diff format for review
   - **Avoid**: Creating new versions of existing files; use patch tools instead

---

## Preferred Editing Workflow

### Step 1: Read & Analyze (Read-Only)
```bash
# Examine current state
view /path/to/file [start_line, end_line]
grep -n "function_name" /path/to/file
```

### Step 2: Plan Changes (Design Phase)
Document which lines will change and why:
- Reference spec requirement or task ID
- List line numbers (before editing)
- Describe minimal change needed

### Step 3: Apply Changes (Patch-First)
Use `edit` tool with exact search/replace blocks:
```
old_str: |
  existing_line_1
  existing_line_2
  existing_line_3
new_str: |
  existing_line_1
  modified_line_2
  existing_line_3
```

### Step 4: Validate
```bash
bash tests/run_all_tests.sh
```

---

## Examples

### ✅ GOOD: Targeted Edit
**Task:** Add timeout handling to `ensure_token()` function

**Approach:**
1. View the function (3 lines context)
2. Identify the exact line to modify
3. Use `edit` tool with minimal search/replace:
   ```
   old_str: |
     local timeout=30
     token_refresh "$token"
   new_str: |
     local timeout=30
     timeout 30 token_refresh "$token" || { echo "Token refresh timeout"; return 1; }
   ```
4. Run tests
5. Report: `[token-timeout-fix] done | 45% → 92% tests passing`

### ❌ BAD: Wholesale Rewrite
**Approach (AVOID):**
1. Output entire 200-line file with one function modified
2. Include placeholder comment "// ... rest of file unchanged"
3. No test verification
4. Report: "Function updated"

**Why it fails:**
- Impossible to review changes
- High risk of accidental deletions
- Wastes tokens on unchanged code
- Breaks version control history

---

## Cross-Platform Considerations

### Path Normalization
- Use shell variable expansion: `$HOME`, `${REPO_ROOT}`, not hardcoded paths
- For Windows: normalize to forward slashes in scripts; use `.sh` + `.ps1` wrappers

### Testing Both Platforms
- Bash scripts: verify with `bash -n` (syntax check) + test suite
- PowerShell: verify `.ps1` wrapper exists and runs equivalent logic

---

## Review Checklist (Before Committing)

- [ ] Each `edit` block is 5–15 lines max (excluding context)
- [ ] Context lines (before/after) are included
- [ ] No placeholder comments ("// ... rest unchanged")
- [ ] File is NOT rewritten in full (unless explicitly requested as new file)
- [ ] All changes reference a spec requirement or task ID
- [ ] Tests pass: `bash tests/run_all_tests.sh`
- [ ] Diffs are reviewable in git (minimal + clear)

---

## When Full Rewrites ARE Acceptable

1. **Creating a new file** — no previous version exists
2. **Complete refactor explicitly requested** — user says "refactor X completely"
3. **Language/framework migration** — switching from one tech to another
4. **Major architectural change approved in spec** — documented in specs/ directory

Even then: **Stage changes in feature branch, get review approval, commit with detailed message.**

---

## Agent Behavior Reference

- **@build: Code / Implement** → Use surgical edits; report completion quantitatively
- **@build: Clarify / Improve** → Ask clarifying questions; refine scope; avoid unnecessary full rewrites
- **@build: Surgical Edits** → Always read first; patch only; validate with tests
- **Safety & Validation** → See `specfarm2/AGENTS.MD` for cross-package impact analysis

---

## Related Documents
- `specfarm2/AGENTS.MD` — Agent behavior and directive handling
- `specs/002-filemodes/spec.md` — File modes & configuration standards
- `tests/run_all_tests.sh` — Validation test suite

---

*Basis: specs/002-filemodes/spec.md (strategies 1–5 for preventing wholesale rewrites)*
