---
description: "Task list for OpenCode Custom Agents Infrastructure feature"
---

# Tasks: OpenCode Custom Agents Infrastructure (003-agents-infra)

**Input**: Design documents from `/specs/003-agents-infra/`  
**Prerequisites**: spec.md (5 user stories), plan.md (implementation strategy), data-model.md (entity definitions), contracts/ (CLI & agent definition format), research.md (architecture decisions)

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and directory structure

- [ ] T001 Create OpenCode agent framework directory structure: `.opencode/agents/` for repo-level agents
- [ ] T002 [P] Create GitHub Actions workflow for agent discovery validation in `.github/workflows/`
- [ ] T003 [P] Setup agent logging and audit trail infrastructure for constraint violation tracking
- [ ] T004 Create test fixtures directory structure: `tests/fixtures/agents/` for mock agent definitions

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before user stories can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T005 Implement Agent Registry system (`agent/registry.go`) - in-memory registry with discovery, validation, lookup
- [ ] T006 [P] Implement Agent Definition Parser (`agent/parser.go`) - parse YAML frontmatter, validate fields, extract metadata
- [ ] T007 [P] Implement Constraint Validation Engine (`agent/constraints.go`) - parse constraint types (handoff, file_pattern, command), translate to OpenCode permissions
- [ ] T008 [P] Implement Handoff Configuration Parser (`agent/handoff.go`) - parse handoff targets, labels, context passing flags, circular detection
- [ ] T009 Implement Agent Discovery Mechanism (`agent/discovery.go`) - scan `.opencode/agents/` and `~/.opencode/agents/`, load into registry with precedence logic
- [ ] T010 [P] Implement Multi-Location Precedence Logic (`agent/discovery.go:precedence`) - repo-level agents override global, error handling for missing directories
- [ ] T011 [P] Create Permission Rule Translator (`agent/permissions.go`) - convert markdown constraints to OpenCode Permission.Ruleset
- [ ] T012 Implement Session-Agent State Management (`session/agent_state.go`) - track current agent per session, preserve context across switches, last-used agent persistence
- [ ] T013 [P] Create Agent Validation Command (`cmd/agent_validate.go`) - validate agent definitions, detect malformed files, check circular handoffs
- [ ] T014 [P] Setup Circular Handoff Detection (`agent/handoff.go:circular_check`) - topological sort to detect A→B→A patterns

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Agent Developer Creates Custom Domain Agent (Priority: P1) 🎯 MVP

**Goal**: Enable agent developers to create new custom agents using standardized format and have them automatically discovered and registered.

**Independent Test**: Create a new test agent following spec, verify it appears in `opencode agent list`, check metadata is correct, confirm it can be selected.

### Implementation for User Story 1

- [ ] T015 [P] [US1] Create Agent Definition Contract implementation in `contracts/agent_definition_contract.go` - validate against contract spec
- [ ] T016 [P] [US1] Implement frontmatter schema validation (`agent/parser.go:validate_schema`) - enforce required fields, type checking, semantic validation
- [ ] T017 [US1] Implement agent discovery hook in OpenCode initialization (`app/init.go:discovery_hook`) - load agents on startup, log errors for invalid agents
- [ ] T018 [P] [US1] Create example agent template `templates/agent-template.md` - starter template with inline comments for developers
- [ ] T019 [P] [US1] Add agent metadata caching (`agent/registry.go:cache`) - store parsed metadata for fast lookup, invalidate on file changes
- [ ] T020 [US1] Implement agent info command (`cmd/agent_info.go`) - display detailed agent information, role, capabilities, constraints, handoffs
- [ ] T021 [P] [US1] Create agent list command (`cmd/agent_list.go`) - output table/JSON with agent names, models, locations, primary vs subagent classification
- [ ] T022 [US1] Create test suite for agent definition parsing (`tests/agent_definition_test.go`) - test valid/invalid frontmatter, field validation, error messages
- [ ] T023 [P] [US1] Add agent documentation (`docs/creating-agents.md`) - guide for developers, example agents, common patterns

**Checkpoint**: Agent developers can create and register new custom agents; agents appear in discovery list

---

## Phase 4: User Story 2 - User Switches Between Primary Agents via UI (Priority: P1) 🎯 MVP

**Goal**: End users can quickly switch between primary agents using Shift+Tab UI switcher, with session context preserved.

**Independent Test**: Launch OpenCode, use Shift+Tab to switch agents, verify session continues with new agent, check last-used agent persists across restarts.

### Implementation for User Story 2

- [ ] T024 [P] [US2] Implement agent switcher dialog component (`tui/agent_dialog.go`) - List primary agents, handle selection, render overlay
- [ ] T025 [P] [US2] Add Shift+Tab keybinding (`tui/keymap.go:Agents`) - bind Shift+Tab to show agent switcher, follow existing dialog pattern (Ctrl+O model switcher)
- [ ] T026 [P] [US2] Implement agent.Service.SwitchAgent() method (`agent/service.go:SwitchAgent`) - load new agent with custom system prompt, preserve session.ID
- [ ] T027 [US2] Implement agent selection handler (`tui/handlers.go:AgentSelectedMsg`) - update app.CoderAgent, update system prompt, continue session
- [ ] T028 [P] [US2] Create last-used agent persistence (`session/agent_state.go:persist_last_used`) - store in session DB or ~/.opencode/last_agent.json
- [ ] T029 [US2] Implement last-used agent restoration (`app/init.go:restore_last_agent`) - on startup, switch to last-used agent if exists
- [ ] T030 [P] [US2] Add agent display in TUI header (`tui/header.go:current_agent`) - show current active agent name/icon
- [ ] T031 [P] [US2] Create agent switcher UI tests (`tests/tui_agent_switcher_test.go`) - test dialog rendering, selection handling, overlay behavior
- [ ] T032 [US2] Implement session context preservation logic (`session/context.go:preserve_on_agent_switch`) - verify conversation history, tools, permissions survive switch

**Checkpoint**: Users can switch between primary agents with UI, session context is preserved

---

## Phase 5: User Story 3 - Agent Delegates to Subagent via Mention (Priority: P2)

**Goal**: Agents can delegate work to subagents using @mention syntax or programmatic handoffs; context can be passed between agents.

**Independent Test**: Configure handoff from agent A to agent B, invoke `@agent_B [task]`, verify agent B receives context, executes task, returns result.

### Implementation for User Story 3

- [ ] T033 [P] [US3] Implement @mention parser (`parser/mention.go`) - extract `@agent_name` and optional `[task description]` from user input
- [ ] T034 [P] [US3] Implement handoff validation (`agent/handoff.go:validate`) - check target agent exists, source agent can delegate, enforce handoff permissions
- [ ] T035 [US3] Implement programmatic handoff (`agent/handoff.go:execute_handoff`) - invoke target agent with context, handle send_context flag, return result
- [ ] T036 [P] [US3] Implement context passing mechanism (`handoff/context.go`) - serialize spec.md, plan.md, tasks.md paths per send_context flag
- [ ] T037 [P] [US3] Create handoff configuration validation (`agent/handoff.go:schema_validation`) - validate target_agent exists, prompt is valid, send_context is boolean
- [ ] T038 [US3] Add handoff error handling (`agent/handoff.go:error_handling`) - clear errors for invalid targets, circular chains, permission violations
- [ ] T039 [P] [US3] Create handoff workflow tests (`tests/handoff_test.go`) - test valid handoffs, invalid targets, circular detection, context passing
- [ ] T040 [US3] Implement handoff audit logging (`agent/handoff.go:audit_log`) - log all handoff attempts, sources, targets, context passed
- [ ] T041 [P] [US3] Add @mention syntax documentation (`docs/handoff-syntax.md`) - examples, supported agents, context passing behavior

**Checkpoint**: Agents can delegate to subagents via @mention and programmatic handoffs with context passing

---

## Phase 6: User Story 4 - Agent Enforces Capability Constraints (Priority: P1) 🎯 MVP

**Goal**: System enforces constraints defined in agent frontmatter, preventing agents from violating their boundaries.

**Independent Test**: Attempt prohibited action from constrained agent (e.g., specwriter calling speckit.implement), verify system blocks with meaningful error.

### Implementation for User Story 4

- [ ] T042 [P] [US4] Implement constraint enforcement hook (`agent/enforcement.go`) - intercept handoff, file, command attempts before execution
- [ ] T043 [P] [US4] Create constraint type handlers (`agent/constraints.go:handlers`) - implement handoff_deny, file_pattern_deny, command_deny logic
- [ ] T044 [US4] Integrate constraint checking in permission system (`permission/permission.go:check_constraints`) - check constraints during Permission.ask() calls
- [ ] T045 [P] [US4] Implement constraint violation error messages (`agent/enforcement.go:error_messages`) - clear, actionable messages explaining why action blocked
- [ ] T046 [P] [US4] Create constraint violation audit trail (`agent/enforcement.go:audit`) - log attempts, reasons, agent involved via event bus
- [ ] T047 [US4] Add constraint inheritance for subagents (`permission/subagent.go:inherit_constraints`) - subagents inherit parent constraints
- [ ] T048 [P] [US4] Create constraint enforcement tests (`tests/constraint_enforcement_test.go`) - test handoff denial, file pattern blocking, command denial
- [ ] T049 [P] [US4] Add constraint visualization (`cmd/agent_info.go:show_constraints`) - display agent constraints in human-readable format
- [ ] T050 [US4] Create constraint documentation (`docs/agent-constraints.md`) - best practices, constraint patterns, examples

**Checkpoint**: Constraints are enforced; agents cannot violate their defined boundaries

---

## Phase 7: User Story 5 - Multi-Location Agent Discovery (Priority: P2)

**Goal**: Agents can be placed in repo-level or global locations; repo-level takes precedence; system handles missing directories gracefully.

**Independent Test**: Create same agent in both locations, verify repo version is used, remove repo version, verify global version becomes available.

### Implementation for User Story 5

- [ ] T051 [P] [US5] Implement global agent location scanning (`agent/discovery.go:scan_global`) - safely handle missing `~/.opencode/agents/` directory
- [ ] T052 [P] [US5] Implement repo agent location scanning (`agent/discovery.go:scan_repo`) - scan `.opencode/agents/` with proper error handling
- [ ] T053 [US5] Implement precedence logic (`agent/discovery.go:merge_with_precedence`) - repo agents override global, handle conflicts, create override tracking
- [ ] T054 [P] [US5] Add location tracking to Agent entity (`agent/registry.go:location_field`) - track "repo" vs "global" for each loaded agent
- [ ] T055 [P] [US5] Implement permission handling for global location (`agent/discovery.go:permission_fallback`) - gracefully handle permission errors for ~/.opencode/agents/
- [ ] T056 [US5] Add location information to agent list output (`cmd/agent_list.go:show_location`) - display "Repo" or "Global" for each agent
- [ ] T057 [P] [US5] Create multi-location discovery tests (`tests/discovery_precedence_test.go`) - test repo override, global-only agents, missing directories
- [ ] T058 [P] [US5] Add multi-location documentation (`docs/agent-locations.md`) - explain repo vs global, override behavior, best practices
- [ ] T059 [US5] Implement agent listing with location info (`cmd/agent_list.go:with_locations`) - show which agents are repo vs global

**Checkpoint**: Multi-location discovery works with proper precedence; agents available from both repo and global

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Documentation, testing, validation, and cross-feature integration

- [ ] T060 Implement opencode agent validate command (`cmd/agent_validate.go:full_impl`) - comprehensive validation of all agents, detect issues, report summary
- [ ] T061 [P] Create comprehensive agent validation tests (`tests/agent_validation_test.go`) - test all validation rules, error detection, circular handoff detection
- [ ] T062 Create agent best practices guide (`docs/agent-best-practices.md`) - design patterns, naming conventions, constraint strategies, handoff patterns
- [ ] T063 [P] Add integration tests for agent workflows (`tests/integration_agent_workflow_test.go`) - end-to-end: create, discover, select, handoff, constrain
- [ ] T064 [P] Create example custom agents (`examples/agent-*.md`) - domain examples (security-auditor, qa-tester, performance-analyzer)
- [ ] T065 Add agent discovery metrics/instrumentation (`agent/metrics.go`) - track discovery time, registry size, handoff frequency
- [ ] T066 [P] Create FAQ documentation (`docs/agent-faq.md`) - common questions, troubleshooting, examples
- [ ] T067 [P] Add agent switching to CLI help text (`cmd/help.go:agents`) - document Shift+Tab, agent commands, syntax
- [ ] T068 Create migration guide for existing agents (`docs/migration-custom-agents.md`) - if there are pre-existing agents, how to migrate
- [ ] T069 [P] Setup agent definition linting (`tools/agent_lint.go`) - style checking, format validation, best practices enforcement
- [ ] T070 [P] Create audit trail export utilities (`cmd/audit_export.go`) - export constraint violations, handoff logs for compliance/debugging
- [ ] T071 Create speckit agent integration tests (`tests/speckit_integration_test.go`) - verify custom agents can handoff to speckit.specify, speckit.plan, etc.
- [ ] T072 [P] Add agent discovery performance benchmarks (`tests/benchmarks/discovery_benchmark_test.go`) - ensure sub-2-second discovery for 50+ agents
- [ ] T073 Add debug/verbose logging (`agent/logger.go`) - trace discovery, validation, loading, switching steps for troubleshooting
- [ ] T074 [P] Create release notes template (`templates/release-notes-agents.md`) - document agent infrastructure feature for users

**Checkpoint**: Feature is production-ready with comprehensive documentation and testing

---

## Dependencies & Execution Order

### Dependency Graph

```
Phase 1 (Setup)
  ↓
Phase 2 (Foundational - ALL prerequisites)
  ├─→ Phase 3 (US1 - Agent Definition)
  │    └─→ Phase 4 (US2 - UI Switching) [requires US1 to work]
  │    └─→ Phase 6 (US4 - Constraints) [requires US1 + Phase 2]
  ├─→ Phase 5 (US3 - Handoffs) [requires Phase 2 + US1]
  └─→ Phase 7 (US5 - Multi-Location) [requires Phase 2]
  
Phase 8 (Polish) [depends on US1 + US2 + US3 + US4 + US5]
```

### Critical Path (Minimal MVP)

1. **Phase 1** (Setup): 4 tasks, ~2 hours
2. **Phase 2** (Foundational): 10 tasks, ~20 hours (BLOCKING)
3. **Phase 3** (US1): 9 tasks, ~15 hours (MVP requires this)
4. **Phase 6** (US4 - Constraints): 9 tasks, ~12 hours (MVP requires this for safety)
5. **Phase 8** (Polish essentials): ~8 hours (docs, validation, tests)

**Minimal MVP completion**: ~57 hours (1.4 weeks at 40 hrs/week)

### Parallel Execution Opportunities

**Phase 2 Parallelization** (can work in parallel):
- T006 (CI/CD), T007 (logging), T008 (test setup) can run in parallel with T005-T009
- T010-T014 can run in parallel once T005-T009 started

**Phase 3 Parallelization** (once Phase 2 complete):
- T015, T016, T018, T019 (validation, contract, template) can run in parallel
- T020-T023 (commands, tests) can run in parallel

**Suggested Parallel Teams**:
- **Team A** (6-8 hours): T005, T009, T017, T020-T021 (Registry + Discovery + Commands)
- **Team B** (6-8 hours): T006-T007, T008, T014 (CI/CD, Logging, Validation)
- **Team C** (5-6 hours): T015-T016, T018-T019, T022-T023 (Validation, Template, Tests)
- **Team D** (5-6 hours): T024-T032 (US2 UI Switching)
- **Team E** (5-6 hours): T033-T041 (US3 Handoffs)
- **Team F** (5-6 hours): T042-T050 (US4 Constraints)
- **Team G** (5-6 hours): T051-T059 (US5 Multi-Location)

---

## MVP Scope & Delivery Strategy

### Recommended MVP Scope (US1 + US4 only)

Phase 1 (Setup) + Phase 2 (Foundational) + Phase 3 (US1) + Phase 6 (US4) = ~57 hours

**What's included in MVP**:
- ✅ Agent developers can create and register custom agents
- ✅ Agents are automatically discovered and listed
- ✅ Agent constraints are enforced (safety/reliability)
- ✅ Basic agent info and list commands

**What's deferred to Phase 2**:
- ⏳ UI agent switcher (Phase 4 - US2) - add in next release
- ⏳ Agent handoffs (Phase 5 - US3) - add when needed
- ⏳ Multi-location discovery (Phase 7 - US5) - can start global-only
- ⏳ Full polish/docs (Phase 8) - minimal docs for MVP

### Incremental Delivery Timeline

1. **Sprint 1** (Week 1): Phases 1-2 (Foundation)
2. **Sprint 2** (Week 2): Phase 3 (US1 - Agent Creation)
3. **Sprint 3** (Week 3): Phase 6 (US4 - Constraints)
4. **Sprint 4** (Week 4): Phase 8 Polish, Phase 4 (US2 - UI Switcher)
5. **Sprints 5+**: Phase 5 (US3), Phase 7 (US5), Advanced features

---

## Task Status & Metrics

**Total Tasks**: 74  
**Tasks by User Story**:
- Setup: 4
- Foundational: 10
- US1 (Agent Creation): 9
- US2 (UI Switching): 9
- US3 (Handoffs): 9
- US4 (Constraints): 9
- US5 (Multi-Location): 9
- Polish: 15

**Parallelizable Tasks [P]**: 43 (58%)  
**Sequential Tasks**: 31 (42%)

**Estimated Effort**:
- Phase 1: 2 hours
- Phase 2: 20 hours
- Phase 3: 15 hours
- Phase 4: 16 hours
- Phase 5: 14 hours
- Phase 6: 12 hours
- Phase 7: 10 hours
- Phase 8: 12 hours
- **Total**: ~101 hours (2.5 weeks at 40 hrs/week, with parallelization)

---

## Key Architecture Decisions Embedded in Tasks

1. **Agent Registry as Singleton** (T005): In-memory, loaded at startup, cached for performance
2. **YAML Frontmatter for Metadata** (T006): Follows OpenCode conventions, matches existing agent definitions
3. **Permission System Integration** (T011): Translate constraints to OpenCode Permission.Ruleset for enforcement
4. **Session Preservation Across Switches** (T028): Same session.ID, update system prompt only
5. **Repo-Level Override** (T051-T053): Repo agents take precedence over global with same ID
6. **Circular Handoff Detection** (T014, T057): Topological sort at discovery time, not runtime
7. **Multi-agent Constraint Inheritance** (T047): Subagents inherit parent constraints for safety

---

## Testing Strategy

### Unit Tests (by phase)
- **Phase 2**: Registry, parser, discovery, constraint validation
- **Phase 3**: Agent definition validation, metadata extraction
- **Phase 4**: Agent switching, session preservation
- **Phase 5**: Handoff parsing, context passing, circular detection
- **Phase 6**: Constraint enforcement, violation detection
- **Phase 7**: Discovery precedence, multi-location handling

### Integration Tests
- **Cross-story**: Create agent → Discover → Select → Enforce Constraints → Handoff
- **Error scenarios**: Invalid definitions, circular handoffs, missing targets, permission errors
- **Performance**: Sub-2-second discovery for 50+ agents, memory efficiency

### Contract Tests
- **Agent Definition Format** (T015): Validate against contract spec
- **CLI Commands** (T020-T021): Verify output format, exit codes

---

## Success Criteria

✅ **All Phases Complete**:
- Agent developers can create agents (US1)
- Users can switch agents via UI (US2)  
- Agents can delegate via handoffs (US3)
- Constraints are enforced (US4)
- Multi-location discovery works (US5)

✅ **Quality Metrics**:
- 90%+ test coverage (all phases except polish)
- Sub-2-second agent discovery (50+ agents)
- Zero unhandled errors (all edge cases covered)
- Audit trail for all constraint violations
- Comprehensive documentation (docs/ and examples/)

✅ **User Experience**:
- Shift+Tab switcher is responsive (<100ms)
- Error messages are clear and actionable
- Agent creation is well-documented
- Examples included for common patterns
