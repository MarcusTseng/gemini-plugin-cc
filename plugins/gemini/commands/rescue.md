---
description: Delegate an investigation, fix, or long-running task to Gemini
argument-hint: '[--wait|--background] [--resume] [--fresh] [task description]'
allowed-tools: Bash(acpx:*), Bash(mkdir:*), Bash(date:*), Bash(ls:*), Bash(echo:*), AskUserQuestion
---

Delegate a task to Gemini via a persistent acpx session. Gemini handles it independently.

Raw user request:
`$ARGUMENTS`

Execution mode:
- If `--background` is in arguments, run background without asking.
- If `--wait` is in arguments, run foreground without asking.
- Otherwise default to foreground.

Session handling:
- Strip `--wait`, `--background`, `--resume`, `--fresh` from the task text before sending.
- If `--resume` is present, check for an existing session:
```bash
acpx gemini sessions list 2>/dev/null | head -5
```
  Resume the most recent session (omit `sessions new`).
- If `--fresh` is present, always create a new session.
- If neither flag is present and this is clearly a follow-up ("continue", "keep going", "apply", "dig deeper"), ask once:
  - `Continue current Gemini session (Recommended)`
  - `Start a new Gemini session`
- Otherwise, start fresh by default.

Foreground flow (new session):
```bash
mkdir -p ~/.claude/gemini-jobs
SESSION_NAME="rescue-$(date +%s)"
acpx gemini sessions new --name "$SESSION_NAME"
acpx gemini -s "$SESSION_NAME" '<task text>'
```
Return Gemini's output verbatim.

Foreground flow (resume):
```bash
acpx gemini '<task text>'
```
(acpx auto-resumes the last session for the current cwd.)

Background flow:
```bash
mkdir -p ~/.claude/gemini-jobs
SESSION_NAME="rescue-$(date +%s)"
```
Then launch with Bash in background:
```typescript
Bash({
  command: `acpx gemini sessions new --name "${SESSION_NAME}" && acpx gemini -s "${SESSION_NAME}" '<task text>' > ~/.claude/gemini-jobs/${SESSION_NAME}.txt 2>&1`,
  description: "Gemini rescue: <task summary>",
  run_in_background: true
})
```
Tell the user: "Gemini rescue started in the background as session `<SESSION_NAME>`. Check `/gemini:result` when done."
