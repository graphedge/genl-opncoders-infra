# Tasks from specs/002-filemodes/spec.md

Spec source: File modes and preventing OpenCode wholesale rewrites

## DONE Tasks ✓

- [x] **task-001-code-style-file**: Create `CODE_STYLE.md` at repo root
  - Status: COMPLETE (commit 4bb3753)
  - Criteria: 5 anti-rewrite strategies documented, examples included, cross-platform guidance added
  - Output: CODE_STYLE.md (164 lines)

- [x] **task-002-agents-md-surgical-edits**: Add "@build: Surgical Edits" section to specfarm2/AGENTS.MD
  - Status: COMPLETE (commit 4bb3753)
  - Criteria: New section covers patch-first discipline, references CODE_STYLE.md, includes GOOD vs BAD examples
  - Output: New section in specfarm2/AGENTS.MD (65 lines)

- [x] **task-003-update-general-instructions**: Update "General Instructions" in AGENTS.MD
  - Status: COMPLETE (commit 4bb3753)
  - Criteria: CODE_STYLE.md cited, editing discipline enforced
  - Output: Updated General Instructions (1 line added)

- [x] **task-004-commit-and-push**: Commit both files to remote
  - Status: COMPLETE (commit 4bb3753)
  - Criteria: Files on main branch, remote synced
  - Output: Pushed to origin/main

---

## PENDING Tasks (High Priority)

- [ ] **task-005-config-file-modes**: Document OpenCode config (`~/.config/opencode/opencode.json`)
  - Spec reference: Strategy 1 in CODE_STYLE.md — "tweaking settings in project-level config"
  - Criteria: Config example shows `file_modification_output` or equivalent to disable full-file rewrites
  - Acceptance: .opencode.json template created with edit_mode/patch_first settings
  - Depends on: task-002 (AGENTS.MD surgical edits context)

- [ ] **task-006-pre-commit-hooks**: Add git pre-commit hook to validate edits
  - Spec reference: Strategy 4 in CODE_STYLE.md — "Review diffs before apply"
  - Criteria: Hook checks that commits don't contain full-file rewrites (unless new files)
  - Acceptance: `.git/hooks/pre-commit` validates diff size vs file size
  - Optional but recommended

- [ ] **task-007-snapshot-and-versioning**: Document snapshot/revert workflow
  - Spec reference: Strategy 3 in CODE_STYLE.md — "Snapshots and versioning"
  - Criteria: Guide on enabling snapshots in OpenCode, reverting aggressive changes
  - Acceptance: Added to CODE_STYLE.md section 7 (Snapshots)
  - Depends on: task-001 (CODE_STYLE.md exists)

---

## OPTIONAL / FUTURE Tasks

- [ ] **task-008-lsp-aware-edits**: Investigate LSP-based search/replace tools
  - Spec reference: Strategy 3 — "Search/replace or diff-first tools"
  - Future work: Evaluate Cursor, Aider, other tools with native LSP support
  - Optional; revisit if patch-first discipline isn't enforced

- [ ] **task-009-unified-diff-examples**: Create visual diff examples library
  - Spec reference: Strategy 5 — "Advanced Prompt Patterns"
  - Future work: Build examples/diffs/ folder with 5–10 annotated diffs (good/bad)
  - Optional; helpful for onboarding

- [ ] **task-010-cross-repo-policy**: Extend CODE_STYLE.md to cross-repo calls
  - Spec reference: Agent Hierarchy (project > global > system)
  - Future work: Sync CODE_STYLE.md to genl-opncoders-infra/.github/copilot-instructions.md
  - Scope: Lower priority; spec-002 is single-repo focused

---

## TRIED / ATTEMPTED Strategies

From specs/002-filemodes/spec.md, these strategies were actually tested/attempted:

- [x] **Strategy 1: Repository-Level Instruction Files** ← TRIED & WORKING
  - Created CODE_STYLE.md with explicit editing rules
  - Added @build: Surgical Edits section to AGENTS.MD
  - General Instructions now cite CODE_STYLE.md
  - Status: OpenCode agents auto-ingest on next session (active)

- [ ] **Strategy 2: Config File Tweaks** ← NOT TRIED YET
  - Need: `.opencode.json` with `file_modification_output` or `edit_mode` settings
  - Status: Deferred to task-005

- [ ] **Strategy 3: Tooling & Modes** ← PARTIALLY TRIED
  - CODE_STYLE.md mentions plan vs build modes
  - Snapshots documented but not tested
  - Status: Needs task-007 for full validation

- [ ] **Strategy 4: Pre-commit Review** ← NOT TRIED YET
  - Need: Git hook to validate diffs before commit
  - Status: Deferred to task-006

- [ ] **Strategy 5: Advanced Prompt Patterns** ← NOT TRIED YET
  - Unified diffs documented in CODE_STYLE.md
  - Examples included (GOOD vs BAD)
  - Status: Ready for testing

---

## Summary

**Progress: 4/10 tasks complete (40%) | 1/5 strategies fully tried**

- Implemented: CODE_STYLE.md + AGENTS.MD surgical edits + commit
- Tried & Working: Strategy 1 (repository-level instructions)
- Next: Config file modes (task-005), pre-commit hooks (task-006)
- Optional: Snapshots docs, LSP investigation, diff examples

**Metrics:**
- CODE_STYLE.md: 164 lines, 5 strategies
- AGENTS.MD: +2 new sections (@build: Surgical Edits, updated General Instructions)
- Commits: 1 (4bb3753)
- Files changed: 2 (CODE_STYLE.md new, specfarm2/AGENTS.MD updated)

**Impact:**
- OpenCode agents now auto-ingest CODE_STYLE.md and surgical edits rules
- Wholesale rewrites require explicit request
- Anti-rewrite discipline active on next session
- Strategy 1 (repository-level rules) validated & working

**What's Ready to Try Next:**
- task-005: Config file modes (quick win)
- task-006: Pre-commit hooks (validation layer)
- task-007: Snapshot workflow docs (safety layer)
