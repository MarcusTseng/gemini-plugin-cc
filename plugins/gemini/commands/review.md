---
description: Run a Gemini code review against local git state using its 1M token context window
argument-hint: '[--wait|--background] [--base <ref>] [focus text]'
allowed-tools: Bash(git:*), Bash(acpx:*), Bash(mktemp:*), Bash(mkdir:*), Bash(rm:*), Bash(date:*), Bash(echo:*), AskUserQuestion
---

Run a Gemini code review on current git changes. Gemini's 1M context window handles large diffs well.

Raw slash-command arguments:
`$ARGUMENTS`

Core constraint:
- This command is review-only. Do not fix issues or apply patches.
- Return Gemini's review output verbatim.

Execution mode rules:
- If `--wait` is in arguments, run foreground without asking.
- If `--background` is in arguments, run background without asking.
- Otherwise estimate size first:
  - Run `git status --short --untracked-files=no` and `git diff --shortstat` (or `git diff --shortstat <base>...HEAD` if `--base <ref>` given).
  - If tiny (1-2 files, small diff), recommend foreground. Otherwise recommend background.
  - Ask once with `AskUserQuestion`:
    - `Wait for results`
    - `Run in background`

Argument handling:
- If `--base <ref>` is present, extract the ref and use `git diff <ref>...HEAD` for the diff.
- Any remaining text after flags becomes the focus instruction appended to the prompt.
- Strip `--wait`, `--background`, `--base <ref>` before extracting focus text.

Foreground flow:

Build and execute in a single shell block — do NOT read or interpolate the diff into Claude's context:

```bash
mkdir -p ~/.claude/gemini-jobs
TMPFILE=$(mktemp /tmp/gemini-review-XXXX.txt)
{
  echo "You are a senior code reviewer. Review the following git diff thoroughly."
  echo "Look for: bugs, security issues, performance problems, code quality, missing edge cases."
  echo "Be specific — cite file and line when possible."
  # If focus text was provided, add: echo "Focus on: <focus text>"
  echo "---"
  git diff --stat HEAD
  echo "---"
  git diff HEAD
} > "$TMPFILE"
acpx gemini exec -f "$TMPFILE"
rm -f "$TMPFILE"
```

If `--base <ref>` was given, replace `git diff HEAD` with `git diff <ref>...HEAD`.

Return Gemini's output verbatim. Do not paraphrase or add commentary.

Background flow:

```bash
mkdir -p ~/.claude/gemini-jobs
TMPFILE=$(mktemp /tmp/gemini-review-XXXX.txt)
JOB="review-$(date +%s)"
{
  echo "You are a senior code reviewer. Review the following git diff thoroughly."
  echo "Look for: bugs, security issues, performance problems, code quality, missing edge cases."
  # If focus text was provided, add: echo "Focus on: <focus text>"
  echo "---"
  git diff --stat HEAD
  echo "---"
  git diff HEAD
} > "$TMPFILE"
```

Then launch with Bash in background:
```typescript
Bash({
  command: `acpx gemini exec -f "$TMPFILE" > ~/.claude/gemini-jobs/${JOB}.txt 2>&1; rm -f "$TMPFILE"`,
  description: "Gemini review",
  run_in_background: true
})
```

Tell the user: "Gemini review started in the background as `<JOB>`. Check `/gemini:result` when done."
