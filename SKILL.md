---
name: codex-headless
description: >-
  This skill should be used when the user asks to "run Codex headless",
  "use codex exec", "execute a Codex prompt", "Codex 腳本",
  "pipe output from Codex", "auto-approve Codex commands",
  or discusses running OpenAI Codex programmatically, integrating into scripts,
  cron jobs, CI/CD workflows, structured JSON output, sandbox policies,
  or Codex CLI features, flags, SDK, and configuration on macOS.
version: 0.1.0
tools: Bash
argument-hint: "[prompt or flags]"
---

# OpenAI Codex Headless (macOS)

Use the locally installed **OpenAI Codex** CLI (`codex exec`) in headless (non-interactive) mode on **macOS**.

> This skill is for driving the `codex` CLI programmatically via `codex exec`, not the OpenAI API directly.

## Environment checks

```bash
# Verify codex is installed
which codex
codex --version

# Quick smoke test
codex exec "Return only the single word OK."
```

If `codex` is not found, install via:
```bash
npm install -g @openai/codex
```

---

## Headless mode basics

Use `codex exec` to run non-interactively. The prompt can be passed as a positional argument or via stdin.

### Simple prompt

```bash
codex exec "What does the auth module do?"
```

### Run in a specific directory

```bash
codex exec --cd /path/to/repo "Summarize this project"
```

### Pipe prompt from stdin

```bash
echo "Explain this codebase" | codex exec
```

---

## PTY wrapper (macOS-specific)

Codex may hang without a TTY in certain environments. On **macOS (BSD)**, use `script` to allocate a pseudo-terminal:

```bash
# macOS BSD syntax (different from Linux!)
script -q /dev/null codex exec "Your prompt here"
```

**Do NOT use Linux syntax** (`script -q -c "cmd" /dev/null`) -- it will fail on macOS.

A wrapper script is provided that handles this automatically:

```bash
python3 ~/.claude/skills/codex-headless/scripts/codex_headless.py \
  "Your prompt here"
```

---

## Key CLI flags for `codex exec`

Most-used flags:

| Flag | Short | Description |
|------|-------|-------------|
| `--model` | `-m` | Model to use (e.g. `o4-mini`) |
| `--sandbox` | `-s` | Sandbox policy: `read-only`, `workspace-write`, `danger-full-access` |
| `--full-auto` | | Convenience: workspace-write + auto-approve |
| `--cd` | `-C` | Working directory for the agent |
| `--json` | | Print events to stdout as JSONL |
| `-o` | | Write last agent message to a file |

See [references/recipes.md](references/recipes.md#complete-cli-flags-for-codex-exec) for the complete flags table.

---

## Structured output

```bash
# JSONL event stream
codex exec --json "Summarize this project"

# Write last message to file
codex exec -o result.txt "Explain the auth module"

# JSON Schema (typed output) -- provide a schema file for structured responses
codex exec --output-schema schema.json "Extract function names from auth.py"
```

---

## Sandbox policies

Control what commands the agent can execute:

| Policy | Description |
|--------|-------------|
| `read-only` | No file writes or destructive commands |
| `workspace-write` | Can write within the workspace directory |
| `danger-full-access` | Full system access (use with caution) |

```bash
# Safe read-only analysis
codex exec --sandbox read-only "Analyze and propose a plan"

# Allow workspace writes (recommended for most tasks)
codex exec --sandbox workspace-write "Refactor the auth module"

# Full auto mode (workspace-write + auto-approve)
codex exec --full-auto "Run tests and fix failures"
```

---

## Session management

```bash
# Resume a session (by ID or most recent)
codex exec resume <SESSION_ID>
codex exec resume --last
codex exec resume --last "Continue working on the fix"

# Fork: new independent session with same conversation history
codex fork <SESSION_ID>
codex fork --last
```

---

## Model selection

```bash
# Use a specific model
codex exec -m o4-mini "Quick code review"

# Use open-source provider
codex exec --oss "Analyze this code"

# Use local provider
codex exec --local-provider ollama -m llama3 "Explain this function"
```

---

## macOS-specific integrations

```bash
# Copy output to clipboard
codex exec -o /dev/stdout "Summarize this project" | pbcopy

# Paste clipboard as input
pbpaste | codex exec "Review this code"

# Desktop notification on completion
codex exec --full-auto "Run all tests"; \
  osascript -e 'display notification "Codex task finished" with title "Codex"'

# Combine: run, notify, and copy result
codex exec -o result.txt "Summarize this project"; \
  cat result.txt | pbcopy; \
  osascript -e 'display notification "Result copied" with title "Codex"'
```

---

## Interactive mode (tmux)

For long-running tasks that need monitoring, use the Python wrapper with tmux:

```bash
python3 ~/.claude/skills/codex-headless/scripts/codex_headless.py \
  --mode interactive \
  --tmux-session my-session \
  --sandbox workspace-write \
  "Your interactive prompt here"
```

Monitor:
```bash
tmux attach -t my-session
```

---

## Common recipes

Quick examples (see [references/recipes.md](references/recipes.md#common-recipes) for the full collection):

```bash
# Code review a PR
gh pr diff 123 | codex exec "Review this PR for bugs and security issues"

# Fix test failures
codex exec --full-auto "Run the test suite and fix any failures"
```

---

## TypeScript SDK (programmatic usage)

For deeper integration, use the `@openai/codex-sdk` package. See [references/recipes.md](references/recipes.md#typescript-sdk-programmatic-usage) for SDK examples.

---

## Background mode (non-blocking)

Run tasks in the background -- the wrapper returns immediately with PID and log path.

```bash
python3 ~/.claude/skills/codex-headless/scripts/codex_headless.py \
  --background --notify \
  --full-auto "Run all tests and fix failures"
```

Flags: `--background` / `--bg` (run in background), `--log-dir <path>` (log directory), `--notify` (macOS notification on completion).

---

## Best practices

1. **Start with `--sandbox read-only`** for analysis, then switch to `--full-auto` for implementation
2. **Use `--full-auto`** as the default for trusted tasks (workspace-write + auto-approve)
3. **Avoid `--yolo`** unless running inside an externally sandboxed environment (Docker, VM)
4. **Use `--json`** for machine-readable event streams in automation pipelines
5. **Use `-o`** to capture the final agent message to a file for post-processing
6. **Use `--output-schema`** for structured, typed responses
7. **Pipe liberally** -- treat `codex exec` as a Unix utility
8. **Use `--cd`** to set the working directory instead of `cd`-ing first
9. **Use `--ephemeral`** for one-off tasks where session persistence isn't needed
10. **Capture session IDs** via `--json` output for resuming or forking later

---

## Looking up Codex documentation

For unfamiliar flags, SDK usage, or advanced configuration, use the **smart-search** skill to query documentation. See [references/recipes.md](references/recipes.md#looking-up-codex-documentation) for example queries and guidance on when to search.

---

## Additional Resources

- [references/recipes.md](references/recipes.md) -- Complete CLI flags table, common usage recipes, TypeScript SDK examples, and documentation lookup guidance
