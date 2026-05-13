# Research Report: OpenCode Custom Agents Infrastructure

**Created**: 2024-12-20  
**Status**: Complete - All NEEDS CLARIFICATION items resolved

---

## Executive Summary

The research confirms that OpenCode and Specfarm have mature, production-ready infrastructure that supports the 003-agents-infra feature. Key findings:

1. ✅ **OpenCode TUI Integration**: Agent switcher is a new feature (no Shift+Tab binding exists yet), but follows proven dialog patterns (Ctrl+O model switcher, Ctrl+S session switcher). Safe to implement as a new dialog component.

2. ✅ **Constraint Enforcement**: OpenCode has a robust permission system (`Permission.ask()`, `Ruleset`, `DeniedError`) that is the correct foundation. Translate markdown constraints into OpenCode's native permission rules at agent-load time.

3. ✅ **Speckit Integration**: All speckit agents (specify, plan, clarify, tasks, checklist, analyze) are mature and discoverable. They expect natural-language `$ARGUMENTS` input only; file paths are derived automatically from git branch via `get_feature_paths()` script. Handoff format is YAML frontmatter with `send: true/false` flag.

4. ✅ **Session Context Preservation**: OpenCode sessions are SQLite-backed and fully persistent. Switching agents does NOT require a new session — same session.ID can process messages from multiple agents in sequence.

5. ⚠️ **System Prompt Swapping**: OpenCode's `agent.Update()` currently only swaps the LLM provider/model, NOT the system prompt. To support multiple custom agents with different prompts, we must either:
   - Add a new `SwitchAgent()` method to agent.Service, OR
   - Replace `app.CoderAgent` wholesale on switch (simpler, retains pub/sub)

---

## Research Findings by Topic

### 1. OpenCode TUI Architecture & Session Management

**Decision**: Agent switcher will follow the 5-step dialog pattern (key binding → dialog state → message → update handler → overlay render)

#### Findings

- **Agent Switcher Does NOT Exist**: Shift+Tab is not bound anywhere in `tui.go:27-81`
- **Dialog Pattern**: All interactive overlays (model switcher Ctrl+O, session switcher Ctrl+S, commands Ctrl+K) follow identical 5-step pattern:
  1. Add keybinding to `keyMap` struct
  2. Add `showXxxDialog` bool and `xxxDialog` field to `appModel`
  3. Handle key press in `Update()` → set `showXxxDialog = true`
  4. Handle `XxxSelectedMsg` in `Update()` → execute logic, render overlay
  5. Render overlay in `View()` using `lipgloss.PlaceOverlay()`

- **Session Preservation**: Sessions are **SQLite-backed** (`session.Session` struct with ID, Title, MessageCount, etc.). Conversation history is persistent and never lost. Switching agents does NOT require a new session; the same `session.ID` can process messages from multiple agents.

- **Current Single Agent**: `app.App.CoderAgent` is a public field of type `agent.Service`. Currently only one agent per app instance.

- **Agent Initialization**: `agent.NewAgent(agentName, sessions, messages, tools...)` is called at app startup. System prompt is baked at construction time via `GetAgentPrompt()` in `prompt/prompt.go:13-28`.

- **Model-Only Switching**: `agent.Update(agentName, modelID)` swaps the LLM provider/model but NOT the system prompt. This is a limitation of current design.

#### Integration Points (Verified Safe)

| Component | File | Mechanism | Status |
|---|---|---|---|
| Keybinding | `tui.go:27-36` | Add to `keyMap` struct | Ready |
| Key handler | `tui.go:453-572` | Add `case key.Matches(keys.Agents, ...)` | Ready |
| Dialog state | `tui.go:98-140` | Add `showAgentDialog`, `agentDialog` fields | Ready |
| Message handler | `tui.go:359-367` | Add `case AgentSelectedMsg:` | Ready |
| Overlay render | `tui.go:827-840` | Add agent dialog to overlay stack | Ready |
| Active agent replacement | `app.go:App.CoderAgent` | Public field, replaceable | Ready |
| System prompt injection | `prompt/prompt.go:13-28` | Add case in `GetAgentPrompt()` switch | Needs Design |

#### Limitation & Workaround

**Limitation**: `agent.Update()` doesn't swap system prompts, only models.

**Recommended Workaround**: Add new `agent.Service` method `SwitchAgent(agentDef CustomAgentDef) error` that creates a new agent with the custom system prompt. This mirrors how `NewAgent()` works. The pub/sub subscriptions from TUI will continue to work.

---

### 2. Constraint Enforcement Architecture

**Decision**: Translate markdown constraints into OpenCode's native `Permission.Ruleset` at agent discovery time. Leverage existing permission system for enforcement.

#### Findings

- **OpenCode Permission System**: Mature, production-ready permission layer with three-state action model:
  - `allow` — action always permitted
  - `deny` — action always blocked, throws `DeniedError` immediately
  - `ask` — action requires user confirmation (async dialog)

- **Core Permission Engine**: `Permission.ask(input)` is the universal gate:
  - Takes `permission` name (e.g. "edit", "bash", "task"), `pattern` (glob or string), `ruleset` (ordered array of rules)
  - Last matching rule wins
  - `deny` action throws `DeniedError` before any execution
  - Exception: if user approves an `ask` action, it's added to `approved` set for that session

- **Tool Filtering**: Before LLM call, `Permission.disabled()` filters tools with `deny` rules entirely. The LLM never sees them in its tool list (prevents hallucination).

- **Per-Agent Permission Merging**: Each agent has a `Permission` field containing default rules merged with:
  - User-level config from `opencode.json`
  - Agent-specific config (e.g., plan agent denies all edits except plan files)

- **Subagent Permission Inheritance**: When agent A delegates to agent B, B's permissions are derived from A's via `deriveSubagentSessionPermission()`. B inherits A's `deny` rules (cannot do less restricted things than parent).

- **Audit Logging**: Event bus publishes `PermissionEvent` for all constraint evaluations, accessible to logging layer.

#### Constraint Mapping

Map markdown constraint types to OpenCode permission rules:

```
Markdown Constraint         → OpenCode Rule
────────────────────────────────────────────
handoff: prohibited-agent   → permission: "handoff"
                              pattern: "prohibited-agent"
                              action: "deny"

file_pattern: "*.js"        → permission: "edit"
                              pattern: "*.js"
                              action: "deny"

command: speckit.implement  → permission: "speckit.implement"
                              pattern: "*"
                              action: "deny"
```

#### Implementation Strategy

1. **Discovery Phase**: When scanning `.opencode/agents/*.md` files, parse YAML frontmatter
2. **Validation Phase**: Convert `constraints[]` array into `Permission.Ruleset`
3. **Agent Load Phase**: Call `agent.Service.SwitchAgent()` with converted rules embedded in system prompt or agent config
4. **Enforcement Phase**: All actions pass through existing `Permission.ask()` gate — no new enforcement needed
5. **Audit Phase**: Log constraint violations via existing event bus (`pubsub.Broker`)

#### Risk Mitigation

- **Risk**: Constraint syntax parsing is complex
  - **Mitigation**: Start with simple glob patterns (file_pattern) and exact matches (handoff, command); regex support deferred to Phase 2

- **Risk**: Subagent inheritance may be too restrictive
  - **Mitigation**: Document inheritance rules clearly; provide escape hatch (e.g., `inherit: false` in handoff config)

---

### 3. Speckit Integration Protocol

**Decision**: Custom agents invoke speckit agents via YAML frontmatter `handoffs` section. File context is auto-derived from git branch; handoff payloads are natural-language only.

#### Findings

**Source of Truth**: Speckit agents live in `graphedge/specfarm2:.github/agents/` as markdown files:
- `speckit.specify.agent.md` — create spec.md from description
- `speckit.clarify.agent.md` — Q&A to resolve ambiguities
- `speckit.plan.agent.md` — generate plan.md + research.md
- `speckit.tasks.agent.md` — break plan into tasks.md
- `speckit.checklist.agent.md` — generate quality checklists
- `speckit.analyze.agent.md` — consistency check across artifacts
- `speckit.constitution.agent.md` — update constitution.md
- `speckit.taskstoissues.agent.md` — convert tasks → GitHub issues
- `specfarm.reviewer4speckit.agent.md` — spec/plan quality review

**Invocation Format**: All speckit agents accept free-text natural language via `$ARGUMENTS` only. There are NO explicit file path arguments passed by caller.

```yaml
# Example: speckit.specify agent
---
description: Create or update the feature specification from a natural language feature description.
model: claude-haiku-4.5
handoffs:
  - label: Build Technical Plan
    agent: speckit.plan
    prompt: Create an implementation plan for this spec. I am building with...
---
# User Input
```text
$ARGUMENTS
```
```

**File Path Derivation**: Speckit agents derive all file paths automatically from git branch name at startup:

```bash
# Called by every speckit agent at startup:
.specify/scripts/bash/common.sh:get_feature_paths()

# Returns environment variables:
FEATURE_DIR='specs/003-agents-infra/'
FEATURE_SPEC='specs/003-agents-infra/spec.md'
IMPL_PLAN='specs/003-agents-infra/plan.md'
TASKS='specs/003-agents-infra/tasks.md'
RESEARCH='specs/003-agents-infra/research.md'
DATA_MODEL='specs/003-agents-infra/data-model.md'
QUICKSTART='specs/003-agents-infra/quickstart.md'
CONTRACTS_DIR='specs/003-agents-infra/contracts'
```

**Branch Convention**: Git branch name encodes feature number (e.g. `001-agents-infra` → `specs/001-agents-infra/`). Can be overridden via `SPECIFY_FEATURE` env var.

**Context Passing**: When `send: true` in handoff config, the current conversation history is forwarded to the target agent. File paths are ALWAYS auto-derived on the target side (not passed as arguments).

```yaml
# Example handoff with context:
handoffs:
  - label: Clarify Spec
    agent: speckit.clarify
    prompt: Clarify the current feature spec
    send: true  # ← passes conversation history; target will derive paths from branch
```

#### Handoff Patterns

**Pattern A: Direct slash command** (user-triggered)
```
/speckit.specify Add OAuth2 authentication to the API
/speckit.clarify
/speckit.plan I am building with Node.js and PostgreSQL
```

**Pattern B: Agent handoff button** (frontmatter-driven)
```yaml
# In custom agent frontmatter:
handoffs:
  - label: Write Feature Spec
    agent: speckit.specify
    prompt: Create a feature spec for...
# OpenCode renders as clickable button; click sends prompt to speckit.specify
```

**Pattern C: Inline workflow** (agent invokes shell script directly)
```bash
# Custom agent can invoke speckit shell scripts directly:
.specify/scripts/bash/setup-plan.sh --json "$ARGUMENTS" --number 5

# Returns JSON:
{"FEATURE_SPEC":"specs/005-feature/spec.md","IMPL_PLAN":"specs/005-feature/plan.md",...}
```

**Pattern D: AI-CONTEXT handoff comment** (cross-agent state)
All agents append inline `// AI-CONTEXT:` comments as text-based state handoff:
```
// AI-CONTEXT: Implemented token refresh per specs/auth-refresh-plan.md;
//   tasks: auth-refresh-impl, auth-test-edge-cases done;
//   timeout handling added per review feedback.
```

#### Reference Implementations

**specwriter agent** (local reference): `.opencode/agents/specwriter.md` in `genl-opncoders-infra`
- Delegates to `speckit.specify` for spec writing
- Delegates to `speckit.clarify` with `send: true` for context-aware clarification
- Delegates to `speckit.plan` for planning
- Documents "OR" pattern: if handoff isn't triggered, can run speckit scripts directly

---

### 4. @Mention Parsing Specification

**Decision**: @mention parsing happens in the agent code (not CLI parser). Syntax: `@agent_name [optional task description]`

#### Canonical Syntax

```
@agent_name task description
@specwriter create a domain-specific agent for build optimization
@buildpro analyze the current build system for bottlenecks
@speckit.specify add multi-tenant support to the SaaS platform
```

#### Parsing Rules

1. **Trigger**: `@` at the start of a word boundary
2. **Agent Name**: Word characters, hyphens, dots allowed: `[a-zA-Z0-9._-]+`
3. **Delimiters**: Space or newline after agent name
4. **Task Description**: Everything after agent name (optional)
5. **Markdown Distinction**: Within markdown code fences (```), @mention is literal text, not a trigger

#### Implementation

- **Where**: In the agent code or OpenCode message preprocessor, NOT the CLI parser
- **Pattern**: Regex: `@([a-zA-Z0-9._-]+)\s+(.+)?` with lookahead for word boundary
- **Fallback**: If no agent found, treat as literal text (no error)
- **Multiple Agents**: Single message can mention multiple agents (parsed into queue of handoffs)

#### Examples

```
@specwriter please create a spec for rate limiting
→ Parsed as: agent="specwriter", task="please create a spec for rate limiting"

I'll use @buildpro to check build times and @planpro to optimize.
→ Two handoffs: buildpro, planpro

Use @mention in code to reference email: @person@example.com
→ Literal text (not a handoff, requires agent exists check)
```

---

### 5. Agent Last-Used Persistence

**Decision**: Store last-selected agent in `~/.opencode/session.json` (global), scoped by repo directory (not per-branch).

#### Storage Design

```json
{
  "sessions": {
    "/path/to/repo1": {
      "last_agent": "specwriter",
      "timestamp": 1703084400
    },
    "/path/to/repo2": {
      "last_agent": "buildpro",
      "timestamp": 1703084500
    }
  }
}
```

#### Algorithm

```
On Session Startup:
1. Get repo_path from git rev-parse --show-toplevel
2. Look up repo_path in ~/.opencode/session.json
3. If found and agent still exists in registry: restore it
4. If not found or agent deleted: use default agent from config

On Agent Switch:
1. Write new agent name to ~/.opencode/session.json[repo_path]
2. Write current timestamp
3. Persist to disk synchronously
```

#### Scoping

- **Global Storage**: `~/.opencode/session.json` (shared across all repos)
- **Key**: Repo directory absolute path (e.g. `/home/user/projects/myapp`)
- **Not Branch-Scoped**: Same agent restored regardless of branch (user preference, not spec requirement)
- **Not Session-Scoped**: Agent persists across all sessions in a repo directory

#### Edge Cases

| Case | Behavior |
|---|---|
| Agent deleted between sessions | Fallback to default agent, log warning |
| Repo moved/renamed | New repo path treated as new entry; old path persists in JSON (cleanup on shutdown) |
| Concurrent sessions in same repo | Last agent to switch wins (last-write-wins); acceptable conflict |
| Permission denied on `~/.opencode/` | Continue without persisting; no error |

---

### 6. Best Practices for Custom Agent Design

**Decision**: Document constraints as "what agent CANNOT do", not "what it CAN do". Constrain early and often.

#### Principles

1. **Non-Overlapping Responsibilities**: Each agent should own a specific domain
   - ✅ `specwriter` — spec and requirements, NOT implementation
   - ✅ `buildpro` — build system tuning, NOT application code
   - ✅ `planpro` — planning and task breakdown, NOT code writing

2. **Constraint-First Design**: Define hard constraints before capabilities
   - Document: "I cannot write code, modify production files, call speckit.implement"
   - Reason: Prevents accidental scope creep and out-of-domain actions

3. **Handoff as Composition**: Use handoffs to delegate rather than attempt out-of-scope work
   - ✅ Good: specwriter hands off to speckit.specify for spec creation
   - ❌ Bad: specwriter attempts to write code inline

4. **Single Responsibility**: If agent does multiple things, consider splitting
   - ✅ `specwriter` — handles spec, plan, docs (all in writing domain)
   - ❌ `everything-agent` — builds, deploys, tests, writes code (too broad)

5. **Explicit Over Implicit**: Name agents clearly for their domain
   - ✅ `buildpro`, `specwriter`, `testmaster` — immediately clear
   - ❌ `worker1`, `agent2`, `processor` — ambiguous purpose

#### Anti-Patterns

1. **Constraint Holes**: Specifying constraints that leave loopholes
   - ❌ Bad: Deny `file: *.py` but allow `file: *.pyx` → same language, loophole
   - ✅ Good: Deny `file: **` except for `file: docs/**`

2. **Missing Handoff Targets**: Agent needs capability it can't perform
   - ❌ Bad: buildpro needs to update spec but has no handoff to specwriter
   - ✅ Good: buildpro has `speckit.plan` in handoffs for plan updates

3. **Circular Handoffs**: A → B → A creates infinite loops
   - System should detect at validation time and warn

4. **Vague Descriptions**: Description doesn't match capabilities
   - ❌ Bad: "I help with everything"
   - ✅ Good: "I specialize in system design and architecture"

#### Reference Implementations (Local Repo)

From `genl-opncoders-infra:.opencode/agents/`:

- **specwriter.md**: Specializes in writing specs, plans, docs. Hands off to speckit.specify, speckit.clarify, speckit.plan. Constraints: Cannot implement code, cannot call speckit.implement.

- **buildpro.md** (planned): Specializes in build systems. Constraints: Cannot modify source code, only build configs.

- **planpro.md** (planned): Specializes in planning and task breakdown. Constraints: Cannot write specs (that's specwriter), cannot implement code.

---

## Resolved NEEDS CLARIFICATION Items

| Item | Resolution |
|---|---|
| **Session Context Preservation** | Sessions are SQLite-backed and persistent. Switching agents does NOT require new session. Same session.ID can process messages from multiple agents. |
| **OpenCode TUI Integration Points** | Agent switcher follows proven dialog pattern (5 steps: keybinding → state → message → update handler → render). Safe integration with `tui.go` and `dialog/` components. |
| **Constraint Enforcement Interception** | Translate markdown constraints into OpenCode's native `Permission.Ruleset`. Leverage existing `Permission.ask()` gate for enforcement. No new interception layer needed. |
| **Speckit Integration Protocol** | All speckit agents expect natural-language `$ARGUMENTS` only. File paths auto-derived from git branch via `get_feature_paths()`. Handoff format: YAML frontmatter with `send: true/false`. |
| **@Mention Parsing Implementation** | Happens in agent code with regex: `@([a-zA-Z0-9._-]+)\s+(.+)?`. Optional task description. Multiple agents per message supported. |
| **Agent Last-Used Persistence** | Stored in `~/.opencode/session.json`, scoped by repo directory (not per-branch). Global storage, keyed by absolute path. |

---

## Dependencies on External Teams

1. **OpenCode Team**: Need confirmation that new `SwitchAgent()` method can be added to `agent.Service` interface. Backcompat: existing `Update()` continues to work for model-only switches.

2. **Speckit Team**: No changes needed on their side. Speckit agents are mature and discoverable. Custom agents will call them via existing patterns.

3. **No Blocking Dependencies**: All research findings indicate features can be implemented without external blockers. OpenCode and Speckit have mature infrastructure.

---

## Alternatives Considered

### Constraint Enforcement: Custom Middleware vs. Native Permission System

| Approach | Pros | Cons | Decision |
|---|---|---|---|
| **Native Permission System** (chosen) | Mature, audited, used production-wide, event bus for logging | Requires translation layer from markdown | USE |
| **Custom Middleware** | Direct control, custom logic | Duplicate permission logic, harder to audit, no event bus | REJECT |

### Agent Switching: Replace Agent vs. New SwitchAgent Method

| Approach | Pros | Cons | Decision |
|---|---|---|---|
| **New `SwitchAgent()` Method** (chosen) | Explicit, doesn't break existing `Update()`, cleaner API | Requires OpenCode change | USE |
| **Replace `app.CoderAgent` wholesale** | No API change | Loses pub/sub subscriptions, requires re-subscribe in TUI | FALLBACK |

### @Mention Parsing: CLI Parser vs. Agent Code

| Approach | Pros | Cons | Decision |
|---|---|---|---|
| **Agent Code** (chosen) | Agent controls its own handoffs, flexible @mention semantics | Duplicated parsing logic in each agent | USE |
| **CLI Parser** | Single parser, centralized | Constrains @mention syntax globally | REJECT |

### Session Scoping: Per-Repo vs. Per-Branch

| Approach | Pros | Cons | Decision |
|---|---|---|---|
| **Per-Repo** (chosen) | Simpler, matches typical workflow | May not match branch-specific agent preferences | USE |
| **Per-Branch** | Fine-grained control | Complex, repo moves → orphaned entries | REJECT |

---

## Updated Technical Context

### Now Known (Resolved)

✅ Session context preservation mechanism  
✅ OpenCode TUI integration hooks  
✅ Constraint enforcement architecture  
✅ Speckit handoff protocol  
✅ @Mention parsing strategy  
✅ Agent persistence mechanism  
✅ Best practices and anti-patterns  

### Still to Design (Phase 1)

🔲 Data model entities (Agent, Handoff, Constraint, AgentRegistry)  
🔲 Interface contracts (CLI commands, TUI components, handoff payloads)  
🔲 Permission rule translation algorithm  
🔲 Circular handoff detection algorithm  
🔲 Validation and error messaging  

---

## Risk Register (Updated)

| Risk | Impact | Probability | Status |
|---|---|---|---|
| OpenCode `SwitchAgent()` not feasible | Blocks multi-agent support | Medium | **MITIGATED** — Fallback: replace agent wholesale |
| Speckit agent APIs incompatible | Blocks handoff feature | Low | **RESOLVED** — APIs verified compatible |
| Session persistence breaks on agent switch | Loses conversation history | Low | **RESOLVED** — SQLite-backed sessions are persistent |
| Constraint enforcement too permissive | Security leak | Low | **MITIGATED** — Leverage mature OpenCode permission system |
| @Mention parsing collides with markdown | Conflicts in code snippets | Medium | **DESIGN** — Add markdown fence awareness in Phase 1 |
| Agent deletion breaks last-used persistence | User confusion | Low | **RESOLVED** — Fallback to default agent on missing agent |

---

## Recommendations for Implementation

### Phase 0 Completion ✅

All NEEDS CLARIFICATION items resolved. Safe to proceed to Phase 1 Design.

### Phase 1 Priority

1. **High Priority**: Data model definition (Agent, Handoff, Constraint) — blocks all downstream work
2. **High Priority**: Agent discovery algorithm — must work before any testing
3. **Medium Priority**: Interface contracts — enables parallel implementation
4. **Medium Priority**: Permission rule translation — unlocks constraint enforcement

### Phase 2 Sequencing

1. Sprint 1: Agent Discovery & Registry (blocking)
2. Sprint 2: UI Agent Switching (depends on Sprint 1)
3. Sprint 3: Agent Handoffs (depends on Sprint 1, 2)
4. Sprint 4: Constraint Enforcement (depends on Sprint 1)
5. Sprint 5: Validation & CLI Tools (depends on Sprint 1)
6. Sprint 6: Documentation & Templates (unblocked, run in parallel with Sprint 4-5)

---

## Conclusion

The OpenCode and Speckit ecosystems have mature infrastructure for all required features. The 003-agents-infra feature is feasible with moderate effort and manageable risks. No external blockers identified. Ready to proceed to Phase 1 design.

**Research Status**: ✅ COMPLETE  
**Gate 1 (Research Phase Completion)**: ✅ PASSED  
**Recommended Action**: Proceed to Phase 1 (Design & Contracts)
