# OpenCode Agent Configuration for genl-opncoders-infra

## Directory Structure

```
.opencode/
├── README.md              # This file
├── AGENTS.md              # Project context + agent definitions (auto-injected)
├── build.agent.md         # @build directive handler (code implementation)
└── plan.agent.md          # @plan directive handler (planning & design)
```

## Usage

### Run @plan Agent
```bash
opencode @plan: Design feature X per specs/feature.md
```
Output: Structured plan with numbered steps, alternatives, and risk assessment.

### Run @build Agent
```bash
opencode @build: Implement task-id-1, task-id-2 per plan.md
```
Output: End-to-end implementation with full test validation.

### Auto-Injection
When you run `opencode` commands in this repository, the system automatically:
1. Loads `.opencode/AGENTS.md` → injects into system prompt
2. Routes `@plan` directives → `plan.agent.md` handler
3. Routes `@build` directives → `build.agent.md` handler

## Integration Points

- **Spec Reference**: `specfarm2/agents.md` (comprehensive agent behaviors)
- **Core Specs**: `.specify/`, `specs/` directories
- **Templates**: `.github/agents/` (plan-template.md, spec-template.md, etc.)
- **Testing**: `tests/run_all_tests.sh` (all agents validate against this)
- **Instructions**: `.github/copilot-instructions.md` (project style guide)

## Key Principles

1. **Spec-Driven**: All planning and implementation anchors to existing specs (file + line reference)
2. **Test-First**: Full test suite runs before marking tasks complete
3. **Quantified Output**: Completion reported as `% → % tests passing` + task IDs
4. **Risk-Aware**: High-risk changes presented as options before implementation
5. **Handoff Notes**: Each agent leaves `// AI-CONTEXT:` for the next agent

## Agent Handoff Flow

```
User Directive
    ↓
    ├─→ @plan agent
    │     └─→ Creates plan + task list
    │         └─→ Handoff to @build (or review)
    │
    ├─→ @build agent (if @plan approved)
    │     └─→ Implements per task IDs
    │         └─→ Validates with tests
    │             └─→ Reports completion
    │
    └─→ @clarify agent (if spec enhancement needed)
          └─→ Asks targeted questions
              └─→ Returns enhanced spec
```

## Examples

### Simple Code Implementation
```
User: @build: Implement run_drift_check() per specs/drift.md L42-50
Agent: [todo-tasks.md] [drift-impl, drift-test] done | 12% → 94% tests passing
```

### Complex Planning with Review
```
User: @plan: Design token refresh flow per specs/auth.md
Planner: [specs/auth-refresh-plan.md] created | [tasks/auth.md] staged
Reviewer: (optional) Plan review complete; suggestions added to plan
Implementation: Call @build agent with approved task IDs
```

## File Locations Reference

| Purpose | Location |
|---------|----------|
| Project agent config | `.opencode/AGENTS.md` (this directory) |
| Agent behaviors | `specfarm2/agents.md` |
| Core specifications | `.specify/`, `specs/` |
| Project instructions | `.github/copilot-instructions.md` |
| Agent templates | `.github/agents/` |
| Test suite | `tests/run_all_tests.sh` |

---

**Repository**: genl-opncoders-infra  
**Primary Language**: Bash, YAML, Markdown  
**Generated**: 2026-05-12
