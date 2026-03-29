---
date: 2026-03-29
topic: AutoResearch
sources: 9
notebook_id: 151decfe-b4b1-4f34-a2e7-9f3aab38e3c1
tags: [ai, research-automation, self-improvement, karpathy, ml-experiments]
---

# AutoResearch

Interesting approach to AI self-improving based on quantitative and objective metrics. Created by Andrej Karpathy, released March 2026.

![AutoResearch Infographic](deliverables/2026-03-29-autoresearch-infographic.png)

[Listen to the audio overview](deliverables/2026-03-29-autoresearch-podcast.mp3)

---

## What is it?

AutoResearch is an open-source Python tool that lets AI agents conduct machine learning research autonomously on a single GPU. It runs experiments in a continuous loop overnight without human intervention, keeping code changes that improve a target metric and reverting those that don't.

The system uses a strict **three-file architecture**:

- **`program.md`** (human-owned): Research directions, goals, and constraints written in plain English. This is the only file the human edits.
- **`train.py`** (agent-owned): The model architecture and training loop. The AI agent freely modifies this code — architecture, hyperparameters, optimization strategies.
- **`prepare.py`** (immutable): Data preparation and the validation metric (`val_bpb`). Neither human nor agent can touch this, preventing "cheating."

The core mechanism is the **ratchet loop**: the agent proposes a change, trains for a fixed 5-minute budget, checks if `val_bpb` improved, and either commits (if better) or reverts via `git reset` (if not). This cycle typically runs 80-100+ experiments overnight.

What makes it noteworthy: unlike traditional AutoML (Optuna, Ray Tune) that searches predefined parameter grids, AutoResearch lets agents **rewrite arbitrary source code** — enabling architectural discoveries, not just hyperparameter tuning. In initial runs, it found 20 optimizations that reduced Karpathy's "Time to GPT-2" by 11%. Shopify's CEO ran 37 experiments overnight and saw a 19% improvement.

---

## When to use it

- **You have an objective, automated scoring metric.** The system needs a clear numerical target — validation loss, latency, throughput, test pass rate.
- **Experiments run fast.** The ratchet loop is designed for 5-minute trials. If your feedback loop is short, you can test 100+ hypotheses overnight.
- **You want to squeeze incremental improvements from a known pipeline.** Ideal for refining model architectures, attention mechanisms, optimizer configs.
- **You're a small team or solo researcher with more hypotheses than time.** One person + one GPU can systematically test what would take a team weeks.

**Use cases beyond ML training:**
- Software performance engineering (rewriting code/CUDA kernels for speed)
- Marketing AB tests (landing pages, ad creatives, email subject lines)
- Automated test generation (Playwright test suites reaching 100% coverage)
- Financial backtesting (optimizing trading strategies overnight)

**Who benefits most:** small teams, startups, individual researchers, and "directors" who can define goals in English but don't want to manually iterate on code.

---

## When NOT to use it / Limitations

**Don't use it when:**
- **The task is subjective** — brand design, UX/UI, creative writing. No objective score means no ratchet.
- **Experiments are slow** — if a single run takes hours/days, the high-throughput loop loses its advantage.
- **You need breakthrough innovation** — the ratchet only accepts immediate improvements. It cannot take a temporary step backward for a larger long-term gain ("creativity ceiling").
- **You're not on NVIDIA** — requires a single NVIDIA GPU with 20GB+ VRAM by default. Community forks exist for Mac/AMD but are unofficial.

**Key limitations:**
- **Local optima trapping:** The agent cycles through minor variations of what worked, rarely proposing radical shifts.
- **Overfitting risk:** 100+ experiments against one static validation set can overfit to that benchmark rather than genuinely generalizing.
- **Codebase bloat:** If `train.py` grows beyond ~630 lines, the agent may lose coherence as it exceeds its context window.
- **RLHF conservatism:** Underlying LLMs (Claude, GPT) are trained to be safe and cautious, which can make them timid experimenters.
- **Gameable metrics:** A bad metric means the agent confidently optimizes for the wrong thing.

**Alternatives:**
- **Optuna / Ray Tune / Auto-Keras** — traditional AutoML for predefined hyperparameter search
- **AlphaEvolve** — Google DeepMind's evolutionary algorithm discovery (closed source)
- **MLJAR AutoLab** — structured alternative with Jupyter notebook transparency
- **General coding agents** (Aider, SWE-Agent, Claude Code) — can write code but lack the built-in experiment-evaluate-revert loop

---

## Getting Started

### Prerequisites
- NVIDIA GPU with 20GB+ VRAM (tested on H100; consumer cards work with reduced model size)
- Python 3.10+
- `uv` (fast Python package manager)
- A coding agent: Claude Code, Cursor, or similar

### Install and prepare

```bash
git clone https://github.com/karpathy/autoresearch
cd autoresearch
uv sync
python prepare.py
```

### Verify your setup

```bash
python train.py
```

If this runs without OOM errors, you're ready. On smaller GPUs, scale down first: set `DEPTH=4`, `vocab_size=256`, `MAX_SEQ_LEN=256`, and switch to TinyStories dataset.

### Launch the autonomous loop

Open your coding agent in the repo root and prompt:

> "Please read program.md and recent results in results.tsv. Based on the instructions, start the autonomous research loop. Run experiments, evaluate them, and commit improvements to the git history. Do not stop to ask for my permission."

### What happens next

1. Agent proposes a change to `train.py`
2. Runs training for 5 minutes
3. Compares new `val_bpb` to previous best
4. Commits if improved, reverts if not
5. Repeats indefinitely

### Monitor progress

- **`results.tsv`** — log of every experiment (including failures)
- **`git log`** — clean history of successful improvements
- **`progress.png`** — staircase visualization of score improvement

Expect 80-100 experiments overnight, typically yielding 15-20 kept improvements.

---

## Sources

### Web
- [GitHub - karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- [DataScienceDojo - Karpathy Autoresearch Explained](https://datasciencedojo.com/blog/karpathy-autoresearch-explained/)
- [DataCamp - Guide to AutoResearch](https://www.datacamp.com/tutorial/guide-to-autoresearch)
- [VentureBeat - Karpathy's AutoResearch](https://venturebeat.com/technology/andrej-karpathys-new-open-source-autoresearch-lets-you-run-hundreds-of-ai)
- [MLJAR - AutoResearch and the Future of Autonomous AI Research](https://mljar.com/blog/autoresearch-karpathy-autonomous-ai-research/)

### YouTube
- [I Built Karpathy's AutoResearch for Playwright Testing](https://www.youtube.com/watch?v=VxMHTk627tE)
- [Karpathy's "autoresearch" broke the internet](https://www.youtube.com/watch?v=qb90PPbAWz4)
- [Skill Issue: Andrej Karpathy on Code Agents, AutoResearch, and the Loopy Era of AI](https://www.youtube.com/watch?v=kwSVtQ7dziU)
- [The only AutoResearch tutorial you'll ever need](https://www.youtube.com/watch?v=uBWuKh1nZ2Y)
