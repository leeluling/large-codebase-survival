# Reddit Post Draft

## Subreddits to post in:
- r/ClaudeAI
- r/cursor
- r/ChatGPTCoding

---

## Title:
I lost 20+ Claude Code sessions to 503 errors before figuring out how to actually work with a 15K-line file. Here's the methodology.

## Body:

I spent months trying to migrate a production desktop app (15,000+ lines of Python, single file) from SimpleGUI to WebView using Claude Code. Every single session crashed with a 503 error. Context overflow. Session permanently dead. Progress lost.

After losing count of how many sessions I burned through, I finally figured out a set of principles that made it work. The project went from "impossible" to "slow but steady progress, every session."

**The core idea:** treat the AI agent as a project manager, not a developer. It should never read the big file directly. Instead, it delegates focused tasks to subagents (disposable workers). If a worker crashes, the manager survives. Progress is saved after every small win. A new session picks up exactly where the last one left off.

**The 6 principles:**

1. **Protect the Main Agent** — delegate all heavy reading/writing to subagents
2. **Save Early, Save Often** — commit after every unit of work, assume the session can die anytime
3. **Minimize Context Consumption** — only read what you need, keep output concise
4. **Break Big Tasks into Small Pieces** — decompose until each piece fits in one agent call
5. **Reserve Context Buffer** — don't push to the limit
6. **Design for Session Continuity** — progress files, descriptive commits, persistent memory

I wrote it up as a guide with copy-paste templates (CLAUDE.md snippet, progress tracker):

**GitHub:** https://github.com/leeluling/large-codebase-survival

Works for Claude Code, Cursor, Windsurf, or any AI coding agent with context limits. MIT licensed.

Hope this saves someone the pain I went through.
