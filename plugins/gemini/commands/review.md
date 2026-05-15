---
description: Run a Gemini code review against local git state using its 1M token context window
argument-hint: '[--wait|--background] [--base <ref>] [focus text]'
allowed-tools: Bash(git:*), Bash(acpx:*), Bash(mktemp:*), Bash(cat:*), Bash(rm:*), AskUserQuestion
---

Run a Gemini code review on current git changes. Gemini's 1M context window handles large diffs well.

Raw slash-command arguments:
`$ARGUMENTS`

Core constraint:
- This command is review-only.
- Do not fix issues or apply patches.
- Return Gemini's review output verbatim.

Execution mode rules:
- If `--wait` is in arguments, run foreground without asking.
- If `--background` is in arguments, run background without asking.
- Otherwise estimate size first:
  - Run `git status --short --untracked-files=no` and `git diff --shortstat` (or `git diff --shortstat <base>...HEAD` if `--base <ref>` given).
  - If tiny (1-2 files, small diff), recommend foreground.
  - Otherwise recommend background.
  - Ask once with `AskUserQuestion`:
    - `Wait for results`
    - `Run in background`

Argument handling:
- If `--base <ref>` is present, extract the ref and use `git diff <ref>...HEAD` for the diff.
- Any remaining text after flags is extra focus instructions to append to the review prompt.
- Strip `--wait`, `--background`, `--base <ref>` before building the focus text.

Foreground flow:
1. Collect the diff:
```bash
DIFF=$(git diff --stat HEAD && echo "---" && git diff HEAD)
```
   If `--base <ref>` given, use `git diff <ref>...HEAD` instead.

2. Build prompt file:
```bash
TMPFILE=$(mktemp /tmp/gemini-review-XXXX.txt)
cat > "$TMPFILE" <<'PROMPT'
You are a senior code reviewer. Review the following git diff thoroughly.
Look for: bugs, security issues, performance problems, code quality, missing edge cases.
Be specific — cite file and line when possible. <FOCUS>
---
<DIFF>
PROMPT
```
   Replace `<FOCUS>` with any extra focus text from arguments (or remove the line if none).
   Replace `<DIFF>` with the actual diff content.

3. Run:
```bash
acpx gemini exec -f "$TMPFILE"
rm -f "$TMPFILE"
```
4. Return output verbatim.

Background flow:
1. Collect diff and build prompt file as above.
2. Launch with Bash in background:
```typescript
Bash({
  command: `acpx gemini exec -f "$TMPFILE" > ~/.claude/gemini-jobs/review-$(date +%s).txt 2>&1 && rm -f "$TMPFILE"`,
  description: "Gemini review",
  run_in_background: true
})
```
3. Ensure `~/.claude/gemini-jobs/` exists first: `mkdir -p ~/.claude/gemini-jobs`
4. Tell the user: "Gemini review started in the background. Check `/gemini:result` when done."
