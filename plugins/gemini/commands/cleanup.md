---
description: Remove old Gemini background job output files
argument-hint: '[--all] [--older-than <days>]'
allowed-tools: Bash(ls:*), Bash(rm:*), Bash(find:*), Bash(echo:*), AskUserQuestion
---

Clean up background job output files in `~/.claude/gemini-jobs/`.

Raw arguments:
`$ARGUMENTS`

Rules:
- If `--all` is given, remove all files after confirmation.
- If `--older-than <days>` is given, remove files older than that many days.
- Otherwise list current jobs and ask what to remove.

List current jobs:
```bash
ls -lt ~/.claude/gemini-jobs/ 2>/dev/null || echo "(no jobs found)"
```

Remove all (only after user confirms via AskUserQuestion):
```bash
rm -f ~/.claude/gemini-jobs/*.txt
echo "All job files removed."
```

Remove older than N days:
```bash
find ~/.claude/gemini-jobs/ -name "*.txt" -mtime +<N> -delete
echo "Removed job files older than <N> days."
```

Always confirm before deleting with `--all`. For `--older-than`, show count first then delete.
