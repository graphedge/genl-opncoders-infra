# Constitution: Monorepo Specification System

## Inferred Rules (from code, docs, and tests)

### 1. System Prompts (todo-001-sys-prompts)
- Monorepo requires specialized prompts to manage context drift across packages
- Prompts should emphasize dependency awareness and cross-package impact
- System prompts configured globally (~/.config/opencode/opencode.json) and per-project (./.AGENTS.md)
- Spec-anchor technique: index key specs in a context.map or index.md

### 2. File Modes (todo-002-filemodes)
- Configuration files managed via consistent filemode patterns
- Operational specs tracked in .md files
- Project-specific overrides in agent definitions

## Key Conventions
- Use spec-first approach for clarity
- Maintain hierarchy: root specs → package specs → README.md
- Annotate complex code with requirement IDs
