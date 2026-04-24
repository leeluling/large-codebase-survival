# Large Codebase Survival Guide

> A battle-tested methodology for working with large codebases (15K+ lines) using AI coding agents — without losing your session to context overflow.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## The Problem

AI coding agents (Claude Code, Cursor, Windsurf, etc.) are incredibly powerful — until they aren't. When working with large files or complex multi-step tasks, the agent's context window fills up and the session **dies permanently** (HTTP 503). No recovery. No `/clear`. All unsaved progress — gone.

This guide was born from **months of painful experience** migrating a 15,000+ line Python application from SimpleGUI to WebView using Claude Code. Countless sessions were lost to 503 errors before a reliable methodology emerged.

**The result:** a set of 6 principles that turned a project from "impossible — keeps crashing" into "slow but steady progress, every session."

## TL;DR

| Principle | One-liner |
|-----------|-----------|
| 1. Protect the Main Agent | Delegate heavy work to subagents — if they die, you survive |
| 2. Save Early, Save Often | Commit after every unit of work — assume the session can die anytime |
| 3. Minimize Context Consumption | Only read what you need, keep output concise |
| 4. Break Big Tasks into Small Pieces | Decompose until each piece fits in one agent call |
| 5. Reserve Context Buffer | Don't push to the limit — leave room for recovery |
| 6. Design for Session Continuity | A new session should pick up exactly where the last one left off |

## Who Is This For?

- Developers using **Claude Code**, **Cursor**, **Windsurf**, or similar AI coding agents
- Anyone working with **large files** (5K+ lines) or **complex multi-step refactors**
- Teams that keep losing AI coding sessions to context overflow / 503 errors
- Anyone who wants to use AI agents on real-world, messy, large codebases — not just toy projects

## Quick Start

**Option 1: Add to your project's `CLAUDE.md`** (for Claude Code users)

Copy [`templates/CLAUDE_SNIPPET.md`](templates/CLAUDE_SNIPPET.md) into your project's `CLAUDE.md` file. Claude Code will follow these rules automatically.

**Option 2: Read and apply manually**

Read the [full methodology](#methodology) below and apply the principles when prompting your AI coding agent.

---

## Methodology

### Principle 1: Protect the Main Agent

The main agent is your session's lifeline. It should **orchestrate**, not do heavy lifting.

**Rules:**
- Never read large files (500+ lines) directly in the main agent — delegate to a subagent
- Never generate large code blocks in the main agent — have a subagent write to files
- If a subagent dies from 503, the main agent survives and can spawn a new one
- Think of the main agent as a **project manager**, not a developer

**How to apply (Claude Code):**
```
Use the Agent tool for:
- Reading and analyzing large files
- Writing or rewriting large code sections
- Researching across multiple files
- Any task that might produce verbose output
```

### Principle 2: Save Early, Save Often

Assume every session can die at any moment. The only safe progress is **persisted progress**.

**Rules:**
- Commit or write to file after every completed unit of work (one function, one module, one component)
- After each milestone, update a progress file or memory so a new session can resume
- Never hold important decisions or code only in conversation context
- Write a brief handoff note after each major step

**How to apply:**
```
After completing each piece:
1. Write/edit the target file immediately
2. Run a quick sanity check (syntax, import, basic test)
3. Git commit with a descriptive message
4. Update a progress tracker (markdown file, memory, or TODO)
```

### Principle 3: Minimize Context Consumption

Every token in context is a finite resource. Spend it wisely.

**Rules:**
- Don't read files you don't need — know exactly what you're looking for before reading
- Don't scan or explore broadly when the target is known
- Keep responses concise — no unnecessary explanations or summaries
- When reading a large file, read only the specific line range needed (use offset/limit)
- Tell subagents to limit their output (e.g. "report in under 200 words")

**Anti-patterns to avoid:**
- Reading an entire 15K-line file "to understand the structure"
- Searching broadly with `Glob **/*` when you know the filename
- Generating verbose explanations after every code change
- Reading documentation files that aren't directly relevant

### Principle 4: Break Big Tasks into Small Pieces

A 15K+ line file rewrite is impossible in one session. A single function rewrite is easy.

**Rules:**
- Decompose the task into the smallest independently completable units
- Each unit should be achievable within a single subagent call
- Work on one unit at a time, save, then move to the next
- Create a task list upfront and check items off as you go

**Example decomposition:**
```
BAD:  "Migrate the entire UI from SimpleGUI to WebView"

GOOD: 
  - [ ] Pass 1: Extract and list all UI components to migrate
  - [ ] Pass 2: Migrate the main window layout
  - [ ] Pass 3: Migrate the settings dialog
  - [ ] Pass 4: Migrate the file browser component
  - [ ] Pass 5: Wire up event handlers
  - [ ] Pass 6: Integration test
```

### Principle 5: Reserve Context Buffer

Don't push the context to its limit. Leave room for error handling and course correction.

**Rules:**
- If you feel the session is getting heavy, proactively save progress and consider starting fresh
- Limit subagent output length explicitly in prompts
- Avoid loading multiple large files in the same subagent
- When a subagent returns, extract only the essential information — don't relay the full output

### Principle 6: Design for Session Continuity

Any session can die. A new session should be able to pick up exactly where the last one left off.

**Rules:**
- Maintain a progress file that tracks: what's done, what's next, known issues
- Use descriptive git commits so `git log` tells the story
- Store key decisions and context in persistent memory, not just conversation
- At the start of a new session, read the progress file before doing anything else

See [`templates/PROGRESS_TRACKER.md`](templates/PROGRESS_TRACKER.md) for a ready-to-use template.

---

## Quick Reference Checklist

Before each action, ask yourself:

| Question | If Yes |
|----------|--------|
| Will this read a large file? | Use a subagent |
| Will this generate long output? | Use a subagent, limit output |
| Did I just finish a unit of work? | Save, commit, update progress |
| Am I reading something I don't need? | Stop. Be targeted. |
| Is the task too big for one pass? | Break it down further |
| Could a new session resume from here? | If not, save context now |

## Example Session Flow

```
Main agent receives: "Migrate function X from old UI to new UI"

1. Main agent creates task list
2. Main agent spawns subagent:
   "Read lines 200-350 of main.py, summarize function X's logic in under 100 words"
3. Subagent returns summary
4. Main agent spawns subagent:
   "Write the new implementation of function X to /tmp/new_func_x.py based on this spec: [summary]"
5. Subagent writes file
6. Main agent integrates the file, runs quick test
7. Main agent commits: "migrate function X to webview"
8. Main agent updates progress tracker
9. Move to next task
```

## Known Limitations

1. **Speed vs. reliability trade-off** — progress is slower per session, but cumulative progress is much faster than losing sessions to 503
2. **Subagent context limits** — subagents have their own limits; keep individual tasks focused
3. **Holistic understanding** — some tasks require seeing a large file as a whole; have a subagent produce a summary/outline first, then work from the outline
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
