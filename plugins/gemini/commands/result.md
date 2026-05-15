---
description: Show the output of the latest (or a specific) background Gemini job
argument-hint: '[job-name]'
allowed-tools: Bash(ls:*), Bash(cat:*)
---

Show the result of a background Gemini job.

Raw arguments:
`$ARGUMENTS`

Rules:
- If a job name is given, read `~/.claude/gemini-jobs/<job-name>.txt`.
- Otherwise read the most recently modified file in `~/.claude/gemini-jobs/`.

List jobs:
```bash
ls -lt ~/.claude/gemini-jobs/ 2>/dev/null | head -10
```

Read latest:
```bash
ls -t ~/.claude/gemini-jobs/ | head -1 | xargs -I{} cat ~/.claude/gemini-jobs/{}
```

Read specific:
```bash
cat ~/.claude/gemini-jobs/<job-name>.txt
```

Return output verbatim. If no jobs exist, tell the user no background jobs have been run yet.
