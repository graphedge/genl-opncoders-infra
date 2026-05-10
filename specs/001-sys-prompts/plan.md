# Implementation Plan: System Prompts for Monorepo

## Objective
Establish standardized system prompts that prevent context drift in large monorepos by emphasizing dependency awareness and cross-package impact.

## Tasks
1. **Create global prompt template** (~/.config/opencode/opencode.json)
   - Define monorepo discovery protocol
   - Add task-list bias for auto-execution
   - Include complex language patterns

2. **Create project override mechanism** (./.AGENTS.md)
   - Team standards per project
   - Tech stack specific guidance
   - Mode-based agent configuration

3. **Implement spec-anchor technique**
   - Create context.map or index.md
   - Link root specs, task lists, API schemas
   - Use breadcrumbs for large monorepos

## Success Criteria
- Prompts reduce context confusion by 80%
- Cross-package impact clearly identified in implementation notes
- Spec hierarchy respected across all workflows
