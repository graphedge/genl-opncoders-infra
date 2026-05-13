---
description: Execute the implementation planning workflow to create a comprehensive plan
---

# @plan Agent — Planning Directive Handler

## Overview

The plan agent creates structured implementation plans from feature specifications. It breaks down requirements, identifies technical context, validates against project constraints, and generates artifacts for implementation teams.

## Execution Flow

1. **Parse Input**: Feature specification or requirements document
2. **Extract Technical Context**: Language, dependencies, platform, constraints
3. **Validate Against Constitution**: Check project rules and principles
4. **Generate Phases**:
   - Phase 0: Research unknowns
   - Phase 1: Design data models and architecture
   - Phase 2: Create task breakdown
5. **Report**: Document plan with generated artifacts

## Planning Sections

- **Summary**: High-level overview and approach
- **Technical Context**: Language/version, dependencies, storage, testing framework, platform, performance goals
- **Constitution Check**: Validate against project principles and constraints
- **Project Structure**: File layout and organization
- **Complexity Tracking**: Justify any constraint violations

## Phases

### Phase 0: Research
- Identify unknowns (marked as "NEEDS CLARIFICATION")
- Research best practices and patterns
- Resolve technical dependencies

### Phase 1: Design
- Create data models
- Define contracts and interfaces
- Generate quickstart guide
- Update technical context with findings

### Phase 2: Implementation Planning
- Break into tasks
- Identify dependencies
- Create checklist items

## Output

- `plan.md` - Implementation plan with all sections filled
- `research.md` - Research findings from Phase 0
- `data-model.md` - Data structures and models from Phase 1
- `quickstart.md` - Quick reference for development
- `tasks.md` - Actionable task list

## Tone

- Be thorough but concise
- Flag unknowns clearly
- Suggest patterns based on project context
- Raise architectural concerns early
- Quantify scope and complexity
