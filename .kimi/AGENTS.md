# context-mode — Context Window Optimization

context-mode MCP tools are available in this project. Use them to reduce context window usage and improve efficiency.

## Available Tools

The following tools are registered (bare names, no prefix):

- `ctx_execute` — Run code in a sandboxed subprocess (12 languages). Only stdout enters context.
- `ctx_execute_file` — Read a file into a sandbox and run code against it.
- `ctx_batch_execute` — Run multiple commands + searches in ONE call. Use concurrency 1-8 for I/O-bound work.
- `ctx_index` — Store content in a searchable FTS5 knowledge base.
- `ctx_search` — Query indexed content with BM25 ranking.
- `ctx_fetch_and_index` — Fetch URL, convert to markdown, chunk and index. Cache respects TTL (default 24h).
- `ctx_stats` — Show context savings and usage statistics.
- `ctx_doctor` — Diagnose installation and runtime issues.
- `ctx_upgrade` — Upgrade to latest context-mode version.
- `ctx_purge` — Delete all indexed content.
- `ctx_insight` — Open analytics dashboard.

## When to Use

- **Analyzing large outputs**: Instead of `Shell` or `ReadFile` dumping raw data into context, use `ctx_execute` or `ctx_execute_file` to process data and only return the result.
- **Web fetching**: Use `ctx_fetch_and_index` + `ctx_search` instead of `FetchURL` for large pages.
- **Batch operations**: Use `ctx_batch_execute` with multiple commands to replace many individual tool calls.
- **Search/grep with large results**: Use `ctx_execute(language: "shell", code: "grep ...")` instead of `Grep` when results may exceed 20 lines.
- **Multi-URL fetching**: Use `ctx_fetch_and_index(requests: [{url, source}, ...], concurrency: N)` for parallel I/O.

## Think in Code

For analysis, counting, filtering, or transformation: write a script via `ctx_execute` and `console.log()` only the answer. Do NOT read raw data into context to process it manually.

## Examples

```
# Instead of reading 50 files to count lines:
ctx_execute(language: "javascript", code: `
  const fs = require('fs');
  const files = fs.readdirSync('src').filter(f => f.endsWith('.ts'));
  files.forEach(f => console.log(f + ': ' + fs.readFileSync('src/'+f,'utf8').split('\\n').length + ' lines'));
`)

# Batch gather + search:
ctx_batch_execute(commands: [
  {label: "git log", command: "git log --oneline -20"},
  {label: "file sizes", command: "find src -type f -exec wc -l {} +"}
], queries: ["recent commits", "largest files"])

# Fetch and index a docs page, then search:
ctx_fetch_and_index(url: "https://example.com/docs", source: "docs")
ctx_search(queries: ["authentication", "rate limits"])
```
