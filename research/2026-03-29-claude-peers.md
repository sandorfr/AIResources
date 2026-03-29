---
date: 2026-03-29
topic: Claude Peers
sources: 8
notebook_id: 899af16c-39a6-44b0-87e8-1936d932ae8b
tags: [mcp, multi-agent, claude-code, peer-to-peer, coordination]
---

# Claude Peers

Get agents to talk to each others using an MCP.

![Claude Peers Infographic](deliverables/2026-03-29-claude-peers-infographic.png)

[Listen to the audio overview](deliverables/2026-03-29-claude-peers-podcast.mp3) | [Watch the video overview](deliverables/2026-03-29-claude-peers-video.mp4)

---

## What is it?

Claude Peers MCP is an open-source peer-to-peer discovery and messaging system built specifically for Claude Code. It lets multiple independent Claude Code sessions running on the same machine find each other, share context, and exchange messages in real time.

The system uses a hub-and-spoke architecture. A central broker daemon runs on `localhost:7899` and maintains a SQLite database (`~/.claude-peers.db`) as a registry for all active peers. Each Claude Code session spawns its own MCP server that registers with the broker on startup. Messages are delivered instantly via the `claude/channel` protocol, which pushes inbound messages directly into the Claude UI.

Every registered peer exposes its working directory, git branch, and an auto-generated summary of its current task, so agents can understand what their peers are doing before reaching out. All communication stays local to the machine for privacy and speed.

Claude Peers is noteworthy because it represents a shift from "AI as an assistant" to "AI as a team." It emerged as a community-driven solution to enable multi-agent coordination natively within Claude Code, pioneering workflows where agents review, delegate, and coordinate without human routing.

---

## When to use it

- **Large-scale refactors** that touch multiple layers simultaneously (backend + frontend, data + API).
- **Parallel feature development** where two or more streams might create code conflicts if not carefully coordinated.
- **Review + Implement workflows** where one agent reviews code while another implements changes, exchanging feedback instantly.
- **Monorepo coordination** across packages with shared dependencies.
- **Splitting monoliths** using separate agents for data layers and API routes while keeping type signatures aligned.

The core problem it solves is the "silo" effect: without coordination, multiple agents working on the same codebase create merge conflicts, break each other's interfaces, and waste tokens re-scanning files or undoing each other's work. Claude Peers eliminates the "human router" overhead of manually copy-pasting context between terminal tabs.

Software engineers managing complex projects with multiple active Claude sessions benefit most, especially those moving toward a model where specialized agents collaborate on shared goals.

---

## When NOT to use it / Limitations

**Skip it when:**
- You're doing routine, isolated tasks (single bug fixes, standalone refactors).
- Work is sequential and a single agent can handle it.
- You're budget-constrained, since each peer is an independent Claude session and token usage scales linearly.

**Limitations:**
- Requires **Bun**, **Claude Code v2.1.80+**, and a **claude.ai login** (API key auth alone won't work).
- Peers do not inherit the lead agent's conversation history; they start fresh with the project's `CLAUDE.md` and their specific task.
- The ecosystem is still experimental: the lead agent may sometimes try to do work itself instead of delegating.
- No built-in session resumption for interrupted teammate agents.

**Common pitfalls:**
- Using relative paths in MCP configuration instead of absolute paths.
- Stale peer lists after restarting one agent but not the broker daemon.
- Assigning tasks with heavy circular dependencies between peers.

**Alternatives:**
- **Claude Code Agent Teams** (official Anthropic feature): native multi-agent with built-in peer messaging, shared task lists, and T-Mux monitoring.
- **Claude Co-work Sub-agents**: better for isolated parallel tasks where agents report only to the hub.
- **Ralphie**: community tool with branch-per-task isolation and multi-backend support (OpenAI, etc.).
- **n8n / Make.com**: for massive, non-human-in-the-loop workflows where traditional automation is more reliable and cost-effective.

---

## Getting Started

### Prerequisites

- [Bun](https://bun.sh) runtime installed
- Claude Code v2.1.80+
- Logged in via `claude.ai` (not API key only)

### Install

```bash
git clone https://github.com/louislva/claude-peers-mcp.git
cd claude-peers-mcp
bun install
```

### Register the MCP server

Use an **absolute path** to avoid the most common configuration error:

```bash
claude mcp add claude-peers "bun run /absolute/path/to/claude-peers-mcp/server.ts"
```

### Launch with channels enabled

The `--channel` flag enables instant push notifications between peers:

```bash
claude --channel
```

Add an alias for convenience:

```bash
alias c="claude --channel"
```

### First coordinated session

1. **Terminal 1:** `claude --channel`
2. **Terminal 2:** `claude --channel`
3. In either terminal, ask: *"List all peers on this machine."* Claude calls `list_peers` and shows every running instance with its working directory and task summary.
4. Send a message: *"Send a message to peer [ID]: 'I just updated the database schema. Check if the API routes need adjustment.'"* The other instance receives it instantly.

The broker daemon starts automatically on first launch. Peer metadata is stored in `~/.claude-peers.db` on port `7899`.

**Optional:** Set `OPENAI_API_KEY` in your environment to enable auto-summary generation via `gpt-5.4-nano`, giving peers richer context about each other's work.

---

## Sources

### Web
- [GitHub - louislva/claude-peers-mcp](https://github.com/louislva/claude-peers-mcp)
- [How to Coordinate Multiple Claude Code Agents Without Losing Your Mind](https://dev.to/alanwest/how-to-coordinate-multiple-claude-code-agents-without-losing-your-mind-1i9f)
- [claude-peers by louislva | Glama](https://glama.ai/mcp/servers/louislva/claude-peers-mcp)
- [louislva/claude-peers-mcp | DeepWiki](https://deepwiki.com/louislva/claude-peers-mcp)
- [Introducing the Model Context Protocol | Anthropic](https://www.anthropic.com/news/model-context-protocol)

### YouTube
- [Claude Cowork is Here! Full Breakdown + Testing](https://www.youtube.com/watch?v=BWAr7gTkll8)
- [Build Agent Teams within Claude Cowork in 17 min](https://www.youtube.com/watch?v=A6RbawFHC80)
- [Claude Code Agent Teams (Full Tutorial)](https://www.youtube.com/watch?v=zm-BBZIAJ0c)
