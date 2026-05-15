---
description: Check whether Gemini CLI and acpx are ready
argument-hint: ''
allowed-tools: Bash(which:*), Bash(gemini:*), Bash(acpx:*)
---

Check that Gemini CLI and acpx are installed and authenticated.

Run these checks in order:

1. Check acpx:
```bash
which acpx && acpx --version 2>/dev/null || echo "acpx: NOT FOUND"
```

2. Check Gemini CLI:
```bash
which gemini && gemini --version 2>/dev/null || echo "gemini: NOT FOUND"
```

3. Check Gemini auth with a quick one-shot ping:
```bash
acpx gemini exec 'reply with only: ok' 2>&1 | tail -3
```

Report results clearly:
- If all pass: "Gemini plugin is ready. Use `/gemini:review`, `/gemini:rescue`, `/gemini:ask`."
- If acpx missing: "Install acpx: `npm install -g acpx`"
- If gemini missing: "Install Gemini CLI: `npm install -g @google/gemini-cli`"
- If auth fails: "Run `gemini auth` or `gemini login` to authenticate."
