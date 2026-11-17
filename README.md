# tracktrace

Where is that parcel, exactly? tracktrace is a small forwarding client for the hosted Last Scanned package-tracking service: the tool list is served from your own machine using the captured definitions, and each lookup is sent upstream, with the service's answer returned exactly as it arrived.

All three of the service's captured tools are exposed. You can follow one tracking number end to end, check a whole batch of numbers in a single call, and read the list of carrier ids the service understands. Every tool only reads carrier scan data; nothing is bought, booked or sent.

## The tools

- **`track_package`** — The current status, estimated delivery and full scan history of one `tracking_number`. Pass the optional `carrier` id when you know it; leave it out and the service works the carrier out from the number itself.
- **`track_packages`** — The batch form: `tracking_numbers` takes from 1 to 100 numbers at once (at most 35 per carrier). Each entry comes back with its own result or its own error, so one bad number never fails the rest. Past the caller's daily limit (one number without a key), the remaining entries come back with a `payment_required` or `rate_limited` error — that is metering against the caller's own allowance, surfaced per entry; there is no purchase or top-up tool here.
- **`list_carriers`** — The carrier ids that can be passed as `carrier` to the other two tools. No arguments.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/tracktrace mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "tracktrace": {
      "command": "npx",
      "args": ["-y", "@reneturcios/tracktrace", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/tracktrace
docker run --rm ghcr.io/cloudcorprecords/tracktrace list_carriers '{}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
tracktrace track_package '{"tracking_number":"1Z999AA10123456784"}'
tracktrace track_packages '{"tracking_numbers":["1Z999AA10123456784","9400111899223856928499"]}'
tracktrace list_carriers '{}'
tracktrace --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `LASTSCANNED_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://mcp.lastscanned.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The Last Scanned registry record lists no headers for its remote, so tracktrace never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `tracktrace error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, tracktrace repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
