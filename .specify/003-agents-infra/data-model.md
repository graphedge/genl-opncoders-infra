# Data Model: Custom Agents Infrastructure

**Status**: Complete  
**Created**: 2024-12-20  
**Repository**: graphedge/genl-opncoders-infra  
**Branch**: 001-agents-infra

---

## Core Entities

### 1. Agent

Represents a specialized autonomous agent with defined capabilities, constraints, and handoff targets.

#### Attributes

| Attribute | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | Unique identifier for the agent (e.g., "specwriter", "buildpro"). Must match filename prefix. |
| `name` | string | Yes | Human-readable agent name (e.g., "Spec Writer", "Build Pro"). |
| `description` | string | Yes | Brief description of agent's purpose and capabilities (1-2 sentences). |
| `model` | string | Yes | LLM model ID (e.g., "gpt-5.2-codex", "claude-haiku-4.5", "gemini-flash"). |
| `role` | string | Yes | Detailed role description explaining primary purpose, domain expertise, typical workflows. |
| `is_primary` | boolean | No (default: true) | Whether agent is switchable via UI (primary) or only callable via @mention/handoff (subagent). |
| `location` | enum | Yes | "repo" (from `.opencode/agents/`) or "global" (from `~/.opencode/agents/`). Set during discovery. |
| `priority` | integer | No (default: 100) | Display order in agent switcher. Lower = earlier. |
| `tags` | array[string] | No | Optional tags for categorization (e.g., ["planning", "specification", "design"]). |
| `created_at` | timestamp | No | When agent definition was created. Set at discovery. |
| `updated_at` | timestamp | No | When agent definition was last modified. Set at discovery. |
| `last_used_at` | timestamp | No | When agent was last selected by user. Updated on each use. |

#### Relationships

- **Handoffs** → array of `Handoff` objects. Defines which agents this agent can delegate to.
- **Constraints** → array of `Constraint` objects. Defines actions this agent cannot perform.
- **Overrides** → Reference to global agent (if this is a repo-level override). Points to global `Agent.id` with same name.

#### Validation Rules

1. `id` must match `[a-zA-Z0-9_-]+` (filename-safe, lowercase with hyphens)
2. `name` must be non-empty, typically Title Case
3. `description` must be non-empty, max 500 characters
4. `model` must be a known LLM model ID or error on discovery
5. `is_primary: true` requires `description` and `role` (for UI display)
6. All handoff targets must reference existing agents in registry (validated at discovery)
7. Constraint patterns must be valid glob or regex per constraint type

#### State Transitions

```
CREATED (discovery) → VALIDATED (if passes rules) → REGISTERED (in AgentRegistry)
                   → INVALID (if fails rules) → LOGGED_ERROR (skipped from registry)

REGISTERED → SELECTED (user chooses via UI) → ACTIVE (processing message)
          → COMPLETED (message processed) → SELECTED (again)

REGISTERED → LAST_USED_UPDATED (when selected)
```

#### Example YAML

```yaml
# .opencode/agents/specwriter.md frontmatter
---
id: specwriter
name: Spec Writer
description: Specializes in writing feature specifications, plans, and documentation.
model: gpt-5.2-codex
is_primary: true
location: repo
priority: 10
tags: [planning, specification, writing]
role: |
  I am Spec Writer. My specialty is creating clear, comprehensive feature specifications
  that drive engineering decisions. I excel at requirements gathering, acceptance criteria
  definition, and technical planning. I work closely with teams to clarify ambiguities and
  prevent implementation errors through thorough upfront specification.
handoffs:
  - target: speckit.specify
    label: Write Feature Spec
    prompt: Create a feature spec for the following user request
    send_context: false
  - target: speckit.clarify
    label: Clarify Spec
    prompt: Clarify ambiguities in the current feature spec
    send_context: true
constraints:
  - type: handoff
    pattern: speckit.implement
    reason: "I cannot implement code; that's for builders"
  - type: file_pattern
    pattern: "**/*.py"
    reason: "I cannot modify source code files"
  - type: file_pattern
    pattern: "**/*.js"
    reason: "I cannot modify source code files"
---
```

---

### 2. Handoff

Represents a delegation from one agent to another. Defines the target agent, invocation trigger, and context passing behavior.

#### Attributes

| Attribute | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | Unique identifier for this handoff within source agent (e.g., "write-spec", "clarify"). |
| `source_agent_id` | string | Yes | ID of the agent performing the handoff (e.g., "specwriter"). |
| `target_agent_id` | string | Yes | ID of the agent receiving the handoff (e.g., "speckit.specify"). |
| `label` | string | Yes | Human-readable label for UI display (e.g., "Write Feature Spec"). |
| `prompt` | string | No | Pre-filled prompt text sent to target agent when handoff is triggered. |
| `send_context` | boolean | No (default: false) | If true, pass conversation history and file context to target agent. |
| `description` | string | No | Additional details about when/why this handoff is used. |
| `requires_approval` | boolean | No (default: false) | If true, user must approve handoff before execution. |

#### Relationships

- **Source Agent** → `Agent.id` (must be a registered agent)
- **Target Agent** → `Agent.id` (must be a registered agent; can be subagent)

#### Validation Rules

1. `source_agent_id` must exist in AgentRegistry (validated at source agent discovery)
2. `target_agent_id` must exist in AgentRegistry (validated at source agent discovery)
3. No circular handoff chains: A → B → A is invalid (detected via topological sort)
4. `label` must be non-empty, typically Title Case
5. `prompt` is optional but recommended (max 1000 characters)
6. If `send_context: true`, target agent must exist and be prepared to receive context

#### Example JSON

```yaml
# In specwriter agent:
handoffs:
  - id: write-spec
    target: speckit.specify
    label: Write Feature Spec
    prompt: |
      Create a comprehensive feature specification for:
      ${FEATURE_DESCRIPTION}
    send_context: false
    description: "Use when starting a new feature. Target writes spec.md from scratch."

  - id: clarify-spec
    target: speckit.clarify
    label: Clarify Spec
    prompt: "Clarify ambiguities in the current feature specification"
    send_context: true
    description: "Use when spec is unclear or incomplete. Target has context of current work."
    requires_approval: false
```

---

### 3. Constraint

Represents a restriction on what an agent can do. Types include: prohibited handoff targets, prohibited file operations, prohibited commands/actions.

#### Attributes

| Attribute | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | Unique identifier within agent (e.g., "no-implement", "no-prod-files"). |
| `agent_id` | string | Yes | ID of the agent this constraint applies to. |
| `type` | enum | Yes | Type of constraint: "handoff", "file_pattern", "command". |
| `pattern` | string | Yes | Pattern to match against action. Glob for file_pattern, agent name for handoff, command name for command. |
| `reason` | string | Yes | Human-readable explanation of why constraint exists (shown to user on violation). |
| `severity` | enum | No (default: "hard") | "hard" (prevent action) or "soft" (warn, allow if user confirms). |

#### Relationships

- **Agent** → `Agent.id` (the agent this constraint restricts)

#### Pattern Syntax

| Type | Pattern Format | Examples |
|---|---|---|
| `handoff` | Agent name (exact match) | `speckit.implement`, `buildpro` |
| `file_pattern` | Glob pattern | `*.py`, `src/**/*.js`, `node_modules/**` |
| `command` | Tool/command name | `bash`, `edit`, `apply_patch` |

#### Validation Rules

1. `type` must be one of: "handoff", "file_pattern", "command"
2. `pattern` must be valid for the type:
   - handoff: agent name exists in registry
   - file_pattern: valid glob syntax
   - command: known tool name
3. `reason` must be non-empty (max 500 characters)
4. No duplicate patterns for same agent+type (redundant)

#### State Transitions

```
DEFINED (in agent frontmatter) → VALIDATED (pattern syntax OK) → ENFORCED (at runtime)
                              → INVALID (pattern syntax error) → LOGGED_ERROR
```

#### Example YAML

```yaml
# In specwriter agent constraints:
constraints:
  - id: no-implement
    type: handoff
    pattern: speckit.implement
    reason: |
      I specialize in specification and planning, not code implementation.
      Please use this agent to define what should be built, then hand off to
      an implementation specialist (buildpro, implementation-team) to execute.
    severity: hard

  - id: no-py-files
    type: file_pattern
    pattern: "**/*.py"
    reason: "I cannot modify Python source files; code writing is out of scope"
    severity: hard

  - id: no-bash
    type: command
    pattern: bash
    reason: "I cannot execute arbitrary bash commands; use subagents for that"
    severity: hard
```

---

### 4. AgentRegistry

In-memory registry of all discovered agents with constraint rules. Acts as the single source of truth for agent availability and capabilities.

#### Attributes

| Attribute | Type | Description |
|---|---|---|
| `agents` | map[string, Agent] | All agents keyed by `agent_id` |
| `repo_agents` | set[string] | Set of agent IDs from repo-level location |
| `global_agents` | set[string] | Set of agent IDs from global location |
| `handoff_graph` | map[string, set[string]] | Adjacency list: agent_id → set of target agent IDs |
| `constraint_rules` | map[string, array[Constraint]] | All constraints keyed by agent_id |
| `last_updated` | timestamp | When registry was last populated |
| `last_search_paths` | array[string] | Paths that were scanned (for cache invalidation) |

#### Methods

```go
// Discover all agents from filesystem
Discover(repoPaths, globalPaths []string) error

// Get agent by ID
GetAgent(id string) (Agent, error)

// Validate a proposed handoff
ValidateHandoff(sourceID, targetID string) error

// Check if action violates constraint
CheckConstraint(agentID, actionType, pattern string) (allowed bool, reason string)

// Detect circular handoff chains
DetectCircularChains() [][]string  // returns cycles if found

// Validate entire registry
ValidateAll() []ValidationError

// Get all primary agents (for UI switcher)
GetPrimaryAgents() []Agent

// Find overridden agents (repo-level versions of global agents)
GetOverrides() map[string]Agent  // maps global agent ID → repo override Agent
```

#### Validation Rules

1. No duplicate agent IDs across locations (within same location, error; across locations, repo overrides global)
2. All handoff targets must exist in registry
3. No circular handoff chains (A → B → C → A)
4. All constraint patterns must be valid
5. No orphaned constraints (agent referenced in constraint must exist)

#### Example Usage (Pseudocode)

```go
registry := NewAgentRegistry()

// Discover agents from filesystem
err := registry.Discover(
    repoPaths: []string{".opencode/agents/"},
    globalPaths: []string{"~/.opencode/agents/"},
)
if err != nil { panic(err) }

// Get agent
agent := registry.GetAgent("specwriter")
fmt.Println(agent.Name, agent.Description)

// Validate handoff before executing
err := registry.ValidateHandoff("specwriter", "speckit.clarify")
if err != nil { return err }  // throws DeniedError

// Check constraint
allowed, reason := registry.CheckConstraint("specwriter", "handoff", "speckit.implement")
if !allowed {
    return fmt.Errorf("Constraint violation: %s", reason)
}

// Get primary agents for UI
primaries := registry.GetPrimaryAgents()
for _, agent := range primaries {
    fmt.Printf("[ ] %s - %s\n", agent.Name, agent.Description)
}
```

---

### 5. Session

Represents an active conversation between user and agent(s).

#### Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | string | Unique session ID (UUID or similar). |
| `current_agent_id` | string | ID of the agent currently active in this session. |
| `directory` | string | Repo directory path where session started. |
| `branch` | string | Git branch name at session creation. |
| `messages` | array[Message] | Ordered list of user/agent messages in this session. |
| `context_files` | map[string, string] | File paths to spec.md, plan.md, tasks.md (auto-derived from branch). |
| `created_at` | timestamp | When session was created. |
| `updated_at` | timestamp | When session was last modified. |
| `metadata` | map[string, string] | Arbitrary key-value data (e.g., task_id, pr_number). |

#### Relationships

- **Current Agent** → `Agent.id` (which agent is processing next message)
- **Messages** → array of `Message` objects (owned by this session)

#### State Transitions

```
CREATED → ACTIVE (user typing) → PROCESSING (agent executing) → ACTIVE → COMPLETED
       → AGENT_SWITCHED (new agent selected) → ACTIVE (session continues)
```

#### Note

Sessions are **SQLite-backed in OpenCode** (persistent). The Agent Switching feature does NOT require a new session — the same session.ID continues to be used, with `current_agent_id` updated on each switch. This preserves conversation history automatically.

---

## Relationships & Dependencies

```
Agent
├── Handoff[] (what this agent can delegate to)
│   └── target_agent_id → Agent (the delegate)
├── Constraint[] (what this agent cannot do)
│   └── action_type + pattern → enforcement at runtime
└── Overrides → Agent (global agent being overridden by repo version)

AgentRegistry
├── agents[] (all discovered agents)
├── handoff_graph (for cycle detection)
└── constraint_rules (for enforcement)

Session
├── current_agent_id → Agent (active agent)
├── messages[] (conversation history)
└── context_files (spec.md, plan.md, tasks.md paths)
```

---

## Key Design Decisions

### Decision 1: Agent ID vs. Agent Name

**Decision**: Use `id` (lowercase with hyphens) as the programmatic identifier; `name` is human-readable.

**Rationale**:
- IDs are stable and filename-friendly (used in `.opencode/agents/{id}.md`)
- Names can change without breaking references
- Example: `id: "specwriter"` but `name: "Spec Writer"`

---

### Decision 2: Constraint Severity (Hard vs. Soft)

**Decision**: MVP supports only "hard" constraints. Soft constraints (warn but allow) deferred to Phase 2.

**Rationale**:
- Hard constraints ensure safety and prevent out-of-scope actions
- Soft constraints add complexity (approval dialogs, logging)
- Can add soft constraints later without breaking existing hard constraints

---

### Decision 3: Handoff Context Passing

**Decision**: `send_context: true` passes conversation history; file paths are ALWAYS auto-derived from git branch.

**Rationale**:
- File paths are deterministic from branch name (via `get_feature_paths()` script)
- Passing file paths redundantly is unnecessary
- Conversation history is the only context that needs explicit passing
- Aligns with Speckit agent expectations

---

### Decision 4: Session Scoping

**Decision**: Last-selected agent stored per-repo directory (NOT per-branch).

**Rationale**:
- Simpler implementation and user mental model
- Most workflows use same agent across branches in a repo
- Per-branch scoping is rare and can be added later
- Matches OpenCode's session model

---

### Decision 5: Registry Precedence (Repo > Global)

**Decision**: If same agent name exists in both locations, repo version is used; global version is ignored.

**Rationale**:
- Matches Unix principle: project config overrides user config overrides system config
- Allows teams to customize global agents for their project
- Clear, predictable behavior

---

## Enums & Constants

### Agent Location
```go
enum AgentLocation {
    REPO = "repo"      // .opencode/agents/
    GLOBAL = "global"  // ~/.opencode/agents/
}
```

### Constraint Type
```go
enum ConstraintType {
    HANDOFF = "handoff"           // prohibited agent delegation
    FILE_PATTERN = "file_pattern" // prohibited file operations
    COMMAND = "command"           // prohibited tool/command execution
}
```

### Constraint Severity (MVP: only "hard")
```go
enum ConstraintSeverity {
    HARD = "hard"   // prevent action immediately
    SOFT = "soft"   // warn and allow if user confirms (Phase 2)
}
```

### Agent Mode
```go
enum AgentMode {
    PRIMARY = "primary"     // switchable via UI
    SUBAGENT = "subagent"   // callable only via @mention/handoff
}
```

---

## Data Validation & Error Handling

### Validation Errors

| Error | Severity | Recovery |
|---|---|---|
| Malformed YAML frontmatter | ERROR | Skip agent, log filename and error line |
| Missing required field (name, model, etc.) | ERROR | Skip agent, log missing field name |
| Agent ID doesn't match filename | WARNING | Use filename as ID; accept, warn to user |
| Handoff target not found | ERROR | Skip agent, list target IDs that don't exist |
| Circular handoff chain detected | ERROR | Skip agent, show cycle (A → B → C → A) |
| Constraint pattern syntax invalid | ERROR | Skip constraint, continue loading agent |
| Duplicate agent ID in same location | ERROR | Skip second agent, log conflict |
| Permission denied on location | WARNING | Continue, skip that location |

### Graceful Degradation

- If `.opencode/agents/` doesn't exist: log INFO, continue with global agents
- If `~/.opencode/agents/` not readable: log WARNING, continue without global agents
- If agent definition has errors: log ERROR, skip agent, registry continues with other agents
- If handoff target doesn't exist: log ERROR during discovery, agent still registered but handoff marked invalid

---

## Scalability Considerations

### Performance Targets (from spec)

- **Discovery + Registration**: < 2 seconds for 50 agents
- **Agent Switching**: < 5 seconds from UI selection to agent ready
- **Registry Lookup**: O(1) for `GetAgent(id)`, O(n) for cycle detection where n = number of handoffs

### Optimization Strategies

1. **Lazy Validation**: Validate agent at discovery time (fail-fast), but defer full handoff graph validation to registry population
2. **Caching**: Cache discovered agents in memory. Invalidate only when `.opencode/agents/` or `~/.opencode/agents/` modification detected
3. **Indexed Lookups**: Use map for O(1) agent lookup by ID
4. **Batch Cycle Detection**: Detect circular chains once at registry population, not per-handoff

---

## Schema Versioning

Agent definition format uses semantic versioning in frontmatter:

```yaml
---
version: 1.0  # bump if breaking changes to frontmatter schema
schema: opencode-agent-v1
---
```

**Future versions** (Phase 2+):
- v1.1: Add agent tags, priority, last_used_at fields
- v2.0: Support constraints inheritance, conditional handoffs, etc.

---

## Appendix: Complete Example Agent Definition

```yaml
# .opencode/agents/specwriter.md

---
version: 1.0
schema: opencode-agent-v1
id: specwriter
name: Spec Writer
description: Specializes in writing feature specifications, plans, and documentation.
model: gpt-5.2-codex
is_primary: true
priority: 10
tags: [planning, specification, writing]
role: |
  I am Spec Writer. My specialty is creating clear, comprehensive feature specifications
  that drive engineering decisions. I excel at:
  - Requirements gathering and clarification
  - Acceptance criteria definition
  - Technical planning and architecture design
  - Risk identification and mitigation strategies
  
  I work closely with teams to clarify ambiguities and prevent implementation errors
  through thorough upfront specification. I am NOT a code implementer — my domain is
  defining what should be built.
  
  When I encounter requests for code implementation, I delegate to appropriate agents
  (buildpro for build systems, speckit.implement for general implementation).

handoffs:
  - id: write-spec
    target: speckit.specify
    label: Write Feature Spec
    prompt: |
      Create a comprehensive feature specification for:
      ${USER_REQUEST}
      
      Structure the spec with: User Scenarios, Functional Requirements, Success Criteria, 
      Assumptions, and Edge Cases.
    send_context: false

  - id: clarify-spec
    target: speckit.clarify
    label: Clarify Spec
    prompt: "Clarify ambiguities in the current feature specification. Ask up to 5 clarification questions."
    send_context: true

  - id: write-plan
    target: speckit.plan
    label: Write Implementation Plan
    prompt: |
      Create an implementation plan for the current spec.
      I am building with: Node.js backend, React frontend, PostgreSQL database.
    send_context: true

  - id: analyze-artifacts
    target: speckit.analyze
    label: Analyze Spec & Plan Consistency
    prompt: "Analyze spec, plan, and tasks for consistency and completeness."
    send_context: true

constraints:
  - id: no-implement
    type: handoff
    pattern: speckit.implement
    reason: |
      I specialize in specification and planning. Code implementation is outside my scope.
      If you need implementation work, please hand off to an implementation specialist.

  - id: no-source-code
    type: file_pattern
    pattern: "**/*.{py,js,ts,go,rs,java}"
    reason: "I cannot modify source code files. Use implementation specialists for coding tasks."

  - id: no-build-files
    type: file_pattern
    pattern: "Makefile,*.sh,setup.py,build.gradle"
    reason: "Build system configuration is buildpro's domain, not mine."

  - id: no-bash
    type: command
    pattern: bash
    reason: "I cannot execute arbitrary bash commands. Use buildpro or execution specialists."

---

# Role (Expanded)

## Capabilities

I specialize in:
- ✓ Writing feature specifications from product requirements
- ✓ Creating implementation plans from specifications
- ✓ Identifying edge cases and validation requirements
- ✓ Writing technical documentation and user guides
- ✓ Orchestrating handoffs to specialized agents (builders, implementers, testers)

## Hard Constraints

I CANNOT:
- ✗ Write application code (that's for implementers)
- ✗ Modify build configurations (that's for buildpro)
- ✗ Execute arbitrary shell commands
- ✗ Deploy or run production systems

## Handoff Workflow

If you ask me to:
- **Implement code** → I'll delegate to `buildpro` (build system specialist)
- **Clarify the spec** → I'll use `speckit.clarify` to ask focused questions
- **Break plan into tasks** → I'll hand off to `speckit.tasks`
- **Check build system** → I'll refer you to `buildpro`

## How to Use Me

1. **Describe the feature**: "I want to add OAuth2 authentication to our SaaS platform"
2. **I'll ask clarifying questions**: via `speckit.clarify` if needed
3. **I'll write the spec**: Clear requirements, acceptance criteria, edge cases
4. **I'll create a plan**: Architecture, integration points, risks, task breakdown
5. **I'll hand off to builders**: Once spec/plan are solid

## Example Workflow

```
You: "Add rate limiting to the API gateway"

Me (Spec Writer):
1. Ask clarification: Per-user or per-IP? Soft or hard limit?
2. Write spec with requirements and acceptance criteria
3. Create plan for implementation
4. Hand off to buildpro: "Please implement according to this spec/plan"

buildpro:
1. Reviews spec and plan
2. Implements rate limiting
3. Writes tests
4. Hands off to deployment specialist
```

---

# $ARGUMENTS

Describe the feature specification you need me to write or plan. Include:
- High-level goal or problem statement
- Key constraints or requirements
- Technology stack (if relevant)
- Success criteria or acceptance conditions
```

---

## Conclusion

This data model provides a complete foundation for the Custom Agents Infrastructure feature. All entities are designed with:

- ✅ Clear validation rules to catch errors early
- ✅ Persistent storage support (SQLite for sessions, filesystem for definitions)
- ✅ O(1) lookup performance for common operations
- ✅ Extensibility for future features (versioning, soft constraints, etc.)
- ✅ Safety guarantees (constraint enforcement, circular chain detection)

Ready for implementation in Phase 2.
