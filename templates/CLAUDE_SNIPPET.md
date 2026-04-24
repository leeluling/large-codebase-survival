# CLAUDE.md Snippet for Large Codebase Survival

Copy everything below the line into your project's `CLAUDE.md`.

---

## Large File Survival Mode

When working with files over 5K lines or when context overflow / 503 errors occur:

1. **Never read large files directly** — use the Agent tool (subagents) for any file read over 300 lines. If the subagent crashes, the main session survives.

2. **Save after every unit of work** — git commit and update `PROGRESS.md` after completing each function/module/component. Never hold progress only in conversation context.

3. **Minimize context usage** — only read what's directly needed. Use line ranges (offset/limit) instead of reading whole files. Keep responses concise. Tell subagents to limit output to under 200 words.

4. **Break tasks into small pieces** — decompose large tasks until each piece can be completed in a single subagent call. No task should be larger than one function or one component.

5. **Monitor context health** — every 3-4 completed tasks, check if the session is getting heavy. If so, save all progress and suggest starting a new session.

6. **Design for session death** — maintain a `PROGRESS.md` tracker. Use descriptive git commits. A new session should resume seamlessly from any point.

### Execution pattern

```
For each small task:
  1. Subagent reads target code section (limited line range, summary under 100 words)
  2. Subagent implements the change (writes directly to file, no code in conversation)
  3. Main agent verifies (quick syntax/import check)
  4. Main agent commits and updates PROGRESS.md
```
