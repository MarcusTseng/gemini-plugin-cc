---
description: Check whether Gemini CLI and acpx are ready
argument-hint: ''
allowed-tools: Bash(which:*), Bash(gemini:*), Bash(acpx:*), Bash(echo:*), Bash(tail:*)
---

Check that Gemini CLI and acpx are installed and authenticated.

Run these checks in order:

1. Check acpx:
```bash
which acpx && acpx --version 2>/dev/null || echo "acpx: NOT FOUND — install with: npm install -g acpx"
```

2. Check Gemini CLI:
```bash
which gemini && gemini --version 2>/dev/null || echo "gemini: NOT FOUND — install with: npm install -g @google/gemini-cli"
```

3. Verify Gemini auth with a quick ping:
```bash
acpx gemini exec 'reply with only: ok' 2>&1 | tail -3
```

Report results clearly:
- If all pass: "Gemini plugin is ready. Use `/gemini:review`, `/gemini:rescue`, `/gemini:ask`."
- If acpx missing: direct user to `npm install -g acpx`
- If gemini missing: direct user to `npm install -g @google/gemini-cli`
- If auth fails: direct user to run `gemini auth`

Note: requires a Unix-like environment (Linux/macOS/WSL) — Windows native shell is not supported.
