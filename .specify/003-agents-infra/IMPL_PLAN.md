# Implementation Plan: OpenCode Custom Agents Infrastructure

**Feature**: 003-agents-infra  
**Status**: Planning Phase  
**Created**: 2024-12-20  
**Target Completion**: Q1 2025

---

## Technical Context

### Known Architecture Decisions

1. **Agent Definition Format**: Markdown files with YAML frontmatter (matches existing OpenCode conventions)
   - Location: `.opencode/agents/` (repo-level) and `~/.opencode/agents/` (global)
   - Priority: Repo-level > global
   - Scope: Repo PR #003-agents-infra branch

2. **Agent Registry Pattern**: In-memory registry loaded at startup
   - Populated during OpenCode initialization
   - Accessible to command interpreter for validation/enforcement
   - Supports 50+ agents with sub-2-second discovery

3. **UI Integration**: Shift+Tab agent switcher in OpenCode TUI
   - Displays primary agents only
   - Preserves session context during switches
   - Restores last-used agent on session restart

4. **Constraint Enforcement Architecture**:
   - Declarative constraints in agent frontmatter
   - Runtime validation intercepting action attempts
   - Three constraint types: handoff, file_pattern, command
   - Logged violations for audit trail

### NEEDS CLARIFICATION

1. **Session Context Preservation** - NEEDS CLARIFICATION
   - How is conversation history preserved across agent switches?
   - What mechanisms exist in OpenCode for session state management?
   - Are there existing patterns for state serialization/deserialization?

2. **OpenCode TUI Integration Points** - NEEDS CLARIFICATION
   - Where is Shift+Tab agent switcher currently implemented?
   - What are the hooks for adding custom agents to the switcher?
   - How does agent selection trigger context switching in the TUI?

3. **Constraint Enforcement Interception** - NEEDS CLARIFICATION
   - How can we intercept handoff attempts, file operations, and command executions?
   - What hooks exist in the OpenCode command interpreter?
   - Are there existing patterns for pre-action validation?

4. **Speckit Integration Protocol** - NEEDS CLARIFICATION
   - What is the exact handoff payload format speckit agents expect?
   - How do we pass spec.md, plan.md, tasks.md paths to speckit agents?
   - Are there existing speckit.handoff or speckit.delegate endpoints?

5. **@Mention Parsing Implementation** - NEEDS CLARIFICATION
   - How should @mention syntax be parsed in user input?
   - Should this happen in the OpenCode CLI parser or in agent code?
   - What's the canonical format: @agent_name [task] or other?

6. **Agent Last-Used Persistence** - NEEDS CLARIFICATION
   - Where should the last-selected agent be stored? (~/.opencode/session or in-memory only?)
   - Should this be scoped per-repo or global?
   - How is session context keyed? (repo path, user, branch?)

### Dependencies & Technologies

- **OpenCode CLI Framework**: For agent discovery hooks, command registration, TUI integration
- **YAML Parsing**: For agent frontmatter validation and parsing
- **Markdown Parser**: For extracting agent metadata from `.md` files
- **Session State Management**: For preserving context across switches
- **Audit Logging**: For constraint violation tracking
- **Test Framework**: For validating agent definitions and handoff behavior

---

## Constitution Check

From `.specify/memory/constitution.md`:

✅ **Spec-First Always**: Feature has numbered spec.md, will generate plan.md before implementation  
✅ **Agent Hierarchy**: Aligns with Project > Global > System hierarchy  
✅ **System Prompt Architecture**: Custom agents can implement OpenCode modes  
✅ **File & Config Modes**: Agent definitions as `.md` in `.opencode/agents/` (project override layer)  
✅ **Specfarm Integration**: Custom agents hand off to speckit workflow agents  
✅ **Co-Agent Synergy**: Custom agents read AGENTS.md and handoff via AI-CONTEXT comments  

**No constitution violations detected.**

---

## Gates & Readiness

### Gate 1: Research Phase Completion
- **Target**: All NEEDS CLARIFICATION items resolved
- **Actions**:
  - Research OpenCode TUI architecture and session management
  - Investigate constraint enforcement patterns in existing CLI tools
  - Document speckit integration protocol
  - Define @mention parsing specification
  - Specify agent persistence strategy

### Gate 2: Design Approval
- **Target**: Technical design reviewed and approved
- **Artifacts**: data-model.md, contracts/, architectural decisions
- **Blockers**: If speckit integration protocol is incompatible, may require speckit API changes

### Gate 3: Dependency Availability
- **OpenCode CLI Hooks**: Required for agent discovery/switching
- **Speckit Agent Endpoints**: Required for handoff integration
- **Action**: Coordinate with OpenCode and speckit teams

---

## Phase 0: Research & Clarification

### Research Tasks

#### Task 0.1: OpenCode TUI Architecture & Session Management
**Goal**: Understand how OpenCode manages session state, UI components, and agent lifecycle

**Questions to Resolve**:
- How is the agent switcher (Shift+Tab) currently implemented?
- What mechanisms preserve conversation history across TUI operations?
- How can we hook into agent initialization/switching events?
- Are there existing patterns for session-scoped state?

**Deliverables**:
- Architecture summary of OpenCode TUI session management
- Identified hooks for agent lifecycle events
- Recommended approach for context preservation

---

#### Task 0.2: Constraint Enforcement Patterns
**Goal**: Design the architecture for intercepting and validating constrained actions

**Questions to Resolve**:
- How can we intercept handoff calls before execution?
- What patterns exist for validating file operations in CLI tools?
- How should constraint violations be logged and reported?
- Is middleware/interceptor pattern viable in OpenCode?

**Deliverables**:
- Constraint enforcement architecture diagram
- Pre-action validation flow
- Audit logging design

---

#### Task 0.3: Speckit Integration Protocol
**Goal**: Document how custom agents hand off to speckit agents with context

**Questions to Resolve**:
- What input format do speckit agents expect? (file paths, JSON, arguments?)
- Is there a speckit.handoff endpoint or should we call speckit agents directly?
- How do we pass spec.md, plan.md, tasks.md to speckit agents?
- What context should flow back from speckit agents?

**Deliverables**:
- Speckit handoff payload specification
- Integration examples (specwriter → speckit.plan)
- Protocol documentation

---

#### Task 0.4: @Mention Parsing Specification
**Goal**: Define the syntax and parsing logic for @agent_name handoff invocation

**Questions to Resolve**:
- Should parsing happen in CLI parser or in agent code?
- Canonical format: @agent_name [task] or @agent_name task?
- How do we distinguish @mention from markdown formatting?
- How should parsing errors be handled?

**Deliverables**:
- @mention syntax specification with examples
- Parsing algorithm and edge cases
- Integration points for parser in OpenCode

---

#### Task 0.5: Agent Persistence & Session Scoping
**Goal**: Design how last-selected agent is stored and restored

**Questions to Resolve**:
- Should last-selected agent be stored globally or per-repo?
- How is session identity managed? (directory, branch, user combination?)
- Should this use ~/.opencode/session or in-memory cache?
- How do we handle agent deletion between sessions?

**Deliverables**:
- Session storage design
- Agent restoration algorithm
- Edge case handling (deleted agent, moved repo, etc.)

---

#### Task 0.6: Best Practices for Custom Agent Design
**Goal**: Research existing agent frameworks and constraint patterns

**Questions to Resolve**:
- How do other multi-agent systems handle constraints?
- What are best practices for non-overlapping agent responsibilities?
- How should agents document their capabilities vs. constraints?
- Are there reference implementations we should learn from?

**Deliverables**:
- Best practices guide
- Design anti-patterns to avoid
- Reference examples from existing agents (specwriter, buildpro, planpro)

---

### Phase 0 Output
**Artifact**: `.specify/003-agents-infra/research.md`
- All NEEDS CLARIFICATION items resolved
- Decision rationale documented
- Alternatives considered for each design choice

---

## Phase 1: Design & Contracts

### 1.1 Data Model Definition

**Output**: `.specify/003-agents-infra/data-model.md`

**Entities**:

1. **Agent**
   - Attributes: `id`, `name`, `description`, `model`, `role`, `location` (repo|global), `is_primary` (bool)
   - Metadata: created_at, updated_at, last_used_at
   - Relationships: handoffs[], constraints[], overrides (global)
   - Validation: required fields, unique name per location

2. **Handoff**
   - Attributes: `id`, `source_agent_id`, `target_agent_id`, `label`, `prompt_template`, `send_context` (bool), `requires_approval` (optional)
   - Validation: target agent must exist, no circular chains, source can delegate to target

3. **Constraint**
   - Attributes: `id`, `agent_id`, `type` (handoff|file_pattern|command), `pattern`, `reason`
   - Validation: pattern syntax valid for type

4. **AgentRegistry**
   - Attributes: `agents` (map), `handoff_graph`, `constraint_rules`, `last_updated`
   - Methods: discover(), get_agent(), validate_handoff(), enforce_constraint()

5. **Session**
   - Attributes: `current_agent_id`, `context` (conversation history), `directory`, `branch`, `created_at`
   - Persistence: last_selected_agent stored in ~/.opencode/session.json per directory

### 1.2 Interface Contracts

**Output**: `.specify/003-agents-infra/contracts/`

#### Agent Definition Contract (agents.contract.md)
- Input: `.opencode/agents/*.md` files with YAML frontmatter
- Output: Parsed Agent objects in AgentRegistry
- Schema: 
  ```yaml
  ---
  name: agent-name
  description: Brief description
  model: gpt-5.2-codex or similar
  is_primary: true/false
  handoffs:
    - target: agent-name
      label: Human readable label
      send_context: true/false
  constraints:
    - type: handoff
      pattern: prohibited-agent-name
      reason: Why prohibited
  ---
  ```

#### Handoff Invocation Contract (handoffs.contract.md)
- Input: @mention syntax `@agent_name [task description]` or programmatic call
- Output: Handoff execution with context preservation if applicable
- Validation: target agent exists, source agent has permission, context format valid

#### Constraint Enforcement Contract (constraints.contract.md)
- Input: Agent action attempt (handoff, file operation, command)
- Output: Action allowed/blocked with reason
- Validation: Constraint patterns match action type, violation logged

#### CLI Commands Contract (cli.contract.md)
- Commands:
  - `opencode agent list` - Lists all available agents
  - `opencode agent info <name>` - Shows agent details
  - `opencode agent validate <name>` - Validates agent definition
  - `opencode agent validate-all` - Validates all agents

#### TUI Integration Contract (tui.contract.md)
- Shift+Tab agent switcher shows primary agents
- Agent selection triggers context switch
- Last-selected agent restored on session restart

### 1.3 Technical Design

**Output**: `.specify/003-agents-infra/design-decisions.md`

#### Architecture: Agent Lifecycle & Registry

```
Startup:
1. OpenCode CLI initializes
2. AgentDiscovery.scan([".opencode/agents/", "~/.opencode/agents/"]) 
3. For each agent.md: parse frontmatter → validate → add to AgentRegistry
4. Build handoff_graph and constraint_rules
5. Load last-selected agent from ~/.opencode/session.json

User Interaction:
1. User presses Shift+Tab
2. TUI displays AgentSwitcher with is_primary agents
3. User selects agent
4. OpenCode validates selection against constraints
5. Session context transferred to new agent
6. Last-selected agent saved to session.json
7. Agent ready to accept input

Handoff:
1. Source agent invokes @target_agent [task]
2. @mention parser extracts target and task
3. Validate: source.handoffs contains target AND target not in source.constraints
4. If send_context=true: pass spec.md, plan.md, tasks.md paths in arguments
5. Execute target agent with task
6. Return results to source agent
```

#### Constraint Enforcement Flow

```
Before any action:
1. Middleware intercepts action (handoff call, file op, command)
2. Look up source agent in AgentRegistry
3. Match action against agent.constraints
4. If constraint violated:
   a. Log violation (agent, action, constraint_reason)
   b. Raise ConstraintViolationError with clear message
   c. Prevent execution
5. If allowed:
   a. Execute action
   b. Continue normally
```

#### Agent Discovery Algorithm

```
discover():
  repo_agents = scan(".opencode/agents/")
  global_agents = scan("~/.opencode/agents/")
  
  registry = {}
  for each (name, agent) in global_agents:
    registry[name] = agent
    
  for each (name, agent) in repo_agents:
    registry[name] = agent  // override global
  
  for each agent in registry.values():
    validate(agent)
    validate_handoffs(agent)
    
  return registry
```

### 1.4 Quickstart for Agent Developers

**Output**: `.specify/003-agents-infra/quickstart.md`

```markdown
# Creating Your First Custom Agent

## Step 1: Create Agent Definition File

Create `.opencode/agents/my-agent.md`:

\`\`\`markdown
---
name: my-agent
description: My specialized agent for task X
model: gpt-5.2-codex
is_primary: true
handoffs:
  - target: specwriter
    label: "Delegate to spec writer"
    send_context: true
constraints:
  - type: handoff
    pattern: speckit.implement
    reason: "This agent cannot implement code"
---

# Role

My agent specializes in X. It can do A, B, C.

# Hard Constraints

- Cannot call `speckit.implement`
- Cannot modify production code
- Cannot delete files

# Capabilities

- Research design patterns
- Review architecture
- Suggest improvements

## $ARGUMENTS

Describe what the agent should do.
\`\`\`

## Step 2: Register

```bash
opencode agent validate my-agent
opencode agent list  # Verify it appears
```

## Step 3: Use

```bash
# Switch via UI
opencode  # Press Shift+Tab, select my-agent

# Or delegate to it
@my-agent please analyze this architecture
```
```

### 1.5 Agent Context Update

**Task**: Run `.specify/scripts/bash/update-agent-context.sh copilot` (or equivalent)
- Adds custom agents infrastructure to agent-specific context files
- Preserves manual additions between markers
- Documents agent discovery, registry, and constraint enforcement patterns

---

## Phase 2: Implementation Breakdown

### MVP (Must Have - P1 & P0)

#### Sprint 1: Core Discovery & Registry
- **FR-001**: Agent definition format in .md files ✓
- **FR-002**: YAML frontmatter parsing ✓
- **FR-008**: Scan .opencode/agents/ directory
- **FR-009**: Scan ~/.opencode/agents/ directory
- **FR-010**: Repo-level precedence over global
- **FR-011**: `opencode agent list` command
- **FR-012**: In-memory registry
- **FR-013**: Handle missing directories gracefully
- **FR-014**: Unique agent identifiers

**Tasks**:
- Implement AgentDiscovery class with scan() and validate()
- Implement AgentRegistry data structure
- Implement CLI command: `opencode agent list`
- Implement CLI command: `opencode agent info <name>`
- Write unit tests for discovery and registry

---

#### Sprint 2: UI Agent Switching
- **FR-015**: Shift+Tab agent switcher
- **FR-016**: Display agent name, description, current indicator
- **FR-017**: Select and switch agents
- **FR-018**: Mark agents as primary vs. subagent
- **FR-019**: Preserve session context during switch
- **FR-020**: Remember last-selected agent

**Tasks**:
- Design TUI agent switcher component
- Implement agent switching logic
- Implement session context preservation
- Implement last-selected agent persistence (session.json)
- Write integration tests with TUI

---

#### Sprint 3: Agent Handoffs & @Mention
- **FR-027**: Declare handoff targets in frontmatter
- **FR-028**: Handoff configuration with labels and context flags
- **FR-029**: @mention syntax and programmatic handoffs
- **FR-030**: Context passing with send:true
- **FR-031**: Clean session with send:false

**Tasks**:
- Implement @mention parser for agent input
- Implement handoff execution engine
- Implement context serialization/deserialization
- Implement speckit integration for handoff payloads
- Write handoff scenario tests

---

#### Sprint 4: Constraint Enforcement
- **FR-021**: Hard Constraints section in definition
- **FR-022**: Runtime constraint validation
- **FR-023**: Prohibited handoff targets
- **FR-024**: Intercept violations and prevent execution
- **FR-025**: Log violations for audit
- **FR-026**: Apply constraints to all actions

**Tasks**:
- Design constraint middleware/interceptor
- Implement ConstraintValidator class
- Hook into OpenCode command interpreter
- Implement violation logging and reporting
- Write constraint enforcement tests (all three types)

---

#### Sprint 5: Validation & CLI Tools
- **FR-039**: Validate agent definitions at discovery
- **FR-040**: `opencode agent validate [name]` command
- **FR-041**: Validation of frontmatter, required fields, handoff targets, constraints
- **FR-042**: Detect circular handoff chains
- **FR-043**: Testing utilities for developers

**Tasks**:
- Implement AgentValidator class
- Implement CLI command: `opencode agent validate [name]`
- Implement CLI command: `opencode agent validate-all`
- Implement circular chain detector
- Create test helper library for agent developers

---

#### Sprint 6: Documentation & Templates
- **FR-044**: Agent template (.opencode/agent-template.md)
- **FR-045**: Template with examples
- **FR-046**: Guidelines for naming, design, constraints, handoffs
- **FR-047**: Creating Custom Agents guide
- **FR-048**: Reference implementations (specwriter, buildpro, planpro examples)

**Tasks**:
- Create agent-template.md with comprehensive examples
- Write AGENTS-HOWTO.md developer guide
- Create reference agents with inline documentation
- Create constraint design patterns guide
- Create handoff composition patterns guide

---

### Phase 2 (P2 - Deferred)

#### Multi-Location Agent Override
- **FR-009**: Global agent discovery (defer to next release)
- **Reason**: MVP focuses on repo-level agents; global agents can be added later

#### Advanced Features (Future)
- Versioned agents (FR-022 extension)
- Sandboxed agent execution
- Agent performance metrics
- Agent dependency management
- Dynamic agent loading/hot-reload

---

## Integration Points

### OpenCode CLI
- Agent discovery at startup
- Command registration: `agent list`, `agent info`, `agent validate`
- TUI Shift+Tab hook for agent switcher
- Session state management for last-selected agent
- Command interceptor for constraint enforcement

### Speckit Workflow
- Handoff to `speckit.specify`, `speckit.plan`, `speckit.implement`, `speckit.clarify`
- Context payload: spec.md, plan.md, tasks.md file paths
- Handoff URL: `@speckit.plan` or `@speckit_plan`

### AGENTS.md
- Project-level agent configuration
- Team standards injected at session start
- Default agent selection

---

## Testing Strategy

### Unit Tests
1. Agent definition parsing and validation
2. Agent registry operations (add, get, validate)
3. Handoff targeting and validation
4. Constraint matching and enforcement
5. @mention parsing
6. Circular chain detection
7. Session persistence

### Integration Tests
1. End-to-end agent discovery (repo + global)
2. Agent switching via TUI with context preservation
3. Handoff invocation with context passing
4. Constraint violation and blocking
5. Multi-agent workflows (A → B → C)
6. Speckit integration (custom agent → speckit agent)

### Acceptance Tests (User Stories)
1. FR-US1: Agent developer creates and registers custom agent
2. FR-US2: User switches between primary agents via UI
3. FR-US3: Agent delegates to subagent via @mention
4. FR-US4: Agent enforces capability constraints
5. FR-US5: Multi-location agent discovery with priority

---

## Success Metrics

From spec Success Criteria:
- SC-001: Create agent in < 15 minutes (template + docs)
- SC-002: Agent discovery in < 2 seconds (50 agents)
- SC-003: Agent switch via UI in < 5 seconds
- SC-004: Handoffs work 100% when preconditions met
- SC-005: Constraint violations caught 100%
- SC-006: Malformed agents detected at discovery
- SC-007: Developer can create agent with zero external help
- SC-008: Context preserved 95% of handoffs
- SC-009: Multi-location priority 100% correct

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|-----------|
| OpenCode TUI integration points not well-documented | Blocks UI work | Medium | Research OpenCode codebase early (Task 0.1) |
| Speckit integration protocol incompatible | Blocks handoff feature | Medium | Engage speckit team early (Task 0.3) |
| Constraint enforcement middleware complex to implement | Schedule slip | Medium | Prototype middleware early (Sprint 4) |
| Session context preservation requires refactoring | Blocks context switch | Low | Use existing OpenCode session mechanisms (Task 0.1) |
| Agent definitions difficult to write correctly | High error rate | Medium | Excellent template + validation tools (Sprint 6) |
| Circular handoff chains hard to detect | Infinite loops | Low | Topological sort early in discovery (Sprint 5) |

---

## Branch & Artifacts

**Branch**: `001-agents-infra`

**Key Artifacts**:
- `.specify/003-agents-infra/research.md` - Research findings
- `.specify/003-agents-infra/data-model.md` - Entity definitions
- `.specify/003-agents-infra/contracts/` - Interface contracts
- `.specify/003-agents-infra/design-decisions.md` - Architecture
- `.specify/003-agents-infra/quickstart.md` - Developer guide
- `.opencode/agent-template.md` - Agent template
- `.opencode/agents/` - Custom agent definitions (generated during impl)
- `src/opencode/agents/` - Agent infrastructure code (registry, discovery, constraints)
- `src/opencode/cli/commands/agent.rs` or `agent.py` - CLI commands
- `src/opencode/tui/switcher.rs` or `switcher.py` - TUI agent switcher
- Tests in `tests/agents/` directory

---

## Next Steps

1. ✅ Generate this implementation plan
2. → Phase 0: Execute research tasks (resolve NEEDS CLARIFICATION)
3. → Phase 1: Create data-model.md, contracts/, design-decisions.md
4. → Phase 1: Update agent context
5. → Phase 2: Begin Sprint 1 implementation
