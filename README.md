# Large Codebase Survival Guide

Working with large files (15K+ lines) or complex multi-step tasks using AI coding agents (Claude Code, Cursor, etc.) often leads to context overflow (503 errors, session death). This skill encodes a battle-tested methodology to make steady progress without losing work.

## Trigger Conditions
- Task involves reading or modifying files over 5K lines
- Task requires multiple rounds of changes across a large codebase
- User mentions 503 concerns, context limits, or session stability
- Any multi-session project (e.g. UI migration, major refactor)

## The Core Problem

AI coding agents have a finite context window. When it overflows:
- The session dies permanently (503 error)
- No recovery is possible — not even `/clear` helps
- All unsaved progress in that session is lost

The overflow is caused by: reading large files into main context, generating long outputs, accumulating tool results, or exploring too many files.

## Prerequisites
- Git (for incremental commits)
- A file-based memory/notes system (Claude Code memory, markdown files, etc.)
- Ability to spawn subagents or child processes

---

## Methodology

### Principle 1: Protect the Main Agent

The main agent is your session's lifeline. It should **orchestrate**, not do heavy lifting.

**Rules:**
- Never read large files (500+ lines) directly in the main agent — delegate to a subagent
- Never generate large code blocks in the main agent — have a subagent write to files
- If a subagent dies from 503, the main agent survives and can spawn a new one
- Think of the main agent as a project manager, not a developer

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

A 15K-line file rewrite is impossible in one session. A single function rewrite is easy.

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

**Progress file template:**
```markdown
# Project: [Name]
## Status: [In Progress / Blocked / Done]
## Last updated: [date]

### Completed
- [x] Component A migrated (commit abc123)
- [x] Component B migrated (commit def456)

### In Progress
- [ ] Component C — started, event handlers remaining

### Known Issues
- Issue X: workaround is Y
- File Z has a quirk at line N

### Key Decisions
- Chose approach A over B because ...
```

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

## Known Limitations

1. This methodology trades speed for reliability — progress is slower per session, but cumulative progress is much faster than losing sessions to 503
2. Subagents have their own context limits — keep individual subagent tasks focused
3. Some tasks genuinely require seeing a large file holistically — in those cases, have a subagent produce a summary/outline first, then work from the outline
4. This is optimized for Claude Code but the principles apply to any AI coding agent with context limits

## Example Session Flow

```
Main agent receives: "Migrate function X from old UI to new UI"

1. Main agent creates task list
2. Main agent spawns subagent: "Read lines 200-350 of main.py, summarize function X's logic in under 100 words"
3. Subagent returns summary
4. Main agent spawns subagent: "Write the new implementation of function X to /tmp/new_func_x.py based on this spec: [summary]"
5. Subagent writes file
6. Main agent integrates the file, runs quick test
7. Main agent commits: "migrate function X to webview"
8. Main agent updates progress tracker
9. Move to next task
```
