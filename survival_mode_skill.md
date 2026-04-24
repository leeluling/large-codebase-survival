# Large Codebase Survival Mode

## Trigger Conditions
- User hits frequent 503 / context overflow errors
- User asks for "survival mode" or mentions working with a large file (5K+ lines)
- User explicitly requests this workflow

## Configuration (optional)

These defaults work for most cases. Adjust if needed:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_lines_main` | 300 | Max lines the main agent can read directly. Increase for larger context models (e.g. 500 for Opus 1M). |
| `max_subagent_output` | 200 words | Word limit for subagent responses. Increase if summaries are too compressed. |
| `checkpoint_frequency` | every task | How often to git commit + update PROGRESS.md. Set to "every 2-3 tasks" if commits feel too granular. |
| `auto_commit` | true | Auto git commit after each task. Set to false if your project has CI hooks or protected branches. |

To override, the user can say e.g. "use survival mode with max_lines_main=500 and auto_commit=false".

## Prerequisites
- Git (for incremental commits, unless auto_commit is false)
- Claude Code Agent tool (subagents)
- A progress file will be auto-created in the project directory

## Execution Steps

### Step 1: Create progress tracker

Create `PROGRESS.md` in the project directory (or read it if it already exists):

```markdown
# Project: [task description from user]
## Status: In Progress
## Last updated: [today's date]

### Completed

### In Progress

### Known Issues

### Key Decisions
```

### Step 2: Analyze and decompose the task

Use a **subagent** to analyze the target file — never read it directly:

```
Agent({
  description: "Analyze file structure",
  prompt: "Read the first 50 and last 50 lines of [file path]. Use Grep to find all class and def/function definitions.
           Output a structural summary in under 200 words. List all functions/classes that need modification."
})
```

Based on the summary, break the task into small independent units. Write each unit into `PROGRESS.md`.

### Step 3: Execute one small task at a time

For each small task, follow this loop:

**3a. Read** — use a subagent to read the target code section (limited line range):
```
Agent({
  description: "Read function X",
  prompt: "Read [file] lines [M]-[N]. Summarize function X's logic in under 100 words.
           List its parameters, return value, and dependencies on other functions."
})
```

**3b. Implement** — use a subagent to write the code:
```
Agent({
  description: "Implement new version of function X",
  prompt: "Implement the new version of function X based on this spec: [paste summary from 3a].
           Write directly to the target file using Edit/Write tools. Do not output full code to the conversation."
})
```

**3c. Verify** — main agent does a quick sanity check:
```bash
python -c "import [module]; print('OK')"
```

**3d. Save** — commit immediately and update progress:
```bash
git add [file] && git commit -m "complete: [task description]"
```
Then update `PROGRESS.md` — mark the task as `[x]`.

### Step 4: Repeat Step 3 for all tasks

Loop through 3a → 3b → 3c → 3d for each small task.

### Step 5: Monitor context health

After every 3-4 small tasks, check:
- If the conversation is getting long → save all progress to `PROGRESS.md` and memory, suggest user starts a new session
- If subagents are slowing down or failing → reduce the scope of each subagent task

## Hard Rules

| Rule | Limit |
|------|-------|
| Main agent file reading | Never read more than 300 lines — must use subagent for larger reads |
| Subagent output | Every subagent prompt must include "under 200 words" or similar limit |
| Save frequency | Must git commit + update PROGRESS.md after every completed small task |
| Task granularity | Each small task should be no more than one function or one component |
| File reading | Must use offset/limit to specify line ranges — never read an entire large file |
| Conversation output | Main agent keeps responses concise — no unnecessary summaries or explanations |

## Known Limitations

1. Slower per session — but cumulative progress is much faster than repeated 503 crashes
2. Some tasks require holistic understanding — have a subagent generate a file outline first, then work from the outline
3. Subagents have their own context limits — keep individual subagent tasks focused

## Example

User input:
> Help me migrate main.py's UI from SimpleGUI to WebView. The file is 15000 lines.

Execution flow:
```
1. Create PROGRESS.md
2. Subagent → scan main.py structure, return summary (under 200 words)
3. Decompose into 12 small tasks, write to PROGRESS.md
4. Task 1: subagent reads lines 200-280 → subagent rewrites → verify → commit
5. Task 2: subagent reads lines 450-520 → subagent rewrites → verify → commit
6. ...
7. Check context health every 3-4 tasks
8. If session is getting full → save progress → suggest new session to continue
```

Output format:
```
[Survival Mode] Completed 4/12 tasks
✓ Main window layout (commit a1b2c3)
✓ Settings dialog (commit d4e5f6)
✓ File browser (commit g7h8i9)
✓ Event bindings (commit j0k1l2)
→ Next: Status bar component

Progress saved to PROGRESS.md
```
