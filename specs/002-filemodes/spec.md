Usually, that's found in a configuration file like a '.config' or a '.env' file in your project repository, or sometimes in the global settings for the application itself. Look for parameters like 'code_style' or 'file_modification_output'. If you can't find it, checking the documentation for your version of OpenCode is always a safe bet. Is there a specific part of the configuration you were looking in?

The reference you mentioned points to a common way to guide **OpenCode** (or similar AI coding agents) toward more surgical edits: tweaking settings in a project-level config file (like `.opencode.json`, `opencode.json`, or `.env`), global app settings, or via documented parameters that influence output style. Parameters resembling `code_style`, `file_modification_output`, `edit_mode`, or similar can steer the agent away from full-file rewrites. If those aren't exposed, the project's documentation for your specific version is the next stop.<grok:render card_id="b766b7" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">5</argument></grok:render>

Here are several **additional practical strategies** to discourage wholesale file replacement and encourage clean patches/diffs instead. These draw from patterns that work across many LLM-based coding tools (including OpenCode, Aider, Cursor, Claude-based agents, etc.):

### 1. Use Repository-Level Instruction Files
Create dedicated Markdown files in your project root that the agent automatically ingests or that you explicitly reference:

- **`CODE_STYLE.md`** or **`.opencode/rules.md`** — Explicitly state your preferred editing policy.
  Example content:
  ```
  ## Editing Rules
  - Never output or write an entire file unless it is brand new or a complete refactor is explicitly requested.
  - For any modification, output only a minimal **unified diff** (patch) or precise search/replace blocks.
  - Use context lines (starting with space) around changes. Prefer small, targeted hunks.
  - Do not include placeholders like "// ... rest of file unchanged" when rewriting files — instead, produce a proper diff that the tool can apply safely.
  - Always explore the file first with read tools before proposing edits.
  ```

- **`ARCHITECTURE.md`** or **AGENTS.md** — Add sections on change management, snapshots, and preferring in-place edits over overwrites. Many OpenCode setups auto-inject these files into the system prompt.<grok:render card_id="18eaa9" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">2</argument></grok:render>

These travel with the repo, stay version-controlled, and apply consistently for anyone (or any agent) working on the project.

### 2. Strengthen Your Prompts and System Instructions
Be explicit and repetitive in your queries:

- "Make **targeted edits only**. Output changes exclusively as a **unified diff** or search/replace format. Do not rewrite the full file."
- "Use the patch tool (or `apply_patch` / `edit_file` with diff) instead of full file writes. Show context lines for accurate application."
- "Plan the changes first in a separate step, then output only the minimal diff. Never output complete file content unless creating a new file."
- For OpenCode specifically: Reference its tools like the `patch` operation when available.<grok:render card_id="eaead2" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">7</argument></grok:render>

Some users get good results by asking the agent to first produce a **Technical Design Spec** (in a Markdown file) outlining exactly which small sections will change, then implement against that spec in follow-up steps. This reduces over-eager full rewrites.<grok:render card_id="9367bf" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">37</argument></grok:render>

### 3. Leverage Tooling and Modes in the Agent
- **Plan vs. Build/Execute modes**: Stay in a read-only "plan" or analysis mode as long as possible. Only switch to the mode that allows file changes once you've reviewed the proposed diffs. OpenCode has distinct plan and build agents — use the plan one for exploration.<grok:render card_id="67a208" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">11</argument></grok:render>
- **Snapshots and versioning**: Keep snapshots enabled (default in OpenCode) so you can easily revert aggressive changes. Disable only if performance demands it, and combine with git branches for safety.<grok:render card_id="f8317d" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">5</argument></grok:render>
- **Search/replace or diff-first tools**: If the agent supports structured tools (e.g., `apply_patch`, search-replace blocks, or LSP-aware edits), explicitly instruct it to use them over raw file writes. Some setups fallback gracefully to full rewrites only for new files or major refactors.

### 4. Workflow and Human-in-the-Loop Practices
- **Review diffs before apply**: Configure or request that the agent shows a preview diff and waits for approval.
- **Break tasks into small steps**: Instead of "refactor this large module," say "make this one specific change using a diff" and iterate. Smaller scopes naturally lead to patch-style outputs.
- **Use git integration**: Instruct the agent to stage changes, create branches, or generate commits with proper diffs. Some agents can open PRs with clean patch histories.
- **Test in isolated environments**: Run the agent on a copy or feature branch first.

### 5. Advanced Prompt Patterns That Help
- Request **unified diff format** explicitly, with `@@` context markers.
- Use "search/replace" blocks with surrounding context instead of full code.
- For stubborn models: Ask for a custom minimal patch format that you can then programmatically apply or convert.
- Combine with examples: Provide 1–2 short "good diff" examples in your instructions or rules file.

These techniques reduce token waste, lower the risk of accidental deletions or "keep the rest unchanged" hallucinations, and make changes easier to review and merge.<grok:render card_id="a30ee1" card_type="citation_card" type="render_inline_citation"><argument name="citation_id">26</argument></grok:render>

If you're running into this with a particular version of OpenCode or a specific model/provider, feel free to share more details (e.g., the exact config section you checked or an example of unwanted full-file output). That would let me refine the advice further!