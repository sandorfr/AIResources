---
name: dequeue-topic
description: >
  Process the next topic from the QUEUE.md ingestion stream. Dequeues the first
  topic, researches it using web search and YouTube, creates a NotebookLM notebook
  with all sources, runs structured analysis, generates a concise research document
  with enrichments (infographic, audio podcast), and updates the queue. Supports
  interrupt/resume via INPROGRESS.md state tracking.
  Trigger on: "dequeue", "next topic", "process queue", "dequeue-topic", or
  "ingestion stream".
---

# Dequeue Topic

Process the next topic from the learning queue, research it, and produce a concise reference document with rich media enrichments.

**Queue file:** `QUEUE.md` (project root)
**State file:** `INPROGRESS.md` (project root)
**Research output:** `research/` (project root)
**Deliverables:** `research/deliverables/` (project root)

---

## NotebookLM CLI Reference

The `notebooklm` CLI is at `~/.local/bin/notebooklm`. All commands below use the correct, tested syntax.

### Notebook management
```bash
notebooklm create "title"                    # returns notebook ID
notebooklm use <id>                           # set active (supports prefix match)
notebooklm status                             # show active notebook info
notebooklm list                               # list all notebooks
```

### Sources — type is AUTO-DETECTED
```bash
notebooklm source add <url>                   # web URL or YouTube — auto-detected
notebooklm source add ./file.md               # local file
notebooklm source add "text content"          # inline text
notebooklm source wait <source-id> --timeout 120   # wait for ONE source by ID
notebooklm source list                        # list sources in active notebook
```

**Important:** `source add` takes content as a positional argument. Do NOT use `--url` or `--youtube` flags — they don't exist. Type is auto-detected from the content.

**Important:** `source wait` requires a `<source-id>` argument. It does not wait for all sources at once.

### Analysis
```bash
notebooklm ask "question"                     # query notebook content
```

### Artifact generation
```bash
notebooklm generate <type>                    # types: infographic, audio, slide-deck, mind-map, report
notebooklm artifact wait <artifact-id> --timeout 300   # wait for generation (can take 1-5 min)
notebooklm download <type> "/path/to/output"  # download to local path
```

**Important:** `artifact wait` requires an `<artifact-id>` argument (returned by `generate`).

---

## Step 0 — Check for in-progress work

Before doing anything, check if `INPROGRESS.md` exists in the project directory and contains an active topic.

**If INPROGRESS.md exists and has content:**
- Read it to recover state: topic name, slug, date, notebook ID, and which steps are completed.
- Resume from the first unchecked step (see the checklist format below).
- Skip to that step in this workflow.

**If INPROGRESS.md does not exist or is empty:**
- Proceed to Step 1 (fresh dequeue).

### INPROGRESS.md format

```markdown
---
topic: <Topic Name>
slug: <topic-slug>
date: <YYYY-MM-DD>
notebook_id: <notebook-id or empty>
---

## Progress

- [ ] Parsed from QUEUE.md
- [ ] Sources supplemented
- [ ] NotebookLM notebook created and sources indexed
- [ ] Analysis complete (4 questions)
- [ ] Research document written
- [ ] Infographic generated
- [ ] Audio podcast generated
- [ ] QUEUE.md updated (moved to Done)
```

**After completing each step**, update INPROGRESS.md immediately:
- Mark the step `[x]`
- Update any new state values in the frontmatter (e.g., `notebook_id` after Step 3)

This ensures that if the process is interrupted at any point, the next invocation can resume cleanly.

---

## Step 1 — Parse QUEUE.md

Read `QUEUE.md` in the project root.

- Find the `## Queue` section.
- Extract the **first** `### ` heading after it — this is the **topic name**.
- Collect all lines between that `### ` and the next `### ` or `## ` heading (or end of file).
- From those lines, separate:
  - **Description**: Non-empty lines that are not URLs and not `####` headings.
  - **Web URLs**: Lines starting with `https://` that do NOT contain `youtube.com/watch` or `youtu.be`.
  - **YouTube URLs**: Lines starting with `https://` that DO contain `youtube.com/watch` or `youtu.be`.
- Compute the **topic slug**: lowercase, replace spaces/special chars with hyphens, strip leading/trailing hyphens.
- Use **today's date** as `YYYY-MM-DD`.

**If no `### ` heading is found under `## Queue`:** report "Queue is empty — nothing to process." and stop.

**Then:** Create INPROGRESS.md with the topic metadata and mark this step `[x]`. Remove the topic block from QUEUE.md at this point (it now lives in INPROGRESS.md).

---

## Step 2 — Supplement sources via search

The queue entry provides seed URLs. Supplement them to reach 5–15 total sources.

1. Use `WebSearch` with the topic name to find official docs, blog posts, tutorials. Extract up to 5 relevant URLs not already in the seed set.
2. Run YouTube search for additional videos:
   ```bash
   yt-dlp "ytsearch5:<topic name>" \
     --print "%(title)s | %(uploader)s | %(view_count)s views | %(duration_string)s | %(webpage_url)s" \
     --no-download --quiet
   ```
3. From YouTube results, pick up to 3 new URLs not already in the seed set, preferring high view counts.
4. Deduplicate. Compile the **final source list** (web URLs + YouTube URLs). If over 15, trim lowest-quality supplemental sources.

Report: "Proceeding with N sources (X from queue, Y supplemental web, Z supplemental YouTube)."

Mark step `[x]` in INPROGRESS.md.

---

## Step 3 — Create NotebookLM notebook & add sources

```bash
# Ensure output directories exist
mkdir -p research/deliverables

# Create notebook
notebooklm create "[YYYY-MM-DD] <Topic Name>"

# Use the newly created notebook (capture ID from create output)
notebooklm use <notebook-id>

# Add each source (URLs are auto-detected as web or YouTube)
notebooklm source add "<url>"    # repeat for each URL

# Wait for each source to be indexed (one at a time, by source ID)
notebooklm source wait <source-id> --timeout 120    # repeat for each source ID

# Verify
notebooklm source list
```

- If any individual `source add` fails (paywalled, inaccessible), log it and continue.
- After waiting, confirm at least 2 sources were indexed. If fewer, abort with error.
- **Update INPROGRESS.md**: set `notebook_id` in frontmatter, mark step `[x]`.

---

## Step 4 — Structured analysis (4 questions)

Run four `notebooklm ask` queries sequentially and capture each response:

```bash
notebooklm ask "What is <Topic Name>? Give a clear, concise definition and explanation of what it does, how it works, and what makes it noteworthy."

notebooklm ask "When should you use <Topic Name>? What problems does it solve? What are the ideal use cases and who benefits most from it?"

notebooklm ask "When should you NOT use <Topic Name>? What are its limitations, drawbacks, common pitfalls, and what alternatives exist?"

notebooklm ask "How do you get started with <Topic Name>? Give a pragmatic, hands-on walkthrough: installation, setup, first working example. Include actual commands and code snippets."
```

Save each answer for use in the research document.

Mark step `[x]` in INPROGRESS.md.

---

## Step 5 — Write research document

Get the notebook info:
```bash
notebooklm status
```

Write the file to `research/YYYY-MM-DD-<slug>.md` (relative to project root):

```markdown
---
date: YYYY-MM-DD
topic: <Topic Name>
sources: <count of indexed sources>
notebook_id: <notebook ID from notebooklm status>
tags: [<2-5 relevant tags>]
---

# <Topic Name>

<Description from QUEUE.md>

![<Topic Name> Infographic](deliverables/YYYY-MM-DD-<slug>-infographic.png)

[Listen to the audio overview](deliverables/YYYY-MM-DD-<slug>-podcast.mp3)

---

## What is it?

<Synthesized from question 1 — rewrite for clarity, no filler, no NotebookLM citation markers>

---

## When to use it

<Synthesized from question 2 — bullet points or short paragraphs>

---

## When NOT to use it / Limitations

<Synthesized from question 3 — honest, practical, includes alternatives>

---

## Getting Started

<Synthesized from question 4 — pragmatic and hands-on: install commands, minimal config, first working example with real commands/code. Reader should go from zero to running in minutes.>

---

## Sources

### Web
- [Domain or title](url)

### YouTube
- [Video title](url)
```

**Guidelines:**
- Rewrite NotebookLM answers into clean, direct prose. Remove hedging, filler, citation markers.
- Each section: 100–300 words. The whole doc should be scannable in under 5 minutes.
- Getting Started must include actual runnable commands and code.
- Derive 2–5 tags from the topic domain.

Mark step `[x]` in INPROGRESS.md.

---

## Step 6 — Generate enrichments

### Infographic
```bash
notebooklm generate infographic
# capture the artifact ID from output
notebooklm artifact wait <artifact-id> --timeout 300
notebooklm download infographic "research/deliverables/YYYY-MM-DD-<slug>-infographic.png"
```

Mark "Infographic generated" `[x]` in INPROGRESS.md.

### Audio podcast
```bash
notebooklm generate audio
# capture the artifact ID from output
notebooklm artifact wait <artifact-id> --timeout 300
notebooklm download audio "research/deliverables/YYYY-MM-DD-<slug>-podcast.mp3"
```

Mark "Audio podcast generated" `[x]` in INPROGRESS.md.

If either generation fails, log the error and continue. Report partial results.

---

## Step 7 — Update QUEUE.md (move to Done)

Re-read `QUEUE.md` (it may have changed).

1. Verify the topic block was already removed in Step 1. If not (e.g., resuming an older run), remove it now.
2. If a `## Done` section exists, append the entry at its end.
3. If `## Done` does not exist, add it after the `## Queue` section content:
   ```markdown
   ## Done

   - [YYYY-MM-DD] **<Topic Name>** — [research doc](research/YYYY-MM-DD-<slug>.md)
   ```

Mark step `[x]` in INPROGRESS.md.

---

## Step 8 — Clean up & report

**Delete INPROGRESS.md** (all steps complete, state no longer needed).

Present to the user:

```
## Dequeue complete: <Topic Name>

### Key findings
- <3-5 bullet points with the most important takeaways>

### Generated files
- Research doc: research/YYYY-MM-DD-<slug>.md
- Infographic: research/deliverables/YYYY-MM-DD-<slug>-infographic.png
- Podcast: research/deliverables/YYYY-MM-DD-<slug>-podcast.mp3

### NotebookLM
- Notebook ID: <id>
- Sources indexed: <N>

### Queue status
- Removed "<Topic Name>" from queue
- Remaining topics: <count>
```

---

## Error handling

- **Empty queue**: Report "Queue is empty — nothing to process." and stop.
- **Source add failures**: Skip failed sources (paywalled, inaccessible), continue. Minimum 2 sources required; if fewer succeed, abort and report.
- **NotebookLM create/use failure**: Abort and report. Suggest checking `notebooklm list` and authentication.
- **Deliverable generation failure**: Continue with remaining deliverables. Report what succeeded and what failed.
- **QUEUE.md conflict**: Re-read before modifying. If the topic is already gone, report "Topic already processed" and stop.
- **Resume after interrupt**: INPROGRESS.md preserves full state. Next invocation picks up from the first unchecked step automatically.

---

## Tips

- Run `/dequeue-topic` repeatedly to drain the queue one topic at a time.
- The INPROGRESS.md file is transient — it only exists while a topic is being processed.
- All deliverables use the `YYYY-MM-DD-<slug>` naming convention for easy cross-referencing.
- The notebook remains in NotebookLM after processing for further exploration if desired.
