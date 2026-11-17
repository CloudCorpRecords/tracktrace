# BUILD REPORT — tracktrace

Clean-room rebuild of `com.lastscanned/lastscanned` v1.0.5
(remote-only MCP server at `https://mcp.lastscanned.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/tracktrace` 0.1.0, registry name
`io.github.CloudCorpRecords/tracktrace`.

**Provenance note (coordinator):** the original builder's
completion preview claimed a BUILD-REPORT and parity
output that were not on disk at the coordinator's first
verification pass (the same pattern as Batch B5's
collegeplan). The builder's parity output did land later;
this report was written by the Lane B Batch B6 coordinator
from the coordinator's own fresh foreground re-runs
(`specs-laneb6/verifyB6.py`). Every number below is from
those re-runs.

## Files (19-file factory set)

`package.json`, `server.json`, `Dockerfile`, `tsconfig.json`,
`.gitignore`, `package-lock.json`, `README.md`, `LICENSE`,
`CHANGELOG.md`, `src/index.ts`, `src/cli.ts`, `src/mcp.ts`,
`src/types.ts`, `src/forward.ts` (core), `test/forward.test.ts`,
`.github/workflows/ci.yml`, `.github/workflows/publish.yml`,
`.github/workflows/publish-container.yml`,
`.github/workflows/registry-publish.yml` — plus this report
and the `parity-out/` directory.

## Surface built

Full surface: **3 of 3 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `track_package` | `tracking_number` |
| `track_packages` | `tracking_numbers` |
| `list_carriers` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema` (the capture's `annotations`
key was dropped at fixture slimming, as the fixture's
`_source` note records). The tool literals were generated
programmatically from
`harness/fixtures/lastscanned-tools-original.json` — never
retyped — and the registration-integrity tests plus the
parity harness confirm the served literals deep-equal the
fixture's 3 tool objects, in order. Tools are registered
via the MCP SDK low-level Server API, not McpServer+zod.

Recorded judgment (from the spec): `track_packages`'
description states that past the caller's daily limit the
remaining entries return a `payment_required` or
`rate_limited` error — metering against the caller's OWN
pre-existing allowance, surfaced as a per-entry upstream
error; there is NO purchase, top-up or key-minting tool
anywhere in the surface (the appindex / acctcheck
distinction).

## Fixture conflicts

**None** found by the coordinator: the served surface
deep-equals the fixture (parity below).

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://mcp.lastscanned.com/mcp`.
- Env override `LASTSCANNED_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote:
  no auth header is ever sent, under any environment
  (asserted in tests).
- Error rendering `tracktrace error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `isError` and extra fields. tools/call is forwarded as
  streamable-HTTP JSON-RPC: initialize (protocolVersion
  `2025-03-26`) → notifications/initialized → tools/call,
  honouring `Mcp-Session-Id` when issued; plain-JSON and
  SSE replies are both parsed. Tests stub the upstream —
  no live network calls in tests, and no live call is
  claimed beyond the coordinator's capture.

## Lockfile

- Route: CLEAN Route 3 from birth
  (`npm install --package-lock-only --ignore-scripts --offline`),
  then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/tracktrace`.
- Package entries: **128**.
- sha256: `bfcac387fe66b859c40c00b72f9d5c3ffe0ba79fb9e637a608de54522c188f92`
  (coordinator-verified from disk at close).

## server.json

- Description exactly: "Track packages by number: current
  status, estimated delivery and full scan history, many
  carriers." — **98 characters** (measured).
- npm package entry carries `"version": "0.1.0"`; the OCI
  entry (`ghcr.io/cloudcorprecords/tracktrace:0.1.0`)
  carries **no** `version` field. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/tracktrace"`.

## Verification (coordinator's own re-runs, 2026-10-28)

1. `tsc --noEmit` (direct binary) → exit 0.
2. `node dist/cli.js --version` → `0.1.0`.
3. Full test suite (tsx --test) → **63 tests, 63 pass,
   0 fail** (registration-integrity deep-equal vs the
   fixture's 3 tools, plus the forwarding-client suites).
4. Parity (harness, tools-file mode vs
   `harness/fixtures/lastscanned-tools-original.json`) →
   **verdict PASS, parity 100.0** (3/3 identical,
   0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`, re-read by the coordinator at close.
