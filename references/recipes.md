# Codex Headless Reference

Supplementary reference material for the `codex-headless` skill.
For core workflow and basics, see the main [SKILL.md](../SKILL.md).

---

## Complete CLI flags for `codex exec`

| Flag | Short | Description |
|------|-------|-------------|
| `PROMPT` | | Initial instructions (positional arg or stdin) |
| `--model` | `-m` | Model to use (e.g. `o4-mini`, `codex-mini-latest`) |
| `--sandbox` | `-s` | Sandbox policy: `read-only`, `workspace-write`, `danger-full-access` |
| `--full-auto` | | Convenience: `-a on-request --sandbox workspace-write` |
| `--profile` | `-p` | Configuration profile from `config.toml` |
| `--cd` | `-C` | Working directory for the agent |
| `--image` | `-i` | Attach image(s) to the initial prompt |
| `--json` | | Print events to stdout as JSONL |
| `--output-last-message` | `-o` | Write last agent message to a file |
| `--output-schema` | | Path to JSON Schema for structured final response |
| `--ephemeral` | | Run without persisting session files |
| `--skip-git-repo-check` | | Allow running outside a Git repository |
| `--add-dir` | | Additional writable directories |
| `--color` | | Color settings for output |
| `--oss` | | Use an open-source provider |
| `--local-provider` | | Local provider (`lmstudio` or `ollama`) |
| `--dangerously-bypass-approvals-and-sandbox` / `--yolo` | | Skip all prompts and sandboxing (use with extreme caution) |

---

## Common recipes

### Git commit from staged changes

```bash
codex exec --full-auto "Look at my staged changes and create an appropriate commit"
```

### Code review a PR

```bash
gh pr diff 123 | codex exec "Review this PR for bugs and security issues"
```

### Fix test failures

```bash
codex exec --full-auto "Run the test suite and fix any failures"
```

### Explain + Plan first, then implement

```bash
# Step 1: plan (read-only)
codex exec --sandbox read-only "Analyze the auth system and propose improvements"

# Step 2: implement
codex exec --full-auto "Implement the improvements to the auth system"
```

### Batch processing with a loop

```bash
for file in src/**/*.py; do
  codex exec --sandbox workspace-write "Add type hints to $file"
done
```

### Pipe build errors for diagnosis

```bash
npm run build 2>&1 | codex exec "Explain the root cause and suggest a fix"
```

### Attach images for context

```bash
codex exec -i screenshot.png "What UI issues do you see in this screenshot?"
```

### Run without git repo

```bash
codex exec --skip-git-repo-check "Analyze the files in this directory"
```

### Use with additional writable directories

```bash
codex exec --full-auto --add-dir /tmp/output "Generate reports and save to /tmp/output"
```

---

## TypeScript SDK (programmatic usage)

For deeper integration, use the Codex TypeScript SDK:

```typescript
import { Codex } from "@openai/codex-sdk";

const codex = new Codex();
const thread = codex.startThread();
const turn = await thread.run("Diagnose the test failure and propose a fix");

console.log(turn.finalResponse);
console.log(turn.items);
```

---

## Looking up Codex documentation

When encountering unfamiliar flags, features, sandbox policies, SDK usage, or any Codex topic not covered in the main skill file, use the **smart-search** skill to query documentation. Smart-search automatically routes queries across DeepWiki, Context7, and Perplexity for the best answer.

### Example queries

```
"What CLI flags are available for codex exec headless mode?"
"How does the sandbox system work? What are all sandbox policies and their permissions?"
"How to configure Codex with config.toml profiles?"
"How does session management work? How to resume and fork sessions?"
"How to use the Codex TypeScript SDK programmatically?"
"How to use Codex with local providers (ollama, lmstudio)?"
"How does --output-schema work for structured responses?"
"How to use Codex with open-source models via --oss flag?"
```

### When to look up documentation

- A flag or option is not documented in this skill
- The user asks about SDK integration, custom providers, or advanced configuration
- Troubleshooting unexpected behavior or error messages
- Verifying the latest syntax or available options
- Any Codex feature beyond basic headless mode usage
