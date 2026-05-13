# OpenCode Agents Configuration

This file is auto-injected into the OpenCode system prompt when running in this repository.

## Project Context

- **Repository**: genl-opncoders-infra
- **Primary Language**: Bash, YAML, Markdown
- **Package Structure**: Multi-package monorepo with specfarm2 subdirectory
- **Testing**: `tests/run_all_tests.sh` (bash-only, zero external dependencies)
- **Agent Base**: See `specfarm2/agents.md` for comprehensive agent behavior

## Core Agent Roles

### @plan Agent
- **Goal**: Spec-driven planning with structured output
- **Input**: User directive + target specification file
- **Output**: Numbered plan + task list updates
- **Validation**: Reference specs by file/line; suggest trade-offs before broad changes

### @build Agent
- **Goal**: Code implementation with full validation
- **Input**: Implementation directive + task ID(s)
- **Output**: End-to-end implementation + test report (% passing)
- **Validation**: Run full test suite before marking done

### @clarify Agent
- **Goal**: Enhance/extend spec with targeted questions
- **Input**: Spec markdown to improve
- **Output**: Enhanced spec with citations + handoff notes

## Default Behaviors

- **Test suite**: Always run `tests/run_all_tests.sh` before marking tasks complete
- **Tab size**: 2 spaces (see `.github/copilot-instructions.md`)
- **Spec precedence**: When specs conflict with copilot instructions, specs take precedence for logic
- **Handoff format**: Add `// AI-CONTEXT:` notes for next agent

## File Locations

- **Core specs**: `.specify/`, `specs/`
- **Project specs**: `specfarm2/agents.md`, `.github/copilot-instructions.md`
- **Templates**: `.github/agents/` (plan-template.md, spec-template.md, etc.)

## Cross-Agent Coordination

1. **Planner → Implementation**: Plan produces task list; implementation consumes task IDs
2. **Review cycle**: Planner → Review → Implementation (optional approval before code)
3. **Handoff notes**: Each agent leaves brief AI-CONTEXT for next agent

See `specfarm2/agents.md` L139–194 for detailed workflows.

---

*Generated for genl-opncoders-infra via OpenCode agent configuration*
