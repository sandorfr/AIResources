---
date: 2026-03-29
topic: Garry Tan Gstack
sources: 7
notebook_id: a12d1882-7e77-4569-96b1-1a0f1379a2f6
tags: [claude-code, ai-workflows, developer-tools, startup-productivity, open-source]
---

# Garry Tan Gstack

Some leadership, startup incubator skill library.

![Garry Tan Gstack Infographic](deliverables/2026-03-29-garry-tan-gstack-infographic.png)

[Listen to the audio overview](deliverables/2026-03-29-garry-tan-gstack-podcast.mp3) | [Watch the video overview](deliverables/2026-03-29-garry-tan-gstack-video.mp4)

---

## What is it?

Gstack is an open-source skill pack for Claude Code created by Garry Tan, President and CEO of Y Combinator. It transforms a single AI coding assistant into a virtual software development team by decomposing the development lifecycle into specialized "Skills" — opinionated personas triggered via slash commands.

Each skill embodies a distinct engineering role. `/plan-ceo-review` acts as a product-minded founder who reframes features around user empathy. `/plan-eng-review` locks down architecture with sequence diagrams and failure mode analysis. `/review` performs a paranoid structural audit hunting for N+1 queries, race conditions, and stale reads. `/qa` spins up a real Chromium browser via Playwright to visually verify UI changes. `/ship` automates branch syncing, test runs, and PR creation.

Under the hood, Gstack uses an accessibility-tree interaction model (refs like `@e1`, `@e2`) instead of brittle CSS selectors, a persistent Chromium daemon that responds in 100-200ms with zero context tokens, and a Conductor tool that can orchestrate up to 10 parallel coding sessions in isolated workspaces. Tan publicly demonstrated averaging 10,000+ lines of production code per day using this setup while running YC full-time. The project surpassed 16,000 GitHub stars within days of release and is MIT-licensed.

---

## When to use it

- **Heavy Claude Code users** who want structured, role-driven workflows instead of generic prompting. Gstack forces the AI into specific "cognitive gears" so planning, reviewing, and shipping don't blur into a mediocre blend.
- **Technical founders and solo developers** who need to maintain high output and product taste simultaneously. The `/office-hours` skill applies YC-style forcing questions to reframe vague ideas into high-value product strategies.
- **Teams shipping production software** that need rigorous multi-stage review. The `/review` + `/codex` pipeline catches production-breaking bugs by getting a second opinion from OpenAI's Codex, while `/qa` performs automated browser-based visual testing.
- **Parallel sprint workflows** where a single developer manages 10-15 concurrent coding sessions via the Conductor tool, each in its own isolated workspace.
- **End-to-end SDLC coverage**: from product reframing (`/office-hours`) through architecture (`/plan-eng-review`), implementation, audit (`/review`), visual QA (`/qa`), and shipping (`/ship`).

---

## When NOT to use it / Limitations

- **Beginners or light AI users**: If you only use AI for occasional autocomplete, the multi-role overhead is overkill. Gstack is designed for developers already shipping production code heavily.
- **Budget-constrained environments**: Running 10-15 parallel sessions via Conductor can multiply Claude API costs by 10x or more.
- **Sensitive production environments**: The `/browse` skill uses a real browser with persistent cookies and session state — don't point it at production systems unless you understand the implications.
- **Windows**: Works via Git Bash or WSL but requires both Bun and Node.js due to a Playwright bug. Cookie import (`/setup-browser-cookies`) only supports macOS Keychain currently.
- **Iframe-heavy applications**: The accessibility-tree ref system cannot cross iframe boundaries.
- **Repo sync complexity**: The git-clone-and-copy installation model can become unwieldy when different repos need divergent review gates or QA flows.

**Alternatives**: GitHub Copilot for simpler in-editor assistance, K9 Audit for deterministic auditing, Greptile for async PR reviews. Gstack skills can also be adapted for OpenAI Codex CLI, Gemini CLI, or Cursor.

---

## Getting Started

**Prerequisites**: Claude Code, Git, and Bun v1.0+ installed. On Windows, also install Node.js.

**Install globally:**

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git \
  ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup
```

**Or install per-repo (for team sharing):**

```bash
cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && \
  rm -rf .claude/skills/gstack/.git && \
  cd .claude/skills/gstack && ./setup
```

**Add to your `CLAUDE.md`:**

```markdown
### gstack
- Use the /browse skill from gstack for all web browsing.
- Never use mcp__claude-in-chrome__* tools.
- Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /review,
  /ship, /qa, /browse, /retro, /gstack-upgrade (and more).
```

**Run your first sprint:**

1. `/office-hours` — Describe your feature idea. The AI reframes it into a high-value product strategy using YC-style forcing questions.
2. `/plan-ceo-review` — Founder-mode product review to find the "10-star product" in your request.
3. `/plan-eng-review` — Engineering Manager locks architecture with ASCII flowcharts, data flows, and failure modes.
4. Write the code.
5. `/review` — Paranoid structural audit for production bugs.
6. `/qa` — Automated browser-based visual verification of your changes.
7. `/ship` — Syncs branches, runs tests, opens a PR.

---

## Sources

### Web
- [garrytan/gstack GitHub repo](https://github.com/garrytan/gstack)
- [GStack — Turn Claude Code into a Virtual Software Development Team](https://gstacks.org/)
- [gstack – Garry Tan's Claude Code Setup | Hacker News](https://news.ycombinator.com/item?id=47355173)
- [gstack/docs/skills.md](https://github.com/garrytan/gstack/blob/main/docs/skills.md)

### YouTube
- [The New Way To Build A Startup](https://www.youtube.com/watch?v=rWUWfj_PqmM)
- [gstack - YC CEO Garry Tan의 뇌를 빌려 딸깍! Claude Code skills](https://www.youtube.com/watch?v=vfn_Ezu1qfk)
- [Day in the Life of Y Combinator President & CEO Garry Tan](https://www.youtube.com/watch?v=GctjcD17iI4)
