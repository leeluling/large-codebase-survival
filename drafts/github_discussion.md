# GitHub Discussion Draft

## Post to: anthropics/claude-code Discussions (Tips & Tricks or Show and Tell category)

---

## Title:
Methodology for working with large codebases (15K+ lines) without hitting 503

## Body:

After losing 20+ Claude Code sessions to 503 errors while working on a 15K-line Python file, I developed a methodology that finally made large codebase work reliable.

### The problem
Working with large files causes context overflow → 503 → permanent session death. No recovery possible.

### The solution
6 principles centered around one idea: **the main agent orchestrates, subagents do the heavy lifting.**

Key practices:
- Use the Agent tool for all file reads over 500 lines — if the subagent 503s, your session survives
- Save progress (git commit + progress file) after every completed unit of work
- Break tasks into the smallest possible pieces — one function per subagent call
- Keep a progress tracker so new sessions resume seamlessly

### Resources
I published a full guide with:
- Detailed methodology (6 principles with examples)
- Copy-paste `CLAUDE.md` snippet that makes Claude Code follow these rules automatically
- Progress tracker template

**Repo:** https://github.com/leeluling/large-codebase-survival

Feedback and contributions welcome. Would love to hear if others have found similar patterns.
