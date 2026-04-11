In a **monorepo**, the biggest challenge for Llama 3.3/4 is "context drift"—getting lost in the sheer volume of packages. To keep Llama 4 Scout's agentic capabilities sharp, the prompt needs to emphasize **dependency awareness** and **cross-package impact**.
Here is a specialized **Monorepo Implementation Lead** system prompt.
### The Monorepo Spec-First Prompt
```markdown
<|begin_of_text|><|start_header_id|>system<|end_header_id|>
You are a Lead Monorepo Architect. Your goal is to implement complex, spec-compliant features across multiple packages while maintaining strict architectural boundaries.

### 1. MONOREPO DISCOVERY PROTOCOL
- **Scope Detection**: Upon any request, first identify which package(s) are affected. Look for the nearest `package.json`, `go.mod`, `Cargo.toml`, or `pyproject.toml`.
- **Dependency Mapping**: In your <thinking> block, note how changes in `@repo/core` or `libs/` will impact consumer applications. 
- **Spec Hierarchy**: Prioritize root-level `/docs` or `/specs` as the global source of truth, followed by package-specific `README.md` or `design.md` files.

### 2. TASK-LIST BIAS (AUTO-EXECUTION)
- **Active Task Hunting**: If no specific task is provided, grep for `TODO:`, `FIXME:`, or check the `PROJECT_ROADMAP.md`. 
- **Implementation Bias**: Do not ask for permission to start a task found in a spec file. If the path is clear, begin the implementation.
- **Completionism**: Your objective is to move tasks from "To Do" to "Done." Always update the corresponding task list/markdown file with a `[x]` upon completion of the code.

### 3. COMPLEX LANGUAGE PATTERNS
- **No Boilerplate**: Use advanced language features (e.g., Higher-Order Components, Generics/Templates, Monads, Async/Await concurrency patterns, or Type-level programming).
- **Industrial Standards**: Implement robust error handling, telemetry hooks (OpenTelemetry), and unit tests (using the repo's established framework) by default.
- **Spec Linkage**: Annotate complex blocks with the specific requirement ID they satisfy from the documentation.

### 4. OPERATIONAL FLOW
1. **<thinking>**: Map the task to the monorepo structure + identify relevant specs + check task list status.
2. **Impact Analysis**: List any breaking changes for other packages in the repo.
3. **Execution**: Provide the code.
4. **Task Update**: Provide a shell command or instruction to mark the task as complete in the repo's tracking file.
<|eot_id|>

```
### Tips for Monorepo Spec-Driven Dev
#### 1. The "Spec-Anchor" Technique
If your monorepo is huge, Llama might miss a spec file buried 4 levels deep. In your opencode.md or project root, keep a **context.map** or a simple index.md that lists:
 * Root Spec: /docs/architecture.md
 * Task List: /tasks/current_sprint.md
 * API Schema: /packages/api/spec.yaml
Llama 4 Scout is excellent at following these "breadcrumbs" if you mention them once in the initial session.
#### 2. Handling "Complex" Code
Llama 3.3 70B can sometimes revert to "helpful assistant" mode and give you simplified examples. To prevent this, use the **"Reference Implementation"** trick:
> *"Implement the Auth provider in @repo/auth. Follow the complexity level of the existing implementation in @repo/billing."*
> 
#### 3. Recommended Repos for Monorepo Prompting
 * **Turborepo Examples:** Great for seeing how professional monorepos structure their README and tasks—Llama models love the way Vercel structures metadata.
 * **Protocols for Agentic Coding:** While designed for Claude, the "Computer Use" and "File System" prompt patterns here are incredibly effective when ported to Llama 4 Scout for monorepo navigation.
Since you're in a monorepo, do you have a specific tool managing it (like **Turborepo, Nx, or Bazel**)? I can add specific logic for running their CLI commands (e.g., turbo run build --filter...) into the system prompt.

### 4. placeholder

### 5. CO-AGENT SYNERGY & ALIGNMENT
- **Rule Mirroring**: You MUST check for `.github/copilot-instructions.md` or `AGENTS.md` at the start of the session. Adopt all style, architectural, and naming conventions defined there to ensure consistency with other AI agents (Copilot/Claude).
- **State Handoff**: When finishing a task, append a "Handoff Note" in a comment if there is a specific context another agent should know when the user switches tools (e.g., `// AI-CONTEXT: Refactored X to support Y, needs Z implementation next`).
- **Conflict Resolution**: If a repository "Spec" contradicts a "Copilot Instruction," the Spec takes precedence for logic, but the Instruction takes precedence for syntax/formatting.

##Replacing open code default agents

Yes, it is very easy. In 2026, OpenCode moved away from hardcoded logic to a **"Mode-based" configuration system**. You can either override the existing plan/build agents or create entirely new ones (like a review or refactor agent).
### 1. The Global Override (Permanent)
To change the system prompts for every project, edit your global JSON config in Debian:
```bash
nano ~/.config/opencode/opencode.json

```
Add or modify the "agent" (or "mode") section:
```json
{
  "agent": {
    "plan": {
      "model": "anthropic/claude-3-5-sonnet", 
      "prompt": "{file:~/.config/opencode/prompts/my-custom-plan.txt}",
      "tools": { "webfetch": true, "bash": false }
    },
    "build": {
      "prompt": "Act as a Senior Debian SysAdmin. Use minimalist code. Always include unit tests.",
      "temperature": 0.2
    }
  }
}

```
 * **The {file:...} trick**: You can point the prompt to a separate .txt file so you don't have to manage a giant string inside a JSON file.
### 2. The Project-Specific Override (Flexible)
If you only want custom agents for a specific project (e.g., a React project vs. a Rust project), use an **AGENTS.md** file in your project root.
OpenCode reads this file and injects it into the agent's instructions.
```markdown
# AGENTS.md

## General Instructions
- We use Tab-size: 2
- No semicolons in TypeScript

## @build
- You are an expert at Debian WSL optimization.
- Before writing code, check if the library is available via `apt`.

## @plan
- Focus on performance and memory usage.
- Output plans in Mermaid diagram format.

```
### 3. How to Verify They Swapped
Run the debug command we used earlier to see your new instructions in the "System Message" block:
```bash
opencode run "check my system prompt" --print-logs --log-level DEBUG

```
### 💡 Pro-Tip: The "Tab" Switcher
Once you have custom agents, you can cycle through them instantly in the OpenCode TUI by hitting **Tab**.
 * **Plan Mode**: Great for "Thinking" and searching (usually read-only).
 * **Build Mode**: Great for "Doing" (has permission to edit files).
 * **Your Custom Mode**: Will appear in the list if you've added it to your JSON.
### Summary of Agent Hierarchy
| Level | Location | Best for... |
|---|---|---|
| **System** | Built-in | Default fallback. |
| **Global** | ~/.config/opencode/opencode.json | Your personal "coding style." |
| **Project** | ./AGENTS.md | Team standards and specific tech stacks. |
**Are you planning to make the "Build" agent more aggressive (e.g., auto-running tests) or more conservative (e.g., always asking for permission)?**

