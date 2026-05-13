# Feature Specification: OpenCode Custom Agents Infrastructure

**Feature Branch**: `003-agents-infra`  
**Created**: 2024-12-20  
**Status**: Draft  
**Input**: User description: "Design a comprehensive custom agents infrastructure for OpenCode that enables teams to create domain-specific agents, provides standardized registration and discovery, supports agent handoffs and capability constraints, integrates with speckit workflow, allows repo-level or global agent location, documents best practices, and supports both primary and subagents."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Agent Developer Creates Custom Domain Agent (Priority: P1)

A team member (Agent Developer) wants to create a custom agent specialized for their domain (e.g., buildpro for advanced builds, specwriter for spec orchestration). They need a clear, standardized way to define the agent, register it with the OpenCode system, and make it discoverable and usable by the team.

**Why this priority**: This is the core capability that enables the entire infrastructure. Without a standardized way to create and register agents, the system cannot function.

**Independent Test**: Can be fully tested by: creating a new agent following the standardized format, verifying it's discoverable via `opencode agent list`, and confirming it can be invoked.

**Acceptance Scenarios**:

1. **Given** an agent developer follows the standardized agent definition format, **When** they place the agent in `.opencode/agents/` or `~/.opencode/agents/`, **Then** the agent is automatically registered and discoverable via `opencode agent list`
2. **Given** an agent is properly defined with frontmatter metadata, **When** OpenCode scans for agents, **Then** the agent metadata is correctly parsed and available for system use
3. **Given** a developer creates an agent, **When** they check `opencode agent list`, **Then** they see the agent name, description, model, and capabilities listed

---

### User Story 2 - User Switches Between Primary Agents via UI (Priority: P1)

An end user wants to switch between different primary agents (e.g., from the default agent to specwriter to buildpro) using the OpenCode UI (Shift+Tab switcher). They need quick, intuitive access to all available domain-specific agents without manual configuration.

**Why this priority**: User-facing agent selection is critical for the feature to be usable in daily workflows. Without this capability, users cannot leverage the specialized agents created by the team.

**Independent Test**: Can be fully tested by: launching OpenCode UI, using Shift+Tab to open the agent switcher, verifying all registered primary agents appear, and successfully switching to each agent.

**Acceptance Scenarios**:

1. **Given** multiple primary agents are registered (specwriter, buildpro, planpro, default agent), **When** user presses Shift+Tab, **Then** the agent switcher shows all available primary agents with descriptions
2. **Given** user selects a different agent from the switcher, **When** selection is confirmed, **Then** the active agent changes and the session continues with the new agent
3. **Given** an agent is marked as `global: false` in repo context, **When** user views the agent switcher in a different repository, **Then** that agent is not listed

---

### User Story 3 - Agent Delegates to Subagent via Mention (Priority: P2)

During task execution, an agent needs to delegate work to another specialized agent (e.g., buildpro needs to call specfarm.reviewer4speckit). The agent developer configures handoff rules that define which agents can delegate to which other agents, and users (or agents) invoke subagents using @mention syntax or programmatic handoffs.

**Why this priority**: Handoffs enable complex workflows and agent composition. While not required for MVP, they're essential for building sophisticated automation that combines multiple specialized agents.

**Independent Test**: Can be fully tested by: configuring handoff rules between agents, invoking a subagent via @mention or programmatic call, and verifying the subagent receives the correct context and can execute the delegated task.

**Acceptance Scenarios**:

1. **Given** an agent has a `handoffs` configuration defining allowed delegates, **When** the agent is invoked with `@subagent_name [task]`, **Then** the subagent is loaded with the appropriate context and executes the delegated task
2. **Given** an agent attempts to delegate to a subagent not in its allowed `handoffs`, **When** the delegation is attempted, **Then** the system prevents the handoff and raises an error
3. **Given** a handoff is configured with `send: true`, **When** the agent delegates, **Then** the current context (spec, plan, tasks) is automatically passed to the subagent

---

### User Story 4 - Agent Enforces Capability Constraints (Priority: P1)

An agent has hard constraints that define what it CANNOT do (e.g., specwriter cannot call speckit.implement or write code). The OpenCode system enforces these constraints and prevents the agent from violating its defined boundaries, ensuring agents stay within their intended scope and responsibilities.

**Why this priority**: Constraint enforcement is critical for system safety and reliability. Preventing agents from straying outside their intended scope prevents errors, maintains clean separation of concerns, and ensures predictable behavior.

**Independent Test**: Can be fully tested by: attempting to perform a restricted action from within a constrained agent (e.g., specwriter trying to call speckit.implement), and verifying the system blocks the action and returns a meaningful error message.

**Acceptance Scenarios**:

1. **Given** an agent has defined `hard_constraints` listing prohibited actions, **When** the agent attempts a prohibited action, **Then** the system intercepts the call and raises an error before execution
2. **Given** specwriter agent has `speckit.implement` in its prohibited handoffs, **When** specwriter is invoked with a code implementation request, **Then** specwriter responds that it cannot implement code and suggests appropriate agents
3. **Given** constraints specify prohibited file patterns (e.g., `.js`, `.py`), **When** the agent attempts to create or modify files matching those patterns, **Then** the system prevents the operation

---

### User Story 5 - Agent Developer Configures Multi-Location Agent Discovery (Priority: P2)

An agent developer can place custom agents in either repo-level (`.opencode/agents/`) or global user-level (`~/.opencode/agents/`) locations. The system discovers agents from both locations, prioritizes repo-level agents over global ones with the same name, and allows teams to share global agents while enabling repo-specific customization.

**Why this priority**: Multi-location support enables both team standardization (global agents) and project customization (repo-level overrides). While important for flexibility, MVP can start with repo-level only and add global support later.

**Independent Test**: Can be fully tested by: creating agents in both repo-level and global locations, running `opencode agent list`, and verifying correct priority and discovery behavior.

**Acceptance Scenarios**:

1. **Given** an agent exists in both repo-level and global locations with the same name, **When** agents are discovered, **Then** the repo-level version is used and global version is ignored
2. **Given** an agent exists only in the global location, **When** no repo-level override exists, **Then** the global agent is discovered and available
3. **Given** a repo-level agent is removed, **When** agents are re-discovered, **Then** the global version (if it exists) becomes available

---

### Edge Cases

- What happens when an agent references a handoff target that doesn't exist?
- How does the system handle circular handoff chains (Agent A → B → C → A)?
- What happens if an agent definition is malformed or missing required fields?
- How does constraint enforcement work if constraints are inherited from base/shared agent definitions?
- What if two agents have the same name in the same location?
- How does agent discovery behave when `.opencode/agents/` directory doesn't exist?
- What happens when `~/.opencode/agents/` is not accessible due to permissions?

## Requirements *(mandatory)*

### Agent Definition Format

- **FR-001**: Each agent MUST be defined in a `.md` file located in `.opencode/agents/` (repo-level) or `~/.opencode/agents/` (global)
- **FR-002**: Agent definition file MUST begin with YAML frontmatter (delimited by `---`) containing metadata including: `description`, `model`, `handoffs` (array of handoff targets), `constraints` (prohibited actions), and optional `global` flag
- **FR-003**: Agent definition file MUST contain a "Role" section describing the agent's primary purpose and capabilities
- **FR-004**: Agent definition file MUST contain "Hard Constraints" section documenting what the agent CANNOT do and prohibited handoff targets
- **FR-005**: Agent definition file MUST specify all handoff targets the agent can delegate to, including the target agent name and a brief description of the delegation purpose
- **FR-006**: Agent definition MUST include a `$ARGUMENTS` placeholder section to capture user input for the agent
- **FR-007**: System MUST parse and validate agent frontmatter on discovery, failing gracefully on malformed metadata

### Agent Discovery & Registration

- **FR-008**: OpenCode MUST automatically scan `.opencode/agents/` directory at startup for custom agents
- **FR-009**: OpenCode MUST automatically scan `~/.opencode/agents/` directory at startup for global agents
- **FR-010**: When an agent exists in both locations, repo-level MUST take precedence over global
- **FR-011**: Agents MUST be discoverable via `opencode agent list` command, displaying agent name, description, and model
- **FR-012**: System MUST catalog discovered agents in an in-memory registry accessible to the OpenCode command interpreter
- **FR-013**: Agent discovery MUST handle missing directories gracefully (e.g., if `.opencode/agents/` doesn't exist, continue without error)
- **FR-014**: System MUST assign each agent a unique identifier for programmatic reference in handoffs and constraints

### Agent Switching & Selection

- **FR-015**: OpenCode UI MUST provide an agent switcher (Shift+Tab) that lists all available primary agents
- **FR-016**: Agent switcher MUST display agent name, brief description, and indicator of current/active agent
- **FR-017**: User MUST be able to select any primary agent from the switcher and switch to it in a single action
- **FR-018**: System MUST support marking agents as primary (switchable in UI) vs. subagent (callable only via @mention or handoff)
- **FR-019**: Switching agents MUST preserve session context (conversation history, current task, file state) when applicable
- **FR-020**: System MUST remember the user's last selected agent and restore it on next OpenCode session in the same repository

### Capability Constraints & Enforcement

- **FR-021**: Each agent definition MUST include a "Hard Constraints" section listing actions the agent cannot perform
- **FR-022**: System MUST enforce agent constraints by validating actions against the agent's constraint list before execution
- **FR-023**: Prohibited handoff targets MUST be specified in the agent definition and enforced during delegation attempts
- **FR-024**: When an agent violates a constraint, system MUST intercept the action, prevent execution, and return a clear error message
- **FR-025**: Constraint violations MUST be logged for auditability and debugging
- **FR-026**: Constraint enforcement MUST apply to all handoff attempts, file operations, and command executions

### Agent Handoffs & Delegation

- **FR-027**: Each agent MUST declare all allowed handoff targets in a `handoffs` array in the frontmatter
- **FR-028**: Handoff configuration MUST include: target agent name, human-readable label, brief prompt/description, and optional `send: true` flag to pass context
- **FR-029**: System MUST support two handoff invocation patterns: (a) @mention syntax in chat (e.g., `@buildpro [task]`), and (b) programmatic handoff from agent code
- **FR-030**: When `send: true` is specified, the current context (spec.md, plan.md, tasks.md, conversation state) MUST be automatically passed to the target agent
- **FR-031**: When `send: false` or unspecified, handoff creates a clean session in the target agent without prior context
- **FR-032**: System MUST prevent unauthorized handoffs (agent attempting to delegate to agent not in its allowed `handoffs` list)
- **FR-033**: Handoff execution MUST be auditable, with clear tracking of which agent handed off to which, and the task/context passed

### Integration with Speckit Workflow

- **FR-034**: OpenCode agents MUST integrate seamlessly with speckit workflow agents (speckit.specify, speckit.plan, speckit.implement, speckit.clarify)
- **FR-035**: Custom agents (specwriter, buildpro, planpro) MUST be able to hand off to speckit agents via handoff configuration
- **FR-036**: Speckit agents MUST remain discoverable and usable independently alongside custom agents
- **FR-037**: Agent handoff payloads MUST be compatible with speckit agents' expected input formats (spec file path, plan file path, task ID, etc.)
- **FR-038**: System MUST provide a clear handoff URL/syntax for invoking speckit agents from custom agents

### Testing & Validation

- **FR-039**: System MUST validate agent definitions at discovery time, detecting missing required fields and logging warnings
- **FR-040**: System MUST provide a validation command (e.g., `opencode agent validate [agent-name]`) to test agent definitions and constraint configurations
- **FR-041**: Validation MUST verify: (a) frontmatter syntax is valid YAML, (b) all required fields are present, (c) handoff targets reference existing agents, (d) constraints are properly formatted
- **FR-042**: System MUST detect and report circular handoff chains to prevent infinite loops
- **FR-043**: System MUST provide testing utilities for developers to verify agent handoffs work as expected

### Documentation & Best Practices

- **FR-044**: System MUST provide a standardized template for creating new agents (`.opencode/agent-template.md`)
- **FR-045**: Template MUST include examples for: basic agent definition, frontmatter structure, handoff configuration, and constraint definition
- **FR-046**: Documentation MUST provide guidelines for: (a) agent naming conventions, (b) designing non-overlapping agent responsibilities, (c) configuring safe constraints, (d) best practices for handoffs
- **FR-047**: System MUST include a "Creating Custom Agents" guide explaining the full workflow from template to registration to usage
- **FR-048**: Documentation MUST provide examples of existing agents (specwriter, buildpro, planpro) as reference implementations

### Key Entities

- **Agent**: A specialized autonomous or semi-autonomous entity capable of executing tasks, with defined capabilities (what it CAN do), constraints (what it CANNOT do), and handoff targets (which other agents it can delegate to). Identified by unique name.
  - Attributes: `name`, `description`, `model`, `role`, `capabilities`, `constraints`, `handoffs`, `location` (repo-level or global), `is_primary` (UI-switchable or not)
  - Relationships: references other agents via `handoffs`, can be overridden at repo-level

- **Handoff**: A configuration that enables one agent to delegate task execution to another agent. Defines the target agent, invocation trigger, and context passing behavior.
  - Attributes: `target_agent`, `label`, `prompt_template`, `send_context` (boolean), `requires_approval` (optional)
  - Relationships: belongs to source agent's `handoffs` array, targets must exist in agent registry

- **Constraint**: A restriction that defines prohibited actions for an agent. Types include: prohibited handoff targets, prohibited file operations, prohibited commands/actions.
  - Attributes: `type` (handoff, file_pattern, command), `pattern` (regex or string), `reason` (explanation)
  - Relationships: belongs to agent's `constraints` array, enforced at runtime

- **Agent Registry**: In-memory mapping of all discovered agents (repo-level and global) with agent metadata and constraint rules. Acts as the single source of truth for agent availability and capabilities.
  - Attributes: `agents` (map of agent_name → Agent), `repo_agents`, `global_agents`, `last_updated`
  - Relationships: populated during OpenCode startup, consulted for agent switching, handoff validation, and constraint enforcement

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Teams can create and register a new custom agent following the standardized format in under 15 minutes without external documentation reference
- **SC-002**: Agent discovery and registration completes in under 2 seconds when OpenCode starts, regardless of number of agents (up to 50 agents)
- **SC-003**: User can switch between primary agents via UI in under 5 seconds (from pressing Shift+Tab to agent loaded and ready to receive input)
- **SC-004**: Agent handoffs work reliably with 100% success rate when target agent exists and is in allowed handoff list
- **SC-005**: Constraint violations are caught and prevented 100% of the time; no constrained agent can execute prohibited actions
- **SC-006**: Agent definitions with errors are detected during discovery and reported with actionable error messages; no malformed agents are registered
- **SC-007**: Documentation enables a developer unfamiliar with the system to create their first custom agent with zero external help
- **SC-008**: 95% of handoffs complete with context preserved correctly when `send: true` is specified
- **SC-009**: Multi-location agent discovery (repo-level + global) prioritizes correctly 100% of the time; repo-level agents always take precedence

### Qualitative Outcomes

- **SC-010**: Teams report that creating domain-specific agents reduces cognitive load by enabling focused agent capabilities rather than single omniscient agent
- **SC-011**: Agent developers report that the constraint model prevents accidental out-of-scope actions and provides confidence in agent safety
- **SC-012**: Users report that agent switching via UI feels natural and intuitive, enabling quick adaptation to different task types

## Assumptions

- **Assumption 1 - Agent Developers are Trusted**: Agents are created and registered only by trusted team members (not arbitrary users); system does not require sandboxing or execution isolation for agent code
- **Assumption 2 - Agent Discovery Frequency**: Agent definitions are updated infrequently and system can use on-demand discovery or caching; hot-reloading of agent definitions is not required for MVP
- **Assumption 3 - Context Passing Format**: When agents hand off context via `send: true`, the target agent expects input in standard speckit format (spec.md, plan.md, tasks.md file paths in arguments); special formatting or transformation is not required
- **Assumption 4 - Session Preservation**: Session state (conversation history, current files) can be preserved across agent switches using standard mechanisms; no special cross-agent state synchronization is required
- **Assumption 5 - Repo vs. Global**: Most teams will use repo-level agents for project-specific customizations; global agents are secondary and used only for company-standard or team-shared agents
- **Assumption 6 - Constraint Syntax**: Constraints can be expressed using simple patterns (string prefixes, file glob patterns, exact matches); complex logic is not required for MVP
- **Assumption 7 - OpenCode UI Availability**: OpenCode provides UI components (agent switcher) and hooks for agent lifecycle events that custom agents can leverage; agents do not need to build their own UI infrastructure
- **Assumption 8 - Speckit Integration**: Existing speckit agents (speckit.specify, speckit.plan, etc.) have compatible input/output formats that work seamlessly with OpenCode's handoff mechanism
- **Assumption 9 - No Agent Versioning**: Individual agents do not require version management; the system manages a single version of each agent per location (repo-level or global)
- **Assumption 10 - Permission Defaults**: If `~/.opencode/agents/` is not readable or doesn't exist, the system continues without error; repo-level agents are not blocked by global permission issues
