---
date: 2026-03-29
topic: CLI Anything
sources: 7
notebook_id: 97d6a50b-5abd-4b85-9c21-a0458bd3eecb
tags: [ai-agents, cli, developer-tools, automation, open-source]
---

# CLI Anything

Generates a CLI easily for pretty much any open source software.

![CLI Anything Infographic](deliverables/2026-03-29-cli-anything-infographic.png)

[Listen to the audio overview](deliverables/2026-03-29-cli-anything-podcast.mp3) | [Watch the video overview](deliverables/2026-03-29-cli-anything-video.mp4)

---

## What is it?

CLI-Anything is an open-source Claude Code plugin from the HKU Data Science Lab that automatically generates production-ready command-line interfaces for any software with an accessible codebase. It transforms GUI-heavy applications — GIMP, Blender, LibreOffice, Draw.io, and dozens more — into "agent-native" tools that AI agents can control via structured text commands and JSON output.

The tool runs a fully automated 7-phase pipeline: it scans source code, maps GUI actions to underlying APIs, architects a command structure, implements a Click-based Python CLI with a stateful REPL mode and 50-level undo/redo stack, generates a comprehensive test suite, and publishes a pip-installable package to your system PATH.

Critically, CLI-Anything calls the real application backends for execution. When it wraps Blender, it uses Blender's actual rendering engine — not a toy reimplementation. This preserves the full professional capabilities of each tool. The project has been validated with over 1,850 tests across 16+ major applications at a 100% pass rate, and gained 13,000+ GitHub stars within six days of launch.

---

## When to use it

- **No existing API or MCP server.** The sweet spot is professional desktop software that lacks structured programmatic access — image editors, 3D tools, office suites, audio editors.
- **Terminal-centric AI workflows.** If you're already using Claude Code, Cursor, or similar agents, CLI-Anything keeps everything in the terminal instead of switching tabs or using brittle screen automation.
- **Cost-sensitive agent pipelines.** Compared to MCP servers, CLI-Anything is reportedly 30x cheaper in token usage because lightweight CLI commands don't bloat the agent's context window with full tool schemas.
- **Internal tooling.** Point it at a private repository to create agent-native interfaces for proprietary internal software — no vendor lock-in, MIT licensed.
- **Creative and media automation.** Batch-process images in GIMP, render 3D scenes in Blender, edit audio in Audacity, manage OBS scenes, or generate professional documents in LibreOffice — all via text commands.
- **CI/CD integration.** Build pipelines that produce professional deliverables (e.g., a GitHub Action that auto-generates PDF release notes from a LibreOffice template).

---

## When NOT to use it / Limitations

- **Closed-source software only available as compiled binaries.** The pipeline relies on analyzing source code. Without it, generation quality degrades significantly.
- **Software with no independent backend.** Purely visual applications that lack an API, backend engine, or structured file format won't produce functional CLIs.
- **Well-maintained MCP servers or APIs already exist.** If a tool like Slack or GitHub already has an official API or high-quality MCP server, use that instead.
- **Weak models.** Generation requires frontier-class models (Claude Sonnet 3.5+ or equivalent). Smaller models produce incomplete or broken CLIs.
- **Python-only output.** All generated CLIs use Python's Click framework — no Node.js, Go, or Rust option.
- **Windows limitations.** Native PowerShell is not supported; you must use Git Bash with cygpath or WSL.
- **Time cost for large codebases.** Simple tools wrap in 5 minutes, but a 200 MB repo can take 20+ minutes.
- **Security considerations.** Granting an agent full CLI access to workspace tools (e.g., Google Workspace) can expose sensitive data. Scope permissions carefully.

**Alternatives:** MCP servers (higher token cost but standard for web services), official APIs (stable but often incomplete), RPA/screen automation (works on closed-source but fragile and slow).

---

## Getting Started

**Prerequisites:** Python 3.10+, the target software installed locally, and Claude Code as your agent (other agents are experimental).

### 1. Install the plugin

```bash
/plugin marketplace add HKUDS/CLI-Anything
/plugin install cli-anything
```

Restart Claude Code or run `/reload plugins` to activate.

### 2. Generate a CLI from a codebase

Clone a target repository and point CLI-Anything at it:

```bash
git clone https://github.com/jgraph/drawio-desktop
/cli-anything ./drawio-desktop
```

This triggers the 7-phase pipeline (analyze, design, implement, plan tests, write tests, document, publish). For a medium codebase, expect ~15-20 minutes.

### 3. Install the generated CLI

```bash
cd drawio/agent-harness
pip install -e .
```

### 4. Verify and use

```bash
cli-anything-drawio --help
```

Then prompt Claude Code with a natural language task:

> "Using the drawio CLI, create a diagram of a typical SaaS backend architecture with an API gateway, microservices, and a database. Style it with professional colors and save it as a PNG."

Claude will issue structured commands to the CLI, which calls the real Draw.io backend to produce the diagram. All operations support undo/redo and a `--json` flag for machine-readable output.

---

## Sources

### Web
- [HKUDS/CLI-Anything GitHub](https://github.com/HKUDS/CLI-Anything)
- [CLI Anything Official Site](https://clianything.org/)
- [How to use CLI-Anything — Apidog](https://apidog.com/blog/how-to-use-cli-anything/)
- [CLI-Anything: How to Make Any Software Controllable by AI](https://pasqualepillitteri.it/en/news/391/cli-anything-software-agent-native-guide)

### YouTube
- [CLI-Anything Just Brought Claude Code Into The Future — Chase AI](https://www.youtube.com/watch?v=Uzd2ckXnsg0)
- [10 CLI Tools That Make Claude Code UNSTOPPABLE — Chase AI](https://www.youtube.com/watch?v=uULvhQrKB_c)
- [CLI-Anything: Claude Code Can Now Run ANY App (30x Cheaper Than MCP) — FuturMinds](https://www.youtube.com/watch?v=O5bjNhUMJV4)
