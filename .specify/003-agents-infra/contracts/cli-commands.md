# Contract: CLI Commands Interface

**Version**: 1.0  
**Status**: Final  
**Scope**: OpenCode command-line interface for agent management

---

## Command: `opencode agent list`

### Purpose
Lists all discovered agents (both repo-level and global) with their metadata.

### Syntax
```bash
opencode agent list [--format json|table|short]
```

### Options
- `--format json` — Output as JSON (machine-readable)
- `--format table` — Output as formatted table (default)
- `--format short` — Compact one-per-line format

### Output (table format)
```
Available Agents:

PRIMARY AGENTS (switchable via UI):
- Default Agent                  gpt-5.2-codex           Repo
- Spec Writer                    claude-haiku-4.5        Repo
- Build Pro                      gpt-5.2-codex           Repo
- Security Auditor               claude-sonnet-4.5       Global

SUBAGENTS (callable via @mention):
- code-reviewer                  claude-haiku-4.5        Repo
- speckit.specify                claude-haiku-4.5        Global
- speckit.plan                   claude-haiku-4.5        Global
- speckit.clarify                claude-haiku-4.5        Global

Total: 8 agents (4 primary + 4 subagent, 5 repo + 3 global)
```

### Output (JSON format)
```json
{
  "total": 8,
  "primary": 4,
  "subagents": 4,
  "repo_count": 5,
  "global_count": 3,
  "agents": [
    {
      "id": "specwriter",
      "name": "Spec Writer",
      "description": "Specializes in writing feature specifications...",
      "model": "claude-haiku-4.5",
      "is_primary": true,
      "location": "repo",
      "priority": 10,
      "tags": ["planning", "specification"]
    },
    ...
  ]
}
```

### Exit Codes
- `0` — Success
- `1` — General error (e.g., registry load failed)
- `2` — No agents found

---

## Command: `opencode agent info <name>`

### Purpose
Display detailed information about a specific agent.

### Syntax
```bash
opencode agent info <agent-id> [--with-details]
```

### Arguments
- `<agent-id>` — The agent ID (e.g., "specwriter", "buildpro")

### Options
- `--with-details` — Include full role, capabilities, constraints, and handoffs

### Output (default)
```
Agent: Spec Writer (specwriter)
Location: .opencode/agents/specwriter.md
Model: claude-haiku-4.5
Is Primary: Yes
Priority: 10
Tags: planning, specification, writing

Description:
  Specializes in writing feature specifications, plans, and documentation.

Role:
  I am Spec Writer. My specialty is creating clear, comprehensive feature
  specifications that drive engineering decisions...

Capabilities: 4
  - Writing feature specifications
  - Creating implementation plans
  - Identifying edge cases
  - Orchestrating handoffs

Constraints: 4
  - no-implement (handoff)
  - no-py (file_pattern)
  - no-js (file_pattern)
  - no-bash (command)

Handoffs: 3
  - write-spec → speckit.specify (with context)
  - clarify-spec → speckit.clarify (with context)
  - create-plan → speckit.plan (with context)
```

### Exit Codes
- `0` — Success
- `1` — Agent not found
- `2` — Invalid agent ID format

---

## Command: `opencode agent validate <name>`

### Purpose
Validate a specific agent definition for correctness.

### Syntax
```bash
opencode agent validate <agent-id> [--strict]
```

### Arguments
- `<agent-id>` — The agent ID to validate

### Options
- `--strict` — Treat warnings as errors (fail validation)

### Output (success)
```
✓ Agent "specwriter" is valid

Validation Results:
  ✓ YAML frontmatter: valid
  ✓ Required fields: all present
  ✓ Handoff targets: all exist
  ✓ Constraint patterns: all valid
  ✓ No circular handoff chains detected

Summary:
  - ID: specwriter
  - Model: claude-haiku-4.5
  - Handoffs: 3
  - Constraints: 4
  - Primary: Yes
```

### Output (error)
```
✗ Agent "specwriter" has validation errors:

File: .opencode/agents/specwriter.md
Line: 8

ERROR: Missing required field "model"
  The agent frontmatter is missing the "model" field.
  Add: model: gpt-5.2-codex

ERROR: Handoff target "badagent" does not exist
  Handoff: write-spec
  Fix: Check spelling or create agent "badagent"

ERROR: Circular handoff chain detected: specwriter → builder → specwriter
  Break the cycle by removing one of the handoffs.

Total Errors: 3
```

### Exit Codes
- `0` — Validation passed
- `1` — Validation failed (errors found)
- `2` — Agent not found

---

## Command: `opencode agent validate-all`

### Purpose
Validate all discovered agents and report issues.

### Syntax
```bash
opencode agent validate-all [--format summary|detailed] [--strict]
```

### Options
- `--format summary` — Show only counts and critical errors (default)
- `--format detailed` — Show detailed validation results for each agent
- `--strict` — Treat warnings as errors

### Output (summary format)
```
Validating all agents...

Summary:
  Total agents: 8
  Valid: 7
  Warnings: 1
  Errors: 1

✓ Agents (7):
  - specwriter
  - buildpro
  - planpro
  - code-reviewer
  - speckit.specify
  - speckit.plan
  - speckit.clarify

⚠ Warnings (1):
  - docwriter: Schema version "1.0" may be outdated

✗ Errors (1):
  - testmaster: Circular handoff chain detected (testmaster → builder → testmaster)

Exit with status 1 if any errors found. Warnings only if --strict flag used.
```

### Exit Codes
- `0` — All agents valid
- `1` — One or more agents have errors
- `2` — --strict flag used and warnings found

---

## Command: `opencode agent graph`

### Purpose
Display the handoff dependency graph as ASCII or JSON.

### Syntax
```bash
opencode agent graph [--format text|dot|json] [--cycles]
```

### Options
- `--format text` — ASCII text graph (default)
- `--format dot` — Graphviz DOT format
- `--format json` — JSON adjacency list
- `--cycles` — Highlight circular chains

### Output (text format)
```
Agent Dependency Graph:

specwriter
  ├─→ speckit.specify (write-spec)
  ├─→ speckit.clarify (clarify-spec)
  └─→ speckit.plan (create-plan)

buildpro
  ├─→ specwriter (request-spec)
  └─→ buildpro (recursive-build)  [⚠ CIRCULAR]

planpro
  ├─→ speckit.tasks (create-tasks)
  └─→ speckit.checklist (create-checklist)

speckit.specify
  └─→ speckit.plan (build-plan)

speckit.plan
  ├─→ speckit.tasks (create-tasks)
  ├─→ speckit.clarify (clarify)
  └─→ speckit.checklist (create-checklist)

Circular chains detected: 1
  - buildpro → buildpro
```

### Exit Codes
- `0` — Graph generated successfully
- `1` — Error generating graph (invalid handoff references)

---

## Agent Discovery Behavior

All commands follow these discovery rules:

1. **Scan Locations** (in order):
   - `.opencode/agents/` (repo-level)
   - `~/.opencode/agents/` (global)

2. **Precedence**: Repo-level agent overrides global agent with same ID

3. **Caching**: Agents are discovered once at command start and cached. To force refresh, restart OpenCode or run `opencode agent refresh` (if implemented).

4. **Error Handling**:
   - If `.opencode/agents/` doesn't exist: skip, continue with global
   - If `~/.opencode/agents/` not readable: log warning, continue
   - If agent definition is malformed: log error, skip agent
   - If validation fails: log error, agent still available but marked invalid

---

## Error Messages & Recovery

### Common Errors

**ERROR: Agent not found**
```
Error: Agent "specwriter" not found
Available agents:
  - default
  - buildpro
  - planpro

Did you mean: writer? (similar name)
```
Recovery: Check spelling or run `opencode agent list` to see available agents.

**ERROR: Invalid agent ID format**
```
Error: Invalid agent ID "my agent" (contains space)
Agent IDs must match: [a-z0-9_-]+
Example: "my-agent" or "my_agent"
```
Recovery: Use valid character format for agent ID.

**ERROR: Circular handoff chain**
```
Error: Circular handoff chain detected:
  specwriter → buildpro → specwriter
This would cause infinite loops during handoff execution.
Fix by removing one of these handoffs:
  - specwriter handoff to buildpro
  - buildpro handoff to specwriter
```
Recovery: Edit agent definition to remove one of the circular links.

**ERROR: Registry load failed**
```
Error: Failed to load agent registry
  Directory: .opencode/agents/
  Reason: Permission denied

Check file permissions or run with:
  chmod 755 .opencode/agents/
```
Recovery: Fix permissions or contact repository owner.

---

## Integration with OpenCode TUI

The agent list/switching feature in the TUI (Shift+Tab) uses these same discovery rules and commands internally. When user presses Shift+Tab:

1. System calls `opencode agent list --format json`
2. Filters to only `is_primary: true` agents
3. Sorts by priority
4. Displays in agent switcher dialog
5. User selects agent
6. System stores selection in `~/.opencode/session.json`
7. Agent switched on next message

---

## Backwards Compatibility

All commands follow semantic versioning:
- `v1.0`: Initial release (MVP)
- `v1.x`: New commands/options added (backward compatible)
- `v2.0`: Breaking changes (if needed)

MVP (v1.0) commands:
- ✅ `agent list`
- ✅ `agent info <name>`
- ✅ `agent validate <name>`
- ✅ `agent validate-all`

Phase 2 additions:
- 🔜 `agent graph`
- 🔜 `agent create [template]`
- 🔜 `agent refresh`
- 🔜 `agent test`

---

## Exit Code Convention

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General error (validation failed, agent not found, etc.) |
| `2` | Invalid arguments or missing required parameter |
| `3` | Permission denied (can't read registry or agent files) |
| `4` | Internal error (unexpected exception) |

---

## Performance Targets

From spec SC-002: Agent discovery must complete in < 2 seconds for 50 agents

| Command | Target | Scalability |
|---|---|---|
| `agent list` | 100ms | O(n) where n = agents |
| `agent info` | 50ms | O(1) lookup |
| `agent validate` | 500ms | O(n) where n = handoffs |
| `agent validate-all` | 1s | O(n²) where n = agents |
| `agent graph` | 200ms | O(n + e) where e = edges |

Implementation strategy: In-memory registry, lazy validation, caching.

---

## Summary

The CLI commands interface provides full agent management capabilities:

- ✅ Discovery and listing
- ✅ Detailed information
- ✅ Validation and error reporting
- ✅ Dependency visualization
- ✅ Clear error messages and recovery guidance

All commands follow Unix conventions (exit codes, stderr for errors, stdout for output).
