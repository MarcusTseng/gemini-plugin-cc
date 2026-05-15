---
description: Ask Gemini a one-shot question, optionally with codebase context
argument-hint: '<question> [--files <glob>]'
allowed-tools: Bash(acpx:*), Bash(find:*), Bash(cat:*), Bash(mktemp:*), Bash(rm:*)
---

Ask Gemini a question. Uses its 1M token context window — good for large codebase context.

Raw user question:
`$ARGUMENTS`

Rules:
- If `--files <glob>` is present, collect matching files and include their content as context.
- Otherwise just ask the question directly.
- Return Gemini's answer verbatim.

Without files:
```bash
acpx gemini exec '<question>'
```

With `--files <glob>`:
1. Extract the glob pattern from arguments.
2. Collect file contents:
```bash
TMPFILE=$(mktemp /tmp/gemini-ask-XXXX.txt)
echo "<question>" > "$TMPFILE"
echo "" >> "$TMPFILE"
echo "--- Codebase context ---" >> "$TMPFILE"
find . -name '<glob>' | head -50 | xargs -I{} sh -c 'echo "=== {} ===" && cat {}' >> "$TMPFILE" 2>/dev/null
```
3. Run:
```bash
acpx gemini exec -f "$TMPFILE"
rm -f "$TMPFILE"
```
4. Return output verbatim.
