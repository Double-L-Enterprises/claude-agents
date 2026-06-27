# Tool Reference for Agents
# Updated: 2026-06-27 CST
# Section 1: ALL base tools inline (auto-injected, ready from turn 1)
# Section 2: Pointer index to individual tool files (role-specific, on-demand)

---

## Section 1 — Base Tools (loaded into every agent's context)

### Bash — Run shell commands
```
Bash(command="your command here")
Bash(command="long running command", timeout=300000)  # 5 min timeout
```
- Chain: `cmd1 && cmd2` (dependent) or `cmd1; cmd2` (independent)
- Capture stderr: `command 2>&1`
- Quote paths with spaces: `cat "path with spaces/file.txt"`
- Output capped at 8K chars. For longer output: redirect to file, then Read it.
- DON'T: `rm`/delete (blocked), `git push --force`, commands >2min without timeout.

### Read — Read file contents
```
Read(file_path="/absolute/path/to/file")
Read(file_path="/absolute/path", offset=100, limit=30)  # lines 100-130 only
Read(file_path="/path/to/doc.pdf", pages="1-5")  # PDF pages
```
- ALWAYS absolute paths, never relative.
- For files >500 lines: Grep first to find the section, then Read with offset/limit.
- Can read images (you're multimodal) and Jupyter notebooks.
- DON'T: Read entire 2000-line files. Re-read same file 3+ times. Use relative paths.

### Write — Create or overwrite files
```
Write(file_path="/absolute/path/to/new-file.py", content="full file content here")
```
- For NEW files and complete rewrites ONLY.
- Must Read existing file before overwriting (enforced).
- For small changes to existing files, use Edit instead.
- DON'T: Use Write to change 3 lines in a 500-line file — that's Edit's job.

### Edit — Modify existing files
```
Edit(file_path="/path/to/file.py", old_string="exact text to find", new_string="replacement")
Edit(file_path="/path", old_string="old_var", new_string="new_var", replace_all=true)
```
- ALWAYS Read the file first (enforced — Edit fails without prior Read).
- `old_string` must be UNIQUE in the file — include surrounding context if needed.
- Preserve exact indentation (tabs/spaces) from the file.
- DON'T: Guess at old_string. Include line numbers in old_string. Skip Read first.

### Grep — Search file CONTENTS
```
Grep(pattern="def my_function", path="/src")                          # find files
Grep(pattern="error.*timeout", path="/src", output_mode="content")    # show matching lines
Grep(pattern="import", path="/src", glob="*.py", output_mode="count") # count matches
Grep(pattern="TODO", path="/src", output_mode="content", -A=3, -n=true)  # with context + line nums
```
- `output_mode`: "files_with_matches" (default), "content" (show lines), "count"
- `glob`: filter files — "*.py", "*.ts", "*.sh"
- `-A`/`-B`/`-C`: lines after/before/around each match. `-i`: case insensitive. `-n`: line numbers.
- Grep searches CONTENTS. Glob finds files by NAME. They're different tools.

### Glob — Find files by NAME pattern
```
Glob(pattern="**/*.py")                              # all Python files anywhere
Glob(pattern="**/settings.json", path="/home")       # find settings.json
Glob(pattern="test_*.py", path="/project/tests")     # test files in specific dir
```
- `**` matches any directory depth. `*` matches within one directory.
- Results sorted by modification time (newest first).
- Glob finds files by NAME. Grep searches file CONTENTS.

### SendMessage — Talk to teammates
```
SendMessage(to="team-lead", summary="5 word label", message="your content here")
```
- `to`: teammate NAME — "team-lead", "researcher", "coder", etc.
- `summary`: short notification label (5-10 words).
- `message`: content. KEEP UNDER 2000 CHARS. Write full output to agent-logs.
- Patterns:
  - START: `"Understood: [task]. Plan: [N steps]. Questions: [any or none]."`
  - PROGRESS: `"[2/3] Completed: [what]. Next: [what]. Issues: [any]."`
  - STUCK: `"STUCK on [X] because [Y]. Do you have this in context, or should I search?"`
  - DONE: `"Done. Files: [list]. Report: ~/.claude/agent-logs/name.md"`
  - SHUTDOWN: `'{"type":"shutdown_response","request_id":"<id>","approve":true}'`

### ToolSearch — Discover MCP tools on demand
```
ToolSearch("description of what you need")
```
- Finds MCP tools ONLY (Notion, Gmail, agent-mcp, GitNexus, etc.).
- NOT for built-in tools — those are already loaded (this section).
- After ToolSearch returns a schema, call that tool directly.
- Key: `ToolSearch("agent-mcp")` for prior session memory search.
- Examples: `"search notion pages"`, `"gmail draft"`, `"google calendar"`, `"code graph"`.

### TaskCreate — Flag discovered work
```
TaskCreate(subject="Brief title", description="Details about what needs doing")
```
- Use when you DISCOVER work that isn't your assignment.
- Team-lead manages tasks — don't self-assign via TaskList.

### TaskUpdate — Update task status
```
TaskUpdate(taskId="1", status="completed")
```
- Only mark completed when FULLY done and verified.
- Statuses: pending, in_progress, completed.

---

## Section 2 — Role-Specific Tools (auto-injected per agent type, or read on demand)

These tools are loaded automatically based on your agent type.
If you need one that isn't loaded, Read the individual file.

### Core Tools (all agents)
- [Bash](tools/bash.md) — Run shell commands
- [Read](tools/read.md) — Read file contents
- [Write](tools/write.md) — Create or overwrite files
- [Edit](tools/edit.md) — Modify existing files
- [Grep](tools/grep.md) — Search file contents
- [Glob](tools/glob.md) — Find files by name pattern
- [SendMessage](tools/sendmessage.md) — Talk to teammates
- [ToolSearch](tools/toolsearch.md) — Discover MCP tools on demand

### Code Intelligence
- [LSP](tools/lsp.md) — Go to definition, find references, hover docs, symbols (coder, researcher, reviewer, debugger)

### Web Access
- [WebSearch](tools/websearch.md) — Search the web for docs, errors, packages (researcher, builder, coder, infra, reviewer, debugger)
- [WebFetch](tools/webfetch.md) — Fetch and parse a URL to markdown (researcher, builder)

### Monitoring
- [Monitor](tools/monitor.md) — Stream events from long-running processes (coder, infra, debugger)

### Notebooks
- [NotebookEdit](tools/notebookedit.md) — Edit Jupyter notebook cells (coder)

### Task Management
- [TaskCreate](tools/taskcreate.md) — Flag discovered work
- [TaskList](tools/tasklist.md) — List all tasks
- [TaskUpdate](tools/taskupdate.md) — Update task status
- [TaskGet](tools/taskget.md) — Get full details of a specific task
- [TaskOutput](tools/taskoutput.md) — Get output from a background task
- [TaskStop](tools/taskstop.md) — Stop a running background task

### Skills
- [Skill](tools/skill.md) — Invoke a predefined skill/workflow (rare — coordinator usually handles)

### Agent & Team Tools (coordinator only)
- [Agent](tools/agent.md) — Spawn a subagent to handle a task
- [TeamCreate](tools/teamcreate.md) — Create a team for agent coordination
- [TeamDelete](tools/teamdelete.md) — Delete a team and clean up resources

### Task Management
- [TaskGet](tools/taskget.md) — Get full details of a specific task
- [TaskOutput](tools/taskoutput.md) — Get output from a background task
- [TaskStop](tools/taskstop.md) — Stop a running background task

### Worktree & Plan Tools
- [EnterWorktree](tools/enterworktree.md) — Create isolated git worktree for parallel work
- [ExitWorktree](tools/exitworktree.md) — Leave a worktree session
- [EnterPlanMode](tools/enterplanmode.md) — Switch to plan mode for complex tasks
- [ExitPlanMode](tools/exitplanmode.md) — Exit plan mode and request user approval

### Scheduling & Workflow
- [AskUserQuestion](tools/askuserquestion.md) — Ask the user a question with multiple-choice options
- [CronCreate](tools/croncreate.md) — Schedule a recurring or one-shot prompt
- [CronDelete](tools/crondelete.md) — Cancel a scheduled cron job
- [CronList](tools/cronlist.md) — List all scheduled cron jobs
- [ScheduleWakeup](tools/schedulewakeup.md) — Schedule when to resume in /loop mode
- [Workflow](tools/workflow.md) — Run a multi-step subagent pipeline with deterministic control flow

### Notifications & Delivery
- [PushNotification](tools/pushnotification.md) — Push a notification to the user's device
- [SendUserFile](tools/senduserfile.md) — Attach files to the chat for download

### Design & Remote
- [DesignSync](tools/designsync.md) — Sync design system projects with claude.ai
- [RemoteTrigger](tools/remotetrigger.md) — Manage claude.ai remote triggers/routines

### MCP Tools (use ToolSearch)
Not listed individually — use `ToolSearch("description")` to discover.
Common: Notion, Gmail, Google Calendar, Google Drive, Slack, GitNexus,
agent-mcp (memory search), Hugging Face, Cloudflare, Microsoft Learn.

---

## Quick Reference

| Error | Fix |
|-------|-----|
| Edit failed (not unique) | Include more surrounding context in old_string |
| File not found | Glob(pattern="**/filename*") to find path |
| Bash timeout | Add timeout param or use Monitor |
| MCP tool missing | ToolSearch with different keywords |
| Permission denied | Message team-lead |
| SendMessage rejected | Use teammate NAME in `to`, not ID |
| Read too large | Grep first → Read with offset/limit |

| Cannot do | Why |
|-----------|-----|
| Delete files | Blocked — ask team-lead |
| Force push | Blocked — never attempt |
| Spawn agents | You're a worker — message team-lead |
| Use MCP without ToolSearch | Schema won't be loaded |
