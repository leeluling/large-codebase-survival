# Large Codebase Rules

Copy this into your project's `CLAUDE.md` to have Claude Code follow these rules automatically.

---

## Context Overflow Prevention

When working with large files (5K+ lines) or multi-step tasks:

1. **Never read large files directly** — use the Agent tool (subagents) for reading/analyzing files over 500 lines. If the subagent crashes, the main session survives.

2. **Save after every unit of work** — write to file, commit, and update progress after completing each function/module/component. Never hold progress only in conversation context.

3. **Minimize context usage** — only read what's directly needed. Use line ranges (offset/limit) instead of reading whole files. Keep responses concise. Tell subagents to limit output to under 200 words.

4. **Break tasks into small pieces** — decompose large tasks until each piece can be completed in a single subagent call. Work one piece at a time.

5. **Reserve context buffer** — if the session feels heavy, save progress and consider a fresh session. Limit subagent output length explicitly.

6. **Design for session death** — maintain a progress tracker file. Use descriptive git commits. Store key decisions in memory. A new session should resume seamlessly from any point.

## Subagent Usage Pattern

```
Main agent = project manager (lightweight, orchestrates)
Subagents = developers (do the heavy reading/writing, disposable)
```

For any task involving large files:
- Spawn a subagent to read and summarize the relevant section
- Spawn a subagent to write the implementation to a file
- Main agent integrates, tests, commits, updates progress
