# Sourcegraph Codex Plugin

An [Agent Plugin](https://agent-plugins.org) that wires Sourcegraph's MCP server into [Codex](https://developers.openai.com/codex), giving Codex disciplined code search, navigation, history, and Deep Search over your Sourcegraph-indexed repositories.

## What's in here

```
plugin.json     # Agent Plugins manifest
mcp.json        # Declares the "sourcegraph" streamable-http MCP server
skills/
  searching-sourcegraph/
    SKILL.md                    # Tool-selection logic, scoping rules, workflows
    query-patterns.md           # Regex query reference by language/intent
    examples/common-searches.md # Worked search examples
    workflows/
      implementing-feature.md
      understanding-code.md
      debugging-issue.md
      fixing-bug.md
      code-review.md
```

The `searching-sourcegraph` skill teaches Codex when to reach for `code_finder` vs `keyword_search` vs `nls_search` vs `deepsearch`, how to scope queries with `repo:`/`file:` filters, and gives step-by-step workflows for common engineering tasks (implementing a feature, debugging, fixing a bug, reviewing a PR).

## Tools exposed

| Tool | Purpose |
|------|---------|
| `code_finder` | Agentic search — describe what you want, it finds candidate files/lines |
| `keyword_search` | Exact-pattern / regex search |
| `nls_search` | Natural-language / semantic search |
| `find_references` | Trace symbol usage |
| `go_to_definition` | Jump to implementation |
| `read_file` / `list_files` | Read and browse repo contents |
| `list_repos` | Find repositories |
| `commit_search` / `diff_search` / `compare_revisions` | History and change tracking |
| `deepsearch` / `deepsearch_read` | Multi-step research jobs over the codebase |
| `get_contributor_repos` | Find repos a given user has worked on |
| `evaluator` | Sandboxed Lua aggregation over search results (counts, joins, filters) |

`code_finder`, `evaluator`, and `deepsearch` require the `/.api/mcp/all` endpoint — see [Known limitations](#known-limitations).

## Installation

Install through Codex's plugin manager, or copy this directory into your Codex plugins location and point Codex at it.

## Configuration

The plugin talks to your own Sourcegraph instance's MCP endpoint. `mcp.json` currently declares:

```json
{
  "mcpServers": {
    "sourcegraph": {
      "type": "streamable-http",
      "url": "${SOURCEGRAPH_ENDPOINT}/.api/mcp/all"
    }
  }
}
```

**`${SOURCEGRAPH_ENDPOINT}` is not yet functional** — see below. Once resolved, it should point at your Sourcegraph instance (e.g. `https://sourcegraph.example.com` or `https://sourcegraph.com`).

Authentication is expected to happen via OAuth: Sourcegraph's MCP server supports OAuth 2.0 Dynamic Client Registration, and Codex owns/injects the `Authorization` header for MCP connections itself, so no access token needs to be configured in this plugin.

## Known limitations

The [Agent Plugins v1 spec](https://github.com/agentplugins/agent-plugins-spec) does not allow `${VAR}`-style expansion in a `streamable-http`/`sse` server's `url` (only `${PLUGIN_ROOT}`/`${PLUGIN_DATA}` are expanded, and only for `stdio` servers' `args`/`env`/`cwd`). Codex's implementation enforces this and substituting `${SOURCEGRAPH_ENDPOINT}` fails URL validation
