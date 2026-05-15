---
description: Show running and recent Gemini background jobs
argument-hint: ''
allowed-tools: Bash(ls:*), Bash(acpx:*)
---

Show recent Gemini background jobs and active acpx sessions.

Rules:
- List recent job output files with timestamps.
- List active acpx Gemini sessions.
- Return results clearly formatted.

Run:
```bash
echo "=== Recent jobs ===" && ls -lt ~/.claude/gemini-jobs/ 2>/dev/null | head -10 || echo "(none)"
echo ""
echo "=== Active acpx sessions ===" && acpx gemini sessions list 2>/dev/null || echo "(none)"
```

Return output as-is.
