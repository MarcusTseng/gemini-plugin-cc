---
description: Ask Gemini a one-shot question, optionally with codebase context
argument-hint: '<question> [--files <glob>]'
allowed-tools: Bash(acpx:*), Bash(find:*), Bash(cat:*), Bash(mktemp:*), Bash(rm:*), Bash(head:*), Bash(grep:*), Bash(echo:*), Bash(xargs:*)
---

Ask Gemini a question. Uses its 1M token context window — good for large codebase context.

Raw user question:
`$ARGUMENTS`

Rules:
- If `--files <glob>` is present, collect matching files and include their content as context.
- Otherwise ask the question directly.
- Return Gemini's answer verbatim.

Security note: the glob pattern is passed to `find -name`, not evaluated by the shell, so it is safe against injection.

Without files:
```bash
acpx gemini exec '<question>'
```

With `--files <glob>`:
1. Extract the glob pattern and the question from arguments (everything before `--files` is the question).
2. Collect file contents — filter out binaries, lock files, and large files:
```bash
TMPFILE=$(mktemp /tmp/gemini-ask-XXXX.txt)
{
  echo "<question>"
  echo ""
  echo "--- Codebase context ---"
  find . \
    -not -path './.git/*' \
    -not -path '*/node_modules/*' \
    -not -name 'package-lock.json' \
    -not -name 'yarn.lock' \
    -not -name 'pnpm-lock.yaml' \
    -name '<glob>' \
    -type f \
    | head -50 \
    | while IFS= read -r f; do
        # Skip binary files
        if grep -qI '' "$f" 2>/dev/null; then
          echo "=== $f ==="
          cat "$f"
          echo ""
        fi
      done
} > "$TMPFILE"
acpx gemini exec -f "$TMPFILE"
rm -f "$TMPFILE"
```
3. Return output verbatim.
