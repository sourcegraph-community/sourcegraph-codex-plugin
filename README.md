# Sourcegraph Codex Plugin

An [Agent Plugin](https://agent-plugins.org) that wires Sourcegraph's MCP server into [Codex](https://developers.openai.com/codex). Giving Codex disciplined code search, navigation, history, and Deep Search over your Sourcegraph-indexed repositories.

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

`code_finder`, `evaluator`, and `deepsearch` require the `/.api/mcp/all` endpoint, which every URL template in `mcp.json` uses.

## Installation

This is a templated plugin. OpenAI publishes the template once, and each ChatGPT workspace administrator creates a configured instance of it that points at their company's Sourcegraph instance. Users connect to and use that configured instance; the template itself is not directly installable.

## Configuration

`mcp.json` declares one templated `streamable-http` MCP server. The endpoint formats it supports are listed in `extensions.com.openai.urlTemplates`:

```json
"urlTemplates": [
  "https://{instance}.sourcegraph.com/.api/mcp/all",
  "https://{instance}.sourcegraphcloud.com/.api/mcp/all"
]
```

When creating an instance, the workspace administrator picks the format their Sourcegraph Cloud instance uses and sets `{instance}` to its subdomain. For example, `https://acme.sourcegraphcloud.com` uses the second format with `instance = acme`. The `url` field (`https://example.sourcegraph.com/.api/mcp/all`) is only an illustrative example that matches the first template; OpenAI does not connect to it.

Authentication uses OAuth with Dynamic Client Registration (`"client": { "mode": "dcr" }`). Sourcegraph's MCP endpoint supports DCR and only grants DCR clients the `mcp` scope, so `defaultScopes` is `["mcp"]`; Sourcegraph adds `offline_access` itself so connections can refresh their tokens. The package contains no client IDs, secrets, or tokens. Administrators can override these defaults per instance.

Listing metadata, such as the display name, description, and support links, lives in `plugin.json` under `extensions.com.openai.interface`.

## Known limitations

- Only Sourcegraph Cloud instances on `*.sourcegraph.com` and `*.sourcegraphcloud.com` subdomains are supported. Every URL template must contain a placeholder, so `https://sourcegraph.com` itself and self-hosted instances on custom domains cannot be selected.
- Codex CLI (checked with 0.169.0) rejects the `extensions` block in an `mcp.json` server entry (``unknown field `extensions`, expected `url` or `headers` ``). Installing this directory locally in Codex loads the plugin and its skill but skips the MCP server. To use it locally, add the server yourself, replacing `acme` with your instance. Codex then opens a browser to sign in to Sourcegraph; run `codex mcp login sourcegraph` to sign in again later.

  ```sh
  codex mcp add sourcegraph --url https://acme.sourcegraphcloud.com/.api/mcp/all
  ```
