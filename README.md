# Gemini plugin for Claude Code

Use Gemini from inside Claude Code to review code or delegate tasks via [acpx](https://github.com/acpx).

## What You Get

- `/gemini:review` — code review using Gemini's 1M token context window
- `/gemini:rescue` — delegate an investigation or task to Gemini (foreground or background)
- `/gemini:ask` — one-shot question, optionally with file context
- `/gemini:status` — list recent background jobs and active sessions
- `/gemini:result` — read output from a background job
- `/gemini:setup` — verify Gemini CLI and acpx are ready

## Requirements

- **Gemini CLI** — install with `npm install -g @google/gemini-cli`, then authenticate with `gemini auth`
- **acpx** — install with `npm install -g acpx`
- A Google account with Gemini CLI access (free tier: 60 req/min, or API key)

## Install

Add this marketplace in Claude Code:

```bash
/plugin marketplace add MarcusTseng/gemini-plugin-cc
```

Install the plugin:

```bash
/plugin install gemini@gemini-local
```

Reload plugins:

```bash
/reload-plugins
```

Then verify:

```bash
/gemini:setup
```

## Usage

### `/gemini:review`

Review current uncommitted changes or a branch diff. Gemini's 1M token context handles large diffs well.

```bash
/gemini:review
/gemini:review --base main
/gemini:review --background
/gemini:review --background --base main focus on auth and security
```

### `/gemini:rescue`

Delegate an investigation, fix, or long-running task to Gemini.

```bash
/gemini:rescue investigate why the tests are failing
/gemini:rescue --background summarize all changes since last release
/gemini:rescue --resume apply the suggestion from last session
```

### `/gemini:ask`

Ask Gemini a one-shot question about your codebase.

```bash
/gemini:ask how does the auth flow work in this repo
/gemini:ask --files "**/*.py" what design patterns are used here
```

### `/gemini:status` / `/gemini:result`

Check background jobs and retrieve output.

```bash
/gemini:status
/gemini:result
/gemini:result rescue-1747123456
```

## How it works

This plugin uses [acpx](https://github.com/acpx) to communicate with the local Gemini CLI over the Agent Client Protocol (ACP). Gemini CLI runs locally — no data is sent to external servers beyond what Gemini normally does.

## Acknowledgements

- [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) — plugin structure and command patterns this repo is modeled after
- [beyond5959/acp-adapter](https://github.com/beyond5959/acp-adapter) — reference for understanding ACP internals and Gemini's native ACP support
- [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) — the upstream Gemini CLI this plugin wraps
- [acpx](https://github.com/acpx) — the ACP client used as the backend transport
