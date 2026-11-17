# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Last Scanned MCP server. The three captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `track_package`, `track_packages`, `list_carriers`. All three are pure reads of carrier scan data: the status, estimated delivery and scan history of one tracking number; a batch lookup of up to 100 numbers at once; and the list of carrier ids.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `tracktrace error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LASTSCANNED_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
