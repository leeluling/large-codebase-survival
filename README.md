# Large Codebase Survival Guide

> A battle-tested methodology for working with large codebases (15K+ lines) using AI coding agents — without losing your session to context overflow.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## The Problem

AI coding agents (Claude Code, Cursor, Windsurf, etc.) are incredibly powerful — until they aren't. When working with large files or complex multi-step tasks, the agent's context window fills up and the session **dies permanently** (HTTP 503). No recovery. No `/clear`. All unsaved progress — gone.

This guide was born from **months of painful experience** migrating a 15,000+ line Python application from SimpleGUI to WebView using Claude Code. Countless sessions were lost to 503 errors before a reliable methodology emerged.

**The result:** a structured workflow called **Survival Mode** that turned a project from "impossible — keeps crashing" into "slow but steady progress, every session."

## TL;DR

| Step | What to do |
|------|-----------|
| 1. Create progress tracker | Create `PROGRESS.md` to track all tasks and status |
| 2. Analyze via subagent | Never read the large file directly — send a subagent to summarize its structure |
| 3. Decompose | Break the task into small units (one function/component each) |
| 4. Execute loop | For each unit: subagent reads → subagent implements → verify → git commit → update progress |
| 5. Monitor context health | Every 3-4 tasks, check if the session is getting heavy — save and restart if needed |

## Who Is This For?

- Developers using **Claude Code**, **Cursor**, **Windsurf**, or similar AI coding agents
- Anyone working with **large files** (5K+ lines) or **complex multi-step refactors**
- Teams that keep losing AI coding sessions to context overflow / 503 errors
- Anyone who wants to use AI agents on real-world, messy, large codebases — not just toy projects

## Quick Start

**Option 1: Add to your project's `CLAUDE.md`** (for Claude Code users)

Copy [`templates/CLAUDE_SNIPPET.md`](templates/CLAUDE_SNIPPET.md) into your project's `CLAUDE.md` file. Claude Code will follow these rules automatically.

**Option 2: Use the Survival Mode skill**

Copy [`survival_mode_skill.md`](survival_mode_skill.md) into your Claude Code skills directory. It activates when you hit 503 errors or ask for "survival mode."

**Option 3: Read and apply manually**

Read the [full methodology](#methodology) below and apply when prompting your AI coding agent.

---

## Methodology

### The Core Idea

Treat the AI agent as a **project manager**, not a developer. The main agent should **orchestrate** — it never reads large files or generates large outputs directly. Instead, it delegates focused tasks to **subagents** (disposable workers). If a worker crashes, the manager survives. Progress is saved after every small win.

### Step 1: Create a Progress Tracker

Before starting any work, create a `PROGRESS.md` file in your project directory. This is the lifeline that lets any new session resume from where the last one left off.

See [`templates/PROGRESS_TRACKER.md`](templates/PROGRESS_TRACKER.md) for a ready-to-use template.

### Step 2: Analyze the Target via Subagent

Never read a large file directly. Send a subagent to analyze its structure:

```
Subagent task: "Read the first 50 and last 50 lines of main.py.
Use Grep to find all class and function definitions.
Output a structural summary in under 200 words."
```

### Step 3: Decompose into Small Tasks

Based on the structural summary, break the work into the smallest independently completable units. Each unit should be:
- No more than one function or one component
- Achievable in a single subagent call
- Independently testable and committable

Write all tasks into `PROGRESS.md`.

### Step 4: Execute the Loop

For each small task, follow this cycle:

```
Read (subagent) → Implement (subagent) → Verify (main agent) → Commit → Update progress
```

**Read:** Subagent reads only the specific line range needed (e.g., lines 200-280), summarizes in under 100 words.

**Implement:** Subagent writes the new code directly to the file using edit tools. Never output full code into the conversation.

**Verify:** Main agent runs a quick sanity check (syntax, import test).

**Commit:** `git commit` immediately with a descriptive message.

**Update:** Mark the task as done in `PROGRESS.md`.

### Step 5: Monitor Context Health

After every 3-4 completed tasks:
- If the conversation feels heavy → save all progress, suggest starting a new session
- If subagents are slowing down or failing → reduce each subagent's scope

---

## Hard Rules

These are non-negotiable when in Survival Mode:

| Rule | Limit |
|------|-------|
| Main agent file reading | Never read more than 300 lines — must use subagent for larger reads |
| Subagent output | Every subagent prompt must include "under 200 words" or similar limit |
| Save frequency | Must git commit + update PROGRESS.md after every completed task |
| Task granularity | Each task should be no more than one function or one component |
| File reading | Must use offset/limit to specify line ranges — never read an entire large file |
| Conversation output | Main agent keeps responses concise — no unnecessary summaries |

## Example Session

```
User: "Help me migrate main.py's UI from SimpleGUI to WebView. The file is 15000 lines."

1. Create PROGRESS.md
2. Subagent → scan main.py structure, return summary (under 200 words)
3. Decompose into 12 small tasks, write to PROGRESS.md
4. Task 1: subagent reads lines 200-280 → subagent rewrites → verify → commit
5. Task 2: subagent reads lines 450-520 → subagent rewrites → verify → commit
6. ...
7. Check context health every 3-4 tasks
8. If session is getting full → save progress → start new session to continue
```

Output after each task:
```
[Survival Mode] Completed 4/12 tasks
✓ Main window layout (commit a1b2c3)
✓ Settings dialog (commit d4e5f6)
✓ File browser (commit g7h8i9)
✓ Event bindings (commit j0k1l2)
→ Next: Status bar component

Progress saved to PROGRESS.md
```

## Known Limitations

1. **Speed vs. reliability trade-off** — progress is slower per session, but cumulative progress is much faster than losing sessions to 503
2. **Holistic understanding** — some tasks require seeing a large file as a whole; have a subagent produce a summary/outline first, then work from the outline
3. **Subagent context limits** — subagents have their own limits; keep individual tasks focused
4. **Agent-specific** — optimized for Claude Code, but the principles apply to any AI coding agent with context limits

## Contributing

Found this useful? Have improvements to suggest?

- Open an issue or pull request
- Share your own war stories and patterns
- Star this repo if it saved you from a 503

## Origin Story

This methodology was developed while migrating a production desktop application (I2D client) from SimpleGUI to WebView. The codebase: a single 15,000+ line Python file with complex business logic, UI rendering, device communication, and state management — all intertwined.

Every AI coding session crashed. Multiple agents, multiple days, zero progress. The 503 errors weren't occasional — they were inevitable. The context window simply couldn't hold a file that large.

The breakthrough came from treating the AI agent not as a developer who reads the whole codebase, but as a **project manager** who delegates focused tasks to disposable workers (subagents). Workers can crash. The manager survives. Progress is saved. The next worker picks up where the last one left off.

What was once impossible became routine — slow, methodical, but reliably forward.

---

*Built with hard-won experience and many lost sessions.*
