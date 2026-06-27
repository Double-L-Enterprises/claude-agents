# MCP Server Index
# Updated: 2026-06-27 CST
# Pointer file — lists connected MCP servers and what they do.
# Read the individual server file for tools it exposes.
# Use ToolSearch to actually load a tool's schema.

## Connected Servers

- [Gmail](mcp/gmail.md) — Read/send email, manage labels, search threads
- [Google Calendar](mcp/google-calendar.md) — List/create/update events, find free time
- [Google Drive](mcp/google-drive.md) — Search/read/create/copy files
- [Notion](mcp/notion.md) — Pages, databases, comments, search, views
- [Slack](mcp/slack.md) — Messaging (requires auth setup)
- [Cloudflare](mcp/cloudflare.md) — Workers, D1, KV, R2, Hyperdrive, docs search
- [AWS Marketplace](mcp/aws-marketplace.md) — Search/compare/evaluate cloud solutions
- [Microsoft Learn](mcp/microsoft-learn.md) — Search Microsoft/Azure docs and code samples
- [Hugging Face](mcp/huggingface.md) — Models, datasets, spaces, papers, hub search
- [Gitnexus Server](mcp/gitnexus-server.md) — Code graph analysis for indexed repos (starred GitHub repos)
- [Morningstar](mcp/morningstar.md) — Financial data (requires auth)

## How to Use
1. Scan this list to find the server that has what you need
2. Read the server file for specific tools and their descriptions
3. Use ToolSearch("description") to load the actual tool schema
4. Call the tool directly after ToolSearch loads it

## Servers NOT Listed (disabled)
- GitNexus MCP Hub — disabled (old project, 100+ tools, never used)
