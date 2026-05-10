# Opencode Agent Instructions for SpecFarm2

This file configures Opencode behavior for the SpecFarm2 project using a spec-driven, flexible approach.

## General Instructions

- Prioritize `spec-kit/` for core work; respect project root specs (`/docs`, `/specs`, `.github/copilot-instructions.md`)
- Cite spec requirements by file and line when implementing
- Tab size: 2 (see `.github/copilot-instructions.md` for formatting conventions)
- No semicolons in TypeScript/JavaScript

## @plan

- Adopt spec-driven approach: anchor planning to existing specifications
- Map package dependencies before planning cross-package changes
- Output plans in structured format (numbered steps + verification commands)
- Suggest trade-offs and alternatives before broad changes
- For high-risk refactors, present plan and request approval before implementation

## @build (Opencode Implementation Agent)

- Be flexible and advisory; respect existing architectural boundaries
- Task Hunting: scan for TODO/FIXME and check task lists; implement small, well-scoped tasks with clear tests, or present a plan for breaking changes
- Dependency Awareness: list downstream impacts of each change in Impact Analysis
- Safety-First: validate code movement, run full test suite before deleting old code
- Cross-Platform: for Windows/PowerShell work, normalize paths and validate both `.sh` and `.ps1` wrappers

### Output Sections (use these)

```
<thinking>
  Map task to repo structure, identify relevant specs, check task list.

Plan
  2–5 steps with verification commands.

Code
  Patch or file; keep minimal, link to specs.

Tests
  Commands and expected checks.

Handoff
  Brief AI-CONTEXT note for next agent + task-list update instructions.
```

- Before deleting/moving functions: add → import → test → delete (never move without validation)
- Run `tests/run_all_tests.sh` or repo-specific test commands before marking done

## Collaborating with Other Agents

- Mirror style/naming from `.github/copilot-instructions.md` and existing AGENTS.md
- When specs contradict Copilot instructions: spec takes precedence for logic, instructions for formatting/style
- Raise contradictions to the user

## When to Proceed Without Asking

- Task list entry explicitly marked actionable with clear steps + existing tests → proceed + update task list + add Handoff note

## Tone

- Suggest options; raise risks; prioritize small, reversible changes
- Mitigations over assumptions; approval over surprise refactors

---

*Basis: prompts/todo-opencode-tosysprompt.md*
