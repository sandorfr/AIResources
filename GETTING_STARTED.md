# Getting Started

This repo is a Claude Code-powered research pipeline. Add topics you want to learn about to `QUEUE.md`, run `/dequeue-topic`, and get a concise research document with infographic, audio podcast, and video — automatically.

## Prerequisites

### Required

- **Claude Code** — the CLI that runs the `/dequeue-topic` skill
- **NotebookLM CLI** — installed at `~/.local/bin/notebooklm`, handles notebook creation, source ingestion, analysis, and artifact generation. Run `notebooklm --help` to verify.
- **yt-dlp** — for supplemental YouTube search. Install: `brew install yt-dlp`
- **ffmpeg** — for converting podcast audio to video. Install: `brew install ffmpeg`

### Optional (YouTube publishing)

To auto-publish videos to a dedicated YouTube channel:

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a new project
2. Enable the **YouTube Data API v3**
3. Create **OAuth 2.0 credentials** (Desktop application) and download `client_secrets.json`
4. Install the uploader:
   ```bash
   pip3 install google-api-python-client youtube-upload
   ```
5. Store credentials:
   ```bash
   mkdir -p ~/.config/youtube-upload
   mv client_secrets.json ~/.config/youtube-upload/
   ```
6. Run one manual upload to complete the browser OAuth flow:
   ```bash
   youtube-upload --client-secrets=~/.config/youtube-upload/client_secrets.json \
     --title="test" some_video.mp4
   ```
   After authenticating, credentials are cached for future uploads.

If this isn't set up, `/dequeue-topic` will skip the YouTube step gracefully.

## How it works

### 1. Add topics to the queue

Edit `QUEUE.md` and add topics under `## Queue`:

```markdown
## Queue

### Topic Name

Short description of why this is interesting.
https://github.com/example/repo
https://some-blog.com/article-about-topic

#### Resources
https://www.youtube.com/watch?v=example1
https://www.youtube.com/watch?v=example2
```

Each topic needs:
- A `### ` heading (the topic name)
- A short description (1-2 sentences)
- One or more URLs (web and/or YouTube) — these are seed sources

The `#### Resources` sub-heading is optional and purely organizational.

### 2. Run the dequeue command

```
/dequeue-topic
```

This processes the **first** topic in the queue. The full pipeline:

1. **Parse** — extracts the topic, description, and URLs from `QUEUE.md`
2. **Research** — supplements seed URLs with web search and YouTube search (targets 5-15 sources)
3. **Analyze** — creates a NotebookLM notebook, ingests all sources, asks 4 structured questions:
   - What is it?
   - When to use it?
   - When NOT to use it / limitations?
   - How to get started? (hands-on walkthrough)
4. **Write** — produces a concise research document in `research/`
5. **Enrich** — generates infographic (PNG), audio podcast (MP3), and video (MP4) via NotebookLM
6. **Publish** — uploads videos to YouTube as unlisted (if configured)
7. **Update** — moves the topic from Queue to Done in `QUEUE.md`

### 3. Interrupt and resume

If the process is interrupted (network issue, timeout, you stop it), state is saved in `INPROGRESS.md`. Running `/dequeue-topic` again picks up exactly where it left off.

### 4. Repeat

Run `/dequeue-topic` again to process the next topic. Keep going until the queue is empty.

## Output structure

```
research/
  2026-03-29-autoresearch.md          # research document
  2026-03-29-openspace.md
  deliverables/
    2026-03-29-autoresearch-infographic.png
    2026-03-29-autoresearch-podcast.mp3
    2026-03-29-autoresearch-video.mp4
    2026-03-29-autoresearch-podcast-video.mp4   # podcast + infographic as video
```

Each research document contains:
- Frontmatter (date, topic, sources count, notebook ID, tags)
- Embedded infographic and media links
- **What is it?** — clear definition and explanation
- **When to use it** — ideal use cases
- **When NOT to use it / Limitations** — honest assessment with alternatives
- **Getting Started** — install commands, setup, first working example
- **Sources** — all web and YouTube sources used

## Files in this repo

| File | Purpose |
|------|---------|
| `QUEUE.md` | Topics to research — add new topics here |
| `INPROGRESS.md` | Transient state file (only exists during processing) |
| `research/*.md` | Generated research documents |
| `research/deliverables/` | Infographics, podcasts, videos |
| `.claude/commands/dequeue-topic.md` | The skill definition |
