---
name: loci
description: >
  Use loci CLI (execute, exec-file, batch, fetch, search, index) instead of Bash/Read/Grep when
  processing large outputs. This skill protects your context window by running commands in a sandbox
  and only returning search results. Make sure to use this skill whenever the user asks to analyze
  build output, summarize logs, process data, parse JSON, filter results, extract errors, check
  build output, analyze dependencies, process API responses, do large file analysis, run tests and
  check output, review git log or recent commits, diff between branches, find TODOs, count lines,
  check codebase statistics, fetch documentation, call APIs, or any operation that may produce more
  than 20 lines of output. The PreToolUse hook enforces routing automatically for curl/wget/WebFetch.
---

# loci-ctx -- context window protection

## Prerequisites

The `loci` binary must be available. Check with `which loci`. If not found:

```bash
# If in the loci-ctx repo with a compiled binary:
./target/release/loci version

# Otherwise install:
cargo install loci-ctx
```

The loci server daemon must be running for execute/exec-file/batch commands:

```bash
loci server start
```

If `loci` is genuinely unavailable, fall back to the Grep tool with targeted file-type filters and small result sets, or use other available sandbox tools (ck-search, etc.) that keep output out of context.

## Tool Routing

The loci CLI uses **long flags only** (`--lang`, `--code`, `--path`, `--url`). There are no short flag aliases.

| Instead of... | Use... |
|---|---|
| `Bash` with large output | `loci execute --lang shell --code '...'` |
| `Bash` with curl/wget | `loci fetch --url <url>` then `loci search -q '...'` |
| `Read` for analysis (not editing) | `loci exec-file --path <file> --lang python --code '...'` |
| `Grep` with many results | `loci execute --lang shell --code 'grep ...'` |
| `WebFetch` | `loci fetch --url <url>` then `loci search -q '...'` |
| Multiple commands + analysis | `loci batch --commands-json '[...]' --queries-json '[...]'` |

**When to use Read instead of exec-file**: If you are reading a file in order to **Edit** it, use Read -- you need the content in context for the edit. Use exec-file only when you are reading to **analyze or summarize**.

## Blocked Commands

These are intercepted by the PreToolUse hook and will be rejected:

- `curl` / `wget` in Bash -- Use `loci fetch --url <url>` instead
- `fetch('http...` / `requests.get(` / `requests.post(` in Bash -- Use `loci execute` instead
- `WebFetch` tool -- Use `loci fetch --url <url>` instead

## Usage Examples

### Run tests and analyze output

```bash
loci batch --commands-json '[{"language":"shell","code":"cargo test 2>&1"}]' \
  --queries-json '["test failures","FAILED","compilation error","warning"]'
```

### Execute code in sandbox

```bash
loci execute --lang python --code 'import json; data = json.load(open("/path/to/large.json")); print(len(data["items"]))'
loci execute --lang shell --code 'find . -name "*.rs" | wc -l'
```

### Analyze a file without flooding context

```bash
loci exec-file --path src/main.rs --lang python --code 'lines = content.splitlines(); print(f"Lines: {len(lines)}"); print("Imports:", [l for l in lines[:20] if l.startswith("use ")])'
```

### Fetch and index web content

```bash
loci fetch --url https://docs.example.com/api
loci search -q "authentication endpoints"
```

Note: `loci fetch` does not support custom HTTP headers. For APIs requiring authentication or User-Agent headers (like GitHub's API), use `loci execute` with a script:

```bash
loci execute --lang python --code '
import urllib.request, json
req = urllib.request.Request("https://api.github.com/repos/owner/repo/releases/latest",
    headers={"User-Agent": "loci-ctx"})
data = json.loads(urllib.request.urlopen(req).read())
print(json.dumps({"tag": data["tag_name"], "date": data["published_at"]}, indent=2))
'
```

### Multi-step batch operation

```bash
loci batch --commands-json '[{"language":"shell","code":"cargo test 2>&1"},{"language":"shell","code":"cargo clippy 2>&1"}]' \
  --queries-json '["test failures","clippy warnings"]'
```

### Index content for later search

```bash
loci index --source "error-log" --content "$(cat /tmp/error.log)"
loci search -q "connection timeout errors"
```

### Query knowledge base

```bash
loci search -q "how does authentication work"
loci search -q "database schema" --source "architecture-docs"
```

## Output Constraints

- Keep responses under 500 words
- Write artifacts (code, configs) to FILES -- never return them as inline text
- Return only: file path + 1-line description

## Subagent Usage

Subagents automatically receive loci tool routing via the PreToolUse hook. You do NOT need to manually add tool names to subagent prompts -- the hook injects them.
