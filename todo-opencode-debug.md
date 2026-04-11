To run **OpenCode** in verbose/debug mode on Debian and verify that your system prompt is loading correctly, you should use the --print-logs or --log-level flags.
In the 2026 version of OpenCode, the "system prompt" is often composed of your global config and any project-specific AGENTS.md or OpenCode.md files.
### 1. The Debug Command
Run this in your Debian terminal to see exactly what is being sent to the LLM (including the system prompt):
```bash
opencode run "hi" --print-logs --log-level DEBUG

```
 * **--print-logs**: Forces the internal logs to stream directly to your terminal screen instead of just hiding in a file.
 * **--log-level DEBUG**: Ensures that the "Payload" (the data sent to the AI) is visible. You should see a large block of text labeled **"System Message"** or **"Prompt Context"**.
### 2. Checking if your System Prompt is Loaded
If you have a custom system prompt defined in ~/.config/opencode/opencode.json, you can verify it without running a full AI request by using:
```bash
opencode config show --system

```
If that returns nothing or a default message, OpenCode isn't picking up your custom file. In that case, check the following locations:
 * **Global:** ~/.config/opencode/opencode.json
 * **Project-specific:** ./.opencode/AGENTS.md (OpenCode automatically injects this into the system prompt if you are in that directory).
### 3. Where are the log files?
If the terminal output is moving too fast to read, Debian stores the full debug history here:
```bash
tail -f ~/.local/share/opencode/log/*.log

```
*Run this in a **second tmux pane** while you start OpenCode in the first one to watch the system prompt load in real-time.*
### 💡 Pro-Tip for 2026:
If you are using the **"Plan"** or **"Build"** agents, they have their own internal system prompts that are harder to override. If your custom prompt isn't showing up, try running with the --raw flag to bypass the agent logic:
```bash
opencode run "test" --raw --print-logs

```
**Did the --print-logs flag show the system prompt block, or is it still showing the default "You are a helpful assistant" message?**
