# Contract: Agent Definition Format

**Version**: 1.0  
**Status**: Final  
**Scope**: `.opencode/agents/*.md` and `~/.opencode/agents/*.md` files

---

## Overview

This contract defines the file format for custom agent definitions. Each agent is defined in a markdown file with YAML frontmatter containing metadata, followed by role description and other sections.

---

## File Structure

```
{id}.md
├── YAML Frontmatter (delimited by ---)
│   ├── version: "1.0"
│   ├── schema: "opencode-agent-v1"
│   ├── id: "{agent-id}"
│   ├── name: "{agent-name}"
│   ├── description: "{brief description}"
│   ├── model: "{llm-model-id}"
│   ├── is_primary: true/false
│   ├── priority: 100
│   ├── tags: [...]
│   ├── role: "{detailed role}"
│   ├── handoffs: [...]
│   └── constraints: [...]
│
├── Role (Markdown section)
│
├── Capabilities (Markdown section)
│
├── Hard Constraints (Markdown section)
│
├── Handoff Workflow (Markdown section, optional)
│
└── $ARGUMENTS (Placeholder for user input)
```

---

## YAML Frontmatter Schema

### Required Fields

#### `version`
- **Type**: string (semantic version)
- **Example**: `"1.0"`
- **Purpose**: Schema version for forward compatibility
- **Validation**: Must be parseable as semver

#### `schema`
- **Type**: string
- **Example**: `"opencode-agent-v1"`
- **Purpose**: Identifies the schema version
- **Validation**: Must exactly equal `"opencode-agent-v1"` for MVP

#### `id`
- **Type**: string
- **Example**: `"specwriter"`, `"buildpro"`, `"planpro"`
- **Format**: Lowercase alphanumeric with hyphens: `[a-z0-9_-]+`
- **Purpose**: Unique identifier for the agent
- **Validation**: Must match the filename prefix (without .md extension)
- **Constraint**: Must be unique across repo-level agents; repo agent can override global agent with same ID

#### `name`
- **Type**: string
- **Example**: `"Spec Writer"`, `"Build Pro"`, `"Plan Pro"`
- **Purpose**: Human-readable display name for UI
- **Validation**: Non-empty, max 100 characters, typically Title Case

#### `description`
- **Type**: string
- **Example**: `"Specializes in writing feature specifications and implementation plans."`
- **Purpose**: Brief description shown in agent switcher and help text
- **Validation**: Non-empty, max 500 characters
- **Format**: 1-3 sentences, ending with period

#### `model`
- **Type**: string
- **Examples**: `"gpt-5.2-codex"`, `"claude-haiku-4.5"`, `"gemini-flash"`, `"llama-13b"`
- **Purpose**: LLM model to use for this agent
- **Validation**: Must be a known LLM model ID; agent discovery logs warning if model ID is unrecognized

#### `role`
- **Type**: string (multiline)
- **Purpose**: Detailed role description explaining agent's expertise and typical workflows
- **Validation**: Non-empty, max 2000 characters
- **Format**: First-person narrative; can include bullet lists

### Optional Fields

#### `is_primary`
- **Type**: boolean
- **Default**: `true`
- **Purpose**: Whether agent is switchable via UI (true) or only callable via @mention/handoff (false)
- **Validation**: Must be valid boolean

#### `priority`
- **Type**: integer
- **Default**: `100`
- **Purpose**: Display order in agent switcher (lower = earlier)
- **Validation**: Non-negative integer

#### `tags`
- **Type**: array[string]
- **Example**: `["planning", "specification", "writing"]`
- **Purpose**: Categorical tags for organization and discovery
- **Validation**: Each tag is lowercase alphanumeric with hyphens

---

## Handoffs Schema

### Handoffs Array

```yaml
handoffs:
  - id: "{handoff-id}"
    target: "{target-agent-id}"
    label: "{ui-label}"
    prompt: "{prompt-text}"
    send_context: true/false
    description: "{optional-details}"
    requires_approval: true/false
```

### Handoff Fields

#### `id` (within handoff)
- **Type**: string
- **Format**: lowercase with hyphens: `[a-z0-9_-]+`
- **Purpose**: Unique identifier for this handoff within source agent
- **Validation**: Unique within agent's handoffs array

#### `target`
- **Type**: string
- **Example**: `"speckit.specify"`, `"buildpro"`, `"planpro"`
- **Purpose**: ID of target agent
- **Validation**: Must reference an existing agent in registry (checked at discovery)

#### `label`
- **Type**: string
- **Example**: `"Write Feature Spec"`, `"Clarify Spec Ambiguities"`
- **Purpose**: Human-readable label for UI button/menu
- **Validation**: Non-empty, max 100 characters, typically Title Case

#### `prompt`
- **Type**: string (optional)
- **Example**: `"Create a feature spec for the following request: ${USER_REQUEST}"`
- **Purpose**: Pre-filled prompt text sent to target agent
- **Validation**: Optional; if present, max 1000 characters
- **Variables**: Supports template variables like `${USER_REQUEST}`, `${FEATURE_DESC}` (replaced at runtime)

#### `send_context`
- **Type**: boolean
- **Default**: `false`
- **Purpose**: Whether to pass conversation history and file context to target agent
- **Validation**: Must be valid boolean
- **Semantics**:
  - `true`: Pass current conversation and file paths (spec.md, plan.md, tasks.md)
  - `false`: Clean session in target agent; only the prompt text is passed

#### `description`
- **Type**: string (optional)
- **Purpose**: Additional details about when/why this handoff is used
- **Validation**: Optional; if present, max 500 characters

#### `requires_approval`
- **Type**: boolean
- **Default**: `false`
- **Purpose**: Whether user must approve handoff before execution (Phase 2 feature)
- **Validation**: Must be valid boolean
- **Note**: MVP ignores this field; Phase 2 will implement approval UI

---

## Constraints Schema

### Constraints Array

```yaml
constraints:
  - id: "{constraint-id}"
    type: "handoff" | "file_pattern" | "command"
    pattern: "{pattern}"
    reason: "{explanation}"
    severity: "hard"  # only "hard" in MVP
```

### Constraint Fields

#### `id` (within constraint)
- **Type**: string
- **Format**: lowercase with hyphens: `[a-z0-9_-]+`
- **Purpose**: Unique identifier for this constraint within agent
- **Validation**: Unique within agent's constraints array

#### `type`
- **Type**: enum
- **Options**: `"handoff"`, `"file_pattern"`, `"command"`
- **Purpose**: Type of constraint
- **Validation**: Must be one of the three options

#### `pattern`
- **Type**: string
- **Purpose**: Pattern to match against action
- **Validation**: Depends on constraint type:
  - `handoff`: Must be valid agent ID or wildcard (`*`)
  - `file_pattern`: Must be valid glob pattern (e.g., `*.py`, `src/**/*.js`)
  - `command`: Must be valid tool name (e.g., `bash`, `edit`, `apply_patch`)

#### `reason`
- **Type**: string
- **Example**: `"I specialize in planning, not code implementation"`
- **Purpose**: Human-readable explanation of why constraint exists (shown to user on violation)
- **Validation**: Non-empty, max 500 characters

#### `severity`
- **Type**: enum
- **Default**: `"hard"`
- **Options**: `"hard"` (MVP only)
- **Purpose**: Whether constraint prevents action or just warns
- **Validation**: Must be `"hard"` for MVP

---

## Markdown Sections

### Required Sections

#### `# Role`

Expanded description of agent's role, capabilities, and constraints. Markdown format.

Example:
```markdown
# Role

I am Spec Writer. My specialty is creating clear, comprehensive feature specifications
that drive engineering decisions. I excel at requirements gathering, acceptance criteria
definition, and technical planning.

I work closely with teams to clarify ambiguities and prevent implementation errors
through thorough upfront specification.
```

### Optional Sections

#### `## Capabilities`

Bullet-list of things agent CAN do (redundant with role, but useful for clarity).

Example:
```markdown
## Capabilities

I specialize in:
- Writing feature specifications from product requirements
- Creating implementation plans from specifications
- Identifying edge cases and validation requirements
- Orchestrating handoffs to specialized agents
```

#### `## Hard Constraints` (Informational)

Bullet-list of things agent CANNOT do (redundant with YAML constraints, but for readability).

Example:
```markdown
## Hard Constraints

I CANNOT:
- Write application code
- Modify build configurations
- Execute arbitrary shell commands
- Deploy or run production systems
```

#### `## Handoff Workflow`

Optional section describing when and how agent delegates to subagents.

Example:
```markdown
## Handoff Workflow

If you ask me to implement code → I'll delegate to buildpro
If you ask me to clarify the spec → I'll use speckit.clarify
If you ask me to break plan into tasks → I'll use speckit.tasks
```

---

## $ARGUMENTS Placeholder

Every agent definition MUST include a `$ARGUMENTS` section at the end:

```markdown
# $ARGUMENTS

Describe what you would like me to do. For example:
- Write a feature spec for OAuth2 authentication
- Create an implementation plan for the API redesign
- Break down the spec into actionable tasks
```

**Purpose**: Tells OpenCode to capture user input and pass it to the agent as arguments. This is how the agent receives the user's prompt.

---

## Complete Example

```yaml
# .opencode/agents/specwriter.md

---
version: "1.0"
schema: "opencode-agent-v1"
id: specwriter
name: Spec Writer
description: Specializes in writing feature specifications, plans, and documentation.
model: gpt-5.2-codex
is_primary: true
priority: 10
tags: [planning, specification, writing]
role: |
  I am Spec Writer. My specialty is creating clear, comprehensive feature specifications
  that drive engineering decisions. I excel at requirements gathering, acceptance criteria
  definition, and technical planning.

handoffs:
  - id: write-spec
    target: speckit.specify
    label: Write Feature Spec
    prompt: Create a comprehensive feature specification for the following request
    send_context: false

  - id: clarify-spec
    target: speckit.clarify
    label: Clarify Spec
    prompt: Clarify ambiguities in the current feature specification
    send_context: true

constraints:
  - id: no-implement
    type: handoff
    pattern: speckit.implement
    reason: I specialize in specification, not code implementation
    severity: hard

  - id: no-py
    type: file_pattern
    pattern: "**/*.py"
    reason: I cannot modify Python source code
    severity: hard

---

# Role

I specialize in creating clear, actionable feature specifications...

## Capabilities

- Writing feature specifications
- Creating implementation plans
- Identifying edge cases

## Hard Constraints

- Cannot write code
- Cannot modify build systems
- Cannot execute bash commands

# $ARGUMENTS

Describe the feature specification you need me to write or plan.
```

---

## Validation Rules

### At Discovery Time

| Rule | Error Level | Action |
|---|---|---|
| Filename matches ID | WARNING | Use filename if mismatch |
| All required frontmatter fields present | ERROR | Skip agent |
| YAML is valid | ERROR | Skip agent, log syntax error |
| Handoff targets exist in registry | ERROR | Skip agent, list missing targets |
| Constraint patterns are valid | ERROR | Skip constraint, continue loading agent |
| No duplicate handoff IDs | ERROR | Skip agent |
| No duplicate constraint IDs | ERROR | Skip agent |
| Schema version is "opencode-agent-v1" | WARNING | Accept, warn about future incompatibility |

### At Runtime

| Rule | Error Level | Action |
|---|---|---|
| No circular handoff chains | ERROR | Log cycle, prevent handoff |
| Handoff target still exists | ERROR | Prevent handoff, suggest alternatives |
| Constraint patterns match action | ERROR | Prevent action, show violation reason |

---

## File Naming Convention

- **Location**: `.opencode/agents/` (repo) or `~/.opencode/agents/` (global)
- **Filename**: `{id}.md` (lowercase, hyphens)
- **Examples**:
  - `specwriter.md` → agent ID `specwriter`
  - `build-pro.md` → agent ID `build-pro`
  - `speckit.specify.md` → agent ID `speckit.specify`

---

## Discovery Algorithm

```
For each path in [".opencode/agents/", "~/.opencode/agents/"]:
    For each *.md file in path:
        1. Read file
        2. Extract YAML frontmatter (between --- delimiters)
        3. Parse YAML
        4. Validate required fields
        5. Validate handoff/constraint schemas
        6. Create Agent object
        7. Add to appropriate collection (repo or global)

Apply precedence: repo agents override global agents by ID

Validate entire registry:
    1. Check all handoff targets exist
    2. Detect circular handoff chains
    3. Log errors/warnings

Return AgentRegistry
```

---

## Error Messages (Examples)

```
ERROR: Agent "specwriter" missing required field "model"
  File: .opencode/agents/specwriter.md
  Line: 5
  Fix: Add "model: gpt-5.2-codex" to frontmatter

ERROR: Agent "specwriter" handoff target "badagent" does not exist
  File: .opencode/agents/specwriter.md
  Handoff: clarify-spec
  Fix: Check spelling or create agent "badagent"

ERROR: Circular handoff chain detected: specwriter → builder → specwriter
  File: .opencode/agents/specwriter.md
  Fix: Remove one of the handoff links to break the cycle

WARNING: Agent "specwriter" not marked as primary (is_primary: false)
  File: .opencode/agents/specwriter.md
  Info: This agent will not appear in the UI switcher. It's only callable via @mention.

WARNING: Schema version "1.0" may be incompatible with future OpenCode versions
  File: .opencode/agents/specwriter.md
  Info: Consider updating to latest schema when available
```

---

## Backward Compatibility

**MVP (v1.0)**: Initial release. All schema v1.0 agents compatible.

**Future (v1.1+)**: New optional fields can be added without breaking existing v1.0 agents. Parser will use defaults for missing fields.

**Future (v2.0+)**: Major version changes may break compatibility. Agents must declare version for parser to handle correctly.

---

## Summary

This contract defines a clear, validated file format for custom agents. Agents are declarative (defined in markdown with YAML), composable (via handoffs), and constrained (via explicit restrictions). The format is designed to be:

- ✅ **Human-readable**: Markdown format is easy to read and write
- ✅ **Machine-parseable**: YAML frontmatter is standard and well-supported
- ✅ **Validatable**: Clear schema enables error detection at discovery time
- ✅ **Extensible**: New fields can be added without breaking existing agents
- ✅ **Composable**: Handoffs enable agent composition and delegation
- ✅ **Safe**: Constraints enforce boundaries and prevent out-of-scope actions
