# Agent Library — Base Template & Universal Standards

**Created:** 2026-05-28 CST
**Purpose:** Define-once, invoke-forever. Every specialized agent in this directory inherits these standards so the invoker need only say *what* task to do, not *how*.

## How to invoke a library agent

These files sync to `~/.claude/agents/` via the SessionStart hook (`sync-rules-on-start.sh`). They become subagent types Claude Code can launch with a minimal prompt:

```python
Agent(
    prompt="<task only — the HOW lives in the agent definition>",
    subagent_type="patch-applier",
    team_name="my-team",
    name="apply-bx-patch"
)
```

The invoker supplies the task; the definition supplies the role, backstory, SOP, output discipline, and anti-patterns.

## Universal standards (every agent inherits)

### 1. Worker mentality (role preamble baked in)
Every agent is a **sub-agent worker**, never an orchestrator. CLAUDE.md Rule #8 ("Claude orchestrates, agents execute") applies to the PARENT, not to these agents. They:
- Execute directly using Bash, Read, Write, Edit, Grep, Glob.
- Never spawn other agents. Never try to delegate or orchestrate.
- Message team-lead via SendMessage if genuinely blocked.
- Use Bash heredocs (`cat >> file << 'EOF'`) for large file writes when the Write tool is awkward.

### 2. Output discipline (non-negotiable)
- Write organized results to `~/.claude/agent-logs/<descriptive-name>.md` — never `output.md`.
- The log file holds ORGANIZED material, not raw tool dumps.
- Return a summary ≤500 tokens to the parent, with the log path explicitly referenced.
- SendMessage to team-lead when done.

### 3. Verify before declare (RLM Phase 4)
- Never claim "done" without proof: command exit code, `node --check`, sentinel count, health-check response, test summary.
- "It should work" is not acceptable. Show the verification.

### 4. Thinking budget
Each agent declares a tier (brief / step-by-step / careful / exhaustive) proportional to its domain risk. Spend reasoning where mistakes are expensive; stay fast where they aren't.

### 5. Fleet awareness
The 3-machine fleet uses SSH aliases `pc` (Windows, 100.102.119.55), `mac` (100.117.90.53), `oci` (100.90.164.6). Windows services live in WSL2 systemd. Always confirm the target machine before running OS-specific commands.

## File structure (every agent follows)

```markdown
---
name: agent-name
description: One line — when to use this agent
tools: [list of tools this agent needs]
---

# [Agent Name]

## Role
## Backstory
## Standard operating procedure
## Output discipline
## Thinking budget
## Common patterns
## What this agent never does
```

## Agents in this library

| Agent | When to use | Thinking |
|-------|-------------|----------|
| `patch-applier` | Apply patches to cli.js on Windows PC via SSH | careful |
| `plan-writer` | Write implementation plans + ntfy notify | step-by-step |
| `memory-updater` | Update ClaudeNotes memory files / session summaries | brief |
| `service-manager` | Manage WSL2 systemd services on PC | careful |
| `router-proxy-patcher` | Modify router-proxy.py (:8016 proxy) | careful |
| `ssh-investigator` | Diagnose problems on Windows PC via SSH | step-by-step |
| `hook-writer` | Create/update Claude Code hooks | careful |
| `ntfy-notifier` | Send notifications to ntfy (:8090) | brief |

## Sync

Source of truth: `ClaudeNotes/shared/claude-agents/`. Auto-synced to `~/.claude/agents/` at SessionStart. Always edit here, never in `.claude/agents/` directly.

## Changelog
- 2026-05-28 — Created. 8 specialized agents + base template.
