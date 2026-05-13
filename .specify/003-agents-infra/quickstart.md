# Quickstart: Creating Your First Custom Agent

**Duration**: 10-15 minutes  
**Difficulty**: Beginner  
**Prerequisites**: Access to `.opencode/` directory in your repo

---

## Overview

This guide walks you through creating a custom agent from scratch. By the end, you'll have:
1. ✅ Created a new agent definition file
2. ✅ Registered it with OpenCode
3. ✅ Tested it in the agent switcher
4. ✅ Configured handoffs to delegate tasks
5. ✅ Set constraints to define boundaries

---

## Step 1: Create Agent Definition File (2 min)

Create a new file: `.opencode/agents/my-agent.md`

```markdown
---
version: "1.0"
schema: "opencode-agent-v1"
id: my-agent
name: My Custom Agent
description: My specialized agent for domain-specific tasks.
model: gpt-5.2-codex
is_primary: true
priority: 100
tags: [custom, domain]
role: |
  I am My Custom Agent. I specialize in [your domain].
  My expertise includes:
  - Task 1
  - Task 2
  - Task 3
  
  I am NOT responsible for [out of scope tasks].

handoffs: []  # Add later

constraints: []  # Add later

---

# Role

Detailed description of my role, expertise, and typical workflows.

## Capabilities

- What I CAN do
- Specific strengths
- Domain expertise

## Hard Constraints

- What I CANNOT do
- Out of scope work
- Delegated responsibilities

# $ARGUMENTS

Describe what you'd like me to help you with.
```

**What to change**:
- `id`: Use lowercase with hyphens (e.g., `security-auditor`, `db-optimizer`)
- `name`: Human-readable title (e.g., "Security Auditor", "DB Optimizer")
- `description`: 1-2 sentence summary
- `model`: Choose your LLM: `gpt-5.2-codex`, `claude-haiku-4.5`, `gemini-flash`, etc.
- `role`: Detailed description of what you specialize in
- `tags`: Categorize your agent

---

## Step 2: Register Agent (1 min)

```bash
cd /path/to/your/repo

# Validate your agent definition
opencode agent validate my-agent

# Should output:
# ✓ Agent "my-agent" is valid
#   Name: My Custom Agent
#   Model: gpt-5.2-codex
#   Is Primary: true
#   Capabilities: 3
#   Constraints: 0
#   Handoffs: 0
```

**If validation fails**, check the error message:

```bash
# ERROR: Agent "my-agent" missing required field "model"
# Fix: Add "model: gpt-5.2-codex" to frontmatter

# ERROR: Schema is not "opencode-agent-v1"
# Fix: Check the schema line in frontmatter

# ERROR: Agent "my-agent" syntax error in YAML
# Fix: Check frontmatter indentation and syntax
```

---

## Step 3: View Agent in List (1 min)

```bash
opencode agent list

# Should output:
# Available Agents:
# 
# PRIMARY AGENTS (switchable via UI):
# - Default Agent                gpt-5.2-codex
# - My Custom Agent              gpt-5.2-codex
# - Spec Writer                  claude-haiku-4.5
#
# SUBAGENTS (callable via @mention):
# - buildpro                     gpt-5.2
#
# Total: 4 agents
```

---

## Step 4: Test Agent in UI (2 min)

```bash
# Start OpenCode
opencode

# In the TUI:
# 1. Press Shift+Tab to open agent switcher
# 2. Look for your agent in the list
# 3. Select it with arrow keys and Enter
# 4. Type a message and press Enter to test
```

If your agent doesn't appear:
- Check that `is_primary: true` in frontmatter
- Run `opencode agent validate my-agent` to find errors
- Check `.opencode/agents/my-agent.md` file exists

---

## Step 5: Add Handoffs (3-5 min)

Handoffs let your agent delegate to other agents. Example:

```yaml
handoffs:
  - id: delegate-to-builder
    target: buildpro
    label: Hand Off to Build Specialist
    prompt: |
      I need help optimizing the build system for my specific domain.
      Here's the current configuration and performance metrics:
      ${USER_INPUT}
    send_context: true
    description: "Use when build system optimization is needed"

  - id: get-spec
    target: speckit.specify
    label: Write Feature Spec
    prompt: Create a feature specification for the following requirements
    send_context: false
```

**Key fields**:
- `id`: Unique identifier for this handoff (used internally)
- `target`: Agent ID you're delegating to (must exist)
- `label`: Button text shown to user
- `prompt`: Pre-filled text sent to target agent
- `send_context: true`: Pass conversation history; `false`: clean session

**Test your handoff**:

```bash
opencode agent list my-agent

# Should show:
# Handoffs:
#   - Handoff to Build Specialist (buildpro)
#   - Write Feature Spec (speckit.specify)
```

---

## Step 6: Define Constraints (2-3 min)

Constraints define what your agent CANNOT do:

```yaml
constraints:
  - id: no-impl
    type: handoff
    pattern: speckit.implement
    reason: "I specialize in [domain], not code implementation. Use buildpro for that."
    severity: hard

  - id: no-source
    type: file_pattern
    pattern: "**/*.{py,js,ts,go}"
    reason: "I cannot modify source code; that's for developers"
    severity: hard

  - id: no-bash
    type: command
    pattern: bash
    reason: "I cannot execute arbitrary shell commands. Use buildpro for system operations."
    severity: hard
```

**Constraint types**:

| Type | Pattern | Example | Purpose |
|---|---|---|---|
| `handoff` | Agent ID | `speckit.implement` | Prevent delegation to certain agents |
| `file_pattern` | Glob pattern | `**/*.py` | Prevent modifying certain files |
| `command` | Tool name | `bash` | Prevent using certain tools |

**Test your constraints**:

```bash
opencode agent validate my-agent

# Should list constraints:
# Constraints:
#   - no-impl (handoff → speckit.implement)
#   - no-source (file_pattern → **/*.{py,js,ts,go})
#   - no-bash (command → bash)
```

---

## Step 7: Enhance Role Description (2-3 min)

Write a detailed role that explains your agent's expertise:

```markdown
---
role: |
  I am Security Auditor. My specialty is identifying and mitigating security vulnerabilities
  in architecture, configuration, and deployment strategies.
  
  **Expertise**:
  - Security architecture review
  - Identifying compliance gaps (OWASP, HIPAA, SOC2, etc.)
  - Recommending mitigation strategies
  - Analyzing threat models
  
  **I am NOT**:
  - A developer who writes secure code (that's buildpro's domain)
  - An operations specialist who implements security controls (use deployment agents)
  - A tester who verifies security fixes (use test agents)
  
  **How I Work**:
  1. You describe your system architecture
  2. I analyze for security vulnerabilities
  3. I recommend fixes with rationale
  4. I hand off implementation to appropriate agents
---
```

---

## Step 8: Test Full Workflow (2-3 min)

```bash
# 1. Launch OpenCode
opencode

# 2. Switch to your agent (Shift+Tab)
# 3. Enter a prompt, e.g.:
#    "Audit the security of our microservices architecture:
#     - API Gateway (Express.js)
#     - Auth Service (JWT tokens)
#     - Database (PostgreSQL)
#     - Message Queue (RabbitMQ)"

# 4. Agent responds with vulnerability analysis

# 5. Ask it to delegate:
#    "Please hand off implementation of these fixes to buildpro"

# 6. Agent uses handoff to delegate to buildpro
```

---

## Common Examples

### Example 1: Documentation Specialist Agent

```yaml
---
version: "1.0"
schema: "opencode-agent-v1"
id: docwriter
name: Doc Writer
description: Specializes in writing clear, comprehensive documentation.
model: claude-haiku-4.5
is_primary: true
tags: [documentation, writing]
role: |
  I am Doc Writer. I specialize in creating clear, comprehensive documentation
  that makes complex topics accessible. I excel at:
  - Writing user guides and tutorials
  - Creating API documentation
  - Explaining architecture and design decisions
  - Organizing documentation structure
  
  I do NOT write code or implementation details.

handoffs:
  - id: ask-builder
    target: buildpro
    label: Ask Builder About Implementation
    prompt: "I need implementation details about [topic] to write accurate documentation"
    send_context: true

constraints:
  - id: no-code
    type: file_pattern
    pattern: "**/*.{py,js,ts,go,java,rb}"
    reason: "I write documentation, not code"
    severity: hard

---

# Role

I specialize in making complex topics understandable through clear writing...
```

### Example 2: Testing Specialist Agent

```yaml
---
version: "1.0"
schema: "opencode-agent-v1"
id: testmaster
name: Test Master
description: Specializes in designing comprehensive test strategies and test cases.
model: gpt-5.2-codex
is_primary: true
tags: [testing, qa]
role: |
  I am Test Master. I specialize in designing robust test strategies and writing
  comprehensive test cases that catch edge cases and prevent regressions.

handoffs:
  - id: verify-with-builder
    target: buildpro
    label: Have Builder Verify Test Design
    prompt: "Please review my test strategy and help refine it"
    send_context: true

constraints:
  - id: no-impl
    type: handoff
    pattern: speckit.implement
    reason: "I design tests, I don't implement features"
    severity: hard

---

# Role

I specialize in comprehensive test design...
```

### Example 3: Subagent (Not in UI Switcher)

```yaml
---
version: "1.0"
schema: "opencode-agent-v1"
id: code-reviewer
name: Code Reviewer
description: Specialized code review agent (callable via @mention only).
model: claude-haiku-4.5
is_primary: false  # Not in UI switcher
tags: [review, code-quality]
role: |
  I am Code Reviewer. I specialize in thorough code review...

handoffs: []
constraints: []

---

# Role

I provide detailed code reviews...

# How to Use

You can invoke me by mentioning @code-reviewer in your message:

"@code-reviewer please review this function implementation"
```

---

## Troubleshooting

### Agent not appearing in switcher

**Problem**: You created the agent but it doesn't appear in Shift+Tab switcher.

**Causes**:
1. `is_primary: false` in frontmatter
2. Agent has validation errors
3. File not in `.opencode/agents/` directory
4. OpenCode hasn't reloaded agent registry

**Fix**:
```bash
# 1. Check validation
opencode agent validate my-agent

# 2. Check file location
ls -la .opencode/agents/my-agent.md

# 3. Check is_primary field
grep "is_primary" .opencode/agents/my-agent.md

# 4. Restart OpenCode
opencode
```

### Handoff target not found

**Problem**: Validation says "handoff target X does not exist"

**Causes**:
- Target agent ID is misspelled
- Target agent hasn't been created yet
- Target agent has validation errors

**Fix**:
```bash
# 1. List all agents
opencode agent list

# 2. Check spelling of target ID
grep "target:" .opencode/agents/my-agent.md

# 3. Validate target agent
opencode agent validate [target-id]
```

### YAML syntax error

**Problem**: "ERROR: Agent syntax error in YAML frontmatter"

**Causes**:
- Indentation is wrong (YAML is whitespace-sensitive)
- Missing quotes around strings
- Invalid field names

**Fix**:
- Check indentation (use spaces, not tabs)
- Quote values with special characters: `reason: "Don't use unquoted apostrophes"`
- Compare with example in this guide

Example of correct YAML:
```yaml
---
version: "1.0"
schema: "opencode-agent-v1"
handoffs:
  - id: handoff-1
    target: buildpro
    label: "Delegate to Builder"  # ← quotes around value with special chars
---
```

Example of incorrect YAML:
```yaml
---
version: 1.0  # ← should be quoted
schema: opencode-agent-v1
handoffs:
- id: handoff-1  # ← wrong indentation
target: buildpro  # ← should be indented under handoff
---
```

---

## Next Steps

Once you've created your first agent:

1. **Share it with team**: Copy `.opencode/agents/my-agent.md` to shared repo so teammates can use it
2. **Document use cases**: Add examples to your team's AGENTS.md guide
3. **Refine constraints**: As you use the agent, add constraints to prevent errors
4. **Add more agents**: Create specialized agents for other domains
5. **Set up handoffs**: Connect agents to create powerful workflows

---

## Getting Help

- **Check agent validation**: `opencode agent validate [name]`
- **View all agents**: `opencode agent list`
- **Read documentation**: See `AGENTS-HOWTO.md` and `AGENTS.md`
- **Report bugs**: Create issue with agent definition and error message

---

## Summary

You now know how to:

✅ Create a custom agent (Step 1)  
✅ Validate the definition (Step 2)  
✅ List and discover agents (Step 3)  
✅ Switch agents in the UI (Step 4)  
✅ Configure handoffs (Step 5)  
✅ Define constraints (Step 6)  
✅ Write detailed role descriptions (Step 7)  
✅ Test the agent end-to-end (Step 8)  

**Time invested**: 10-15 minutes  
**Agents created**: 1  
**Ready to scale**: ✅

Happy agent building! 🚀
