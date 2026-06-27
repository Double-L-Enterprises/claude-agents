# Agent Orientation — Deep Dive
# Updated: 2026-06-26 CST
# Read this if you need more context about the environment you're working in.
# The auto-injected onboarding covers the essentials. This goes deeper.

## Where You Are

You're working in a custom AI infrastructure stack run by Double L Enterprises.
This is NOT a standard Claude Code installation. Key differences:

- **71 models** available through a router-proxy on port 8016
- **38 NVIDIA models** are free ($0 per call) — use these for routine work
- **3 machines** (Windows/WSL2, Mac, OCI) connected via Tailscale mesh
- **Custom hooks** enforce rules — if something blocks you, read the block message
- **agent-mcp** indexes ALL prior session knowledge — always check it first

## The Model Ecosystem

You don't pick your own model — the coordinator or auto-picker does that.
But you should know: the models here speak OpenAI API format through
a router-proxy at localhost:8016. If you need to make a direct API call:

```bash
curl -s http://localhost:8016/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"nvidia/nemotron-3-nano-30b","messages":[{"role":"user","content":"hello"}]}'
```

## The Memory System

This infrastructure has a sophisticated memory system. As an agent, you can access:

1. **agent-mcp** (semantic search over 4000+ indexed memories):
   - ToolSearch("agent-mcp") → loads the query tool
   - Search for anything: error messages, service names, file paths, concepts
   - Returns relevant memories with context from prior sessions

2. **Memory files** (direct file access):
   - ~/ClaudeNotes/shared/memory/ — cross-session findings, decisions, patterns
   - ~/ClaudeNotes/shared/troubleshooting/ — verified fix guides
   - ~/ClaudeNotes/shared/build-notes/ — architecture decisions, design docs
   - ~/ClaudeNotes/shared/infrastructure/ — service topology, port maps

3. **Rolling summary** (injected automatically):
   - The most recent session context is injected into your bootstrap
   - It tells you what the coordinator has been working on
   - Use it to understand your task's broader context

## Key Services (Windows/WSL2 machine)

| Port | Service | What it does |
|------|---------|-------------|
| 8016 | router-proxy | Routes ALL model requests. Entry point for everything. |
| 3002 | LiteLLM | API gateway with model catalog |
| 18010 | vLLM | Local Qwen3-Coder-30B for fast code generation |
| 8082 | LocalAI | Whisper STT + TTS (CPU) |
| 8030 | agent-mcp | On Mac (100.117.90.53). Memory index. MCP/stdio access. |
| 9090 | Prometheus | Metrics. curl localhost:9090/api/v1/query?query=up |
| 3001 | Grafana | Dashboards. No login required. |
| 4001 | SearXNG | Self-hosted web search |

## Common Patterns

### Finding a file you know exists
```bash
# By name
Glob(pattern="**/router-proxy.py")

# By content
Grep(pattern="def _smart_pick", path="/home/larryloden")
```

### Checking if a service is running
```bash
systemctl is-active router-proxy
curl -s --max-time 5 localhost:8016/health
```

### Reading a large file efficiently
```bash
# Find what you need first
Grep(pattern="function_name", path="/path/to/file.py", output_mode="content")
# Then read just that section
Read(file_path="/path/to/file.py", offset=150, limit=30)
```

### Making changes safely
```bash
# ALWAYS: Read → Understand → Edit → Verify
Read(file_path="target.py")          # 1. understand current state
Edit(file_path="target.py", ...)     # 2. make the change
Bash(command="python3 -m py_compile target.py")  # 3. verify it parses
```

## What Hooks Do

Hooks are scripts that run before/after your tool calls. They may:
- **Block** an action (exit code 2) — read the message, it tells you why and how to proceed
- **Warn** about something (stderr message) — informational, action still proceeds
- **Inject context** (this onboarding is a hook!) — adds information to your context

If a hook blocks you:
1. Read the block message carefully — it says WHY and HOW TO PROCEED
2. Follow its instructions (usually: read a file first, use a different approach)
3. If still blocked after following instructions: message team-lead

## SSH Shortcuts

If you need to run commands on another machine:
```bash
ssh pc    # → Windows (100.102.119.55)
ssh mac   # → Mac (100.117.90.53)
ssh oci   # → OCI (100.90.164.6)
```

## Things That Will Get You Killed (terminated)

1. Going silent for 3+ minutes without a heartbeat
2. Trying to spawn other agents (you're a worker, not a coordinator)
3. Dumping 10K+ tokens of raw output to team-lead (write to file, summarize)
4. Deleting files without asking team-lead
5. Not responding to "status?" from team-lead within 2 minutes

## Things That Will Make You Effective

1. Check agent-mcp BEFORE starting (prior context saves hours)
2. Send heartbeats at natural work boundaries (not fixed intervals)
3. Say "STUCK" immediately when blocked — don't silently retry
4. Write detailed output to agent-logs, send concise summary
5. Read files before editing them
6. Use Grep to find sections, then Read with offset — don't read whole files
