## Project Purpose

`genl-opncoders-infra` is an **agentic infrastructure monorepo** for configuring, orchestrating, and standardising AI coding agents (OpenCode, GitHub Copilot, Gemini, Llama, Claude) across projects. Its primary goal is a **spec-first, multi-agent pipeline** — the Specfarm — that automatically generates constitutions, specifications, and implementation plans from raw TODO docs and source context.

---

## Core Rules / Principles

### 1. Spec-First, Always
- Every feature begins as a numbered TODO doc (`todo-NNN-slug.md` or plain `todo-NNN-slug`).
- The Specfarm pipeline converts TODOs → constitution → clarifications → spec → plan.
- No code is written without a corresponding spec and plan in `specs/NNN-slug/`.

### 2. Agent Hierarchy (Project > Global > System)
- **System**: Built-in defaults for each agent (OpenCode, Copilot, etc.).
- **Global**: `~/.config/opencode/opencode.json` — personal style, model choice, custom modes.
- **Project**: `./AGENTS.md` — team standards, stack-specific rules, injected at session start.
- Project-level config overrides global; global overrides system defaults.

### 3. System Prompt Architecture (todo-001)
- Monorepo agents MUST implement the **Monorepo Discovery Protocol**: detect affected packages, map dependencies, apply spec hierarchy.
- Prompts use the **Spec-Anchor Technique**: a `context.map` or `index.md` at repo root lists the canonical paths to root spec, task list, and API schema.
- Agent modes (plan / build / custom) are configured via the `agent` key in `opencode.json` and switchable with Tab in the TUI.
- Complex code blocks MUST be annotated with the requirement ID they satisfy.

### 4. File & Config Modes (todo-002)
- Configuration files follow a strict three-tier filemode hierarchy: **read-only system** → **global user** → **project override**.
- Operational specs are stored as `.md` files in `specs/NNN-slug/` (spec.md + plan.md).
- Config entries use `{file:path}` references instead of inline strings where possible.

### 5. Specfarm Pipeline
- Workflow: `specfarm.repoinit.yaml` (in `.github/workflows/`).
- Stages: `speckit.constitution` → `speckit.clarify` → `speckit.specify` → `speckit.plan`.
- Agent templates sourced from `spec-kit` v0.4.4 (pinned).
- Inference backend priority: self-inference (calling LLM) → Copilot relay → Haiku → Gemini Flash → Sonnet → Codex → Stub.
- Output: `.specify/memory/` (constitution + clarifications) and `specs/NNN-inferred/` (spec + plan per TODO).

### 6. Co-Agent Synergy
- All agents MUST read `.github/copilot-instructions.md` or `AGENTS.md` at session start.
- On task completion, append an `AI-CONTEXT:` handoff comment for the next agent.
- Spec takes precedence over Copilot Instructions for logic; Instructions take precedence for syntax/formatting.

---

## What to Build Next

1. **todo-001 — System Prompt Configuration**: Author the global `opencode.json` agent config and project `AGENTS.md` template implementing the Monorepo Discovery Protocol and Spec-Anchor Technique.
2. **todo-002 — File Mode Standards**: Define and document the filemode convention, enforce via pre-commit hooks and CI audit.
3. **Specfarm local runner**: A `bin/specfarm-local` script to run the full pipeline locally without GitHub Actions.
4. **`context.map` / index.md**: Create the spec-anchor index at repo root listing all canonical paths.
