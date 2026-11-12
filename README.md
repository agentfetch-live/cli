# AgentFetch CLI

Field notes on giving AI agents web access they can actually use.

`agentfetch` fetches live web pages as clean Markdown, text or HTML, straight from your terminal or your agent's shell.
It is the command line client for the [AgentFetch API](https://agentfetch.live).

## Quick start

```bash
npx @agentfetch-live/cli login                                   # paste your API key (not echoed)
npx @agentfetch-live/cli fetch https://example.com -o text       # the page as Markdown
```

Get a key at [agentfetch.live](https://agentfetch.live). Node.js 18 or later.

## Install

```bash
npm install -g @agentfetch-live/cli
agentfetch --help
```

## Use

```bash
agentfetch fetch https://example.com/a https://example.com/b      # up to 10 URLs; JSON with results[] and statuses[]
agentfetch fetch https://example.com -o text                      # just the content (Markdown, else text, else HTML)
agentfetch fetch https://example.com -o html --html               # the page HTML
agentfetch fetch https://example.com --country-code de --render   # from Germany, rendered in a real browser
agentfetch fetch --data @request.json                             # the whole request body from a file (- for stdin)
agentfetch account                                                # plan, credits and rate limit
agentfetch fetch --help                                           # every option, with the API's own descriptions
```

A fetch that is still running after about 100 seconds is collected automatically with the same
`Idempotency-Key`, so it is charged once.

## MCP server

The AgentFetch API is also a remote MCP server (Streamable HTTP) at `https://agentfetch.live/mcp`, with the tools
`get_page_contents` and `get_account_status`. It takes the same API key as a bearer token.

```bash
claude mcp add --transport http agentfetch https://agentfetch.live/mcp \
  --header "Authorization: Bearer $AGENTFETCH_API_KEY"
```

Any other client: URL `https://agentfetch.live/mcp`, header `Authorization: Bearer <key>`. Config for Cursor, VS Code and
Claude Desktop: [MCP docs](https://agentfetch.live/docs/mcp). The server's entry for the MCP Registry is [server.json](server.json)
(`live.agentfetch/agentfetch`).

## Agent plugin and skill

This repo is also a plugin for Claude Code, Grok Build and Gemini CLI, and holds an agent skill. Each one connects the
MCP server above and teaches the agent when to fetch a page with it or with `agentfetch`.

```bash
npx skills add agentfetch-live/cli                                    # the skill, for any agent that reads SKILL.md
claude plugin marketplace add agentfetch-live/cli && claude plugin install agentfetch@agentfetch
gemini extensions install https://github.com/agentfetch-live/cli
```

Network and credentials: the plugin calls `https://agentfetch.live/mcp` only. The CLI calls `https://agentfetch.live/api` and
`https://agentfetch.live/openapi.json`, and `https://registry.npmjs.org` (`npx` and a once-a-day update check, off with
`AGENTFETCH_NO_UPDATE_NOTIFIER=1`). Both need one credential, your AgentFetch API key,
read from `AGENTFETCH_API_KEY` or saved by `agentfetch login`. No hooks, no scripts run on install.

## The commands follow the live API

`agentfetch` builds its commands and options from the API's OpenAPI document at
[https://agentfetch.live/openapi.json](https://agentfetch.live/openapi.json), cached for an hour. When the API gains a field or an endpoint, the
option or command is there on your next run, without updating the CLI. Offline, it uses the last cached
copy, then the copy bundled with the installed version (`agentfetch spec` shows which one; `--offline` skips the
download). An hourly job here publishes a new version whenever the API spec changed.

## Configuration

| Setting | Option | Environment | Default |
|---|---|---|---|
| API key | `--api-key` | `AGENTFETCH_API_KEY` | the key saved by `agentfetch login` (`~/.config/agentfetch/config.json`, mode 0600) |
| API base URL | `--base-url` | `AGENTFETCH_API_BASE_URL` | `https://agentfetch.live/api` |
| API spec URL | `--spec-url` | `AGENTFETCH_SPEC_URL` | `https://agentfetch.live/openapi.json` |
| Output | `-o json\|text\|<field>` | | `json` |
| Update notice | | `AGENTFETCH_NO_UPDATE_NOTIFIER=1` turns it off | once a day, terminals only |

Requests and your key only ever go to the base URL, never to a server named in the spec.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | OK |
| 1 | The API answered with an error (JSON on stderr, with its `requestId`), every URL failed with `-o text`, or the fetch was still running at `--max-wait` |
| 2 | Usage: unknown command or option, a value the API does not accept |
| 3 | Network: no answer. The error carries the `Idempotency-Key` to collect the result without a new charge |

## Links

- [agentfetch.live](https://agentfetch.live)
- [CLI docs](https://agentfetch.live/docs/cli)
- [API docs](https://agentfetch.live/docs)
- [API spec](https://agentfetch.live/openapi.json)

## License

MIT, © agentfetch.live
