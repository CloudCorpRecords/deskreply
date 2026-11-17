# deskreply

The front desk, documented. deskreply is a small forwarding client for the hosted Larkella service — an AI receptionist that answers a business's calls and assists its owner. The tool list is served from your own machine using the captured definitions, and each call is sent upstream, with the service's answer returned exactly as it arrived.

All five of the service's captured tools are exposed, and every one of them is a pure content read: what the service is and does, the kinds of business it answers calls for, a deep look at one named industry with an example call, the questions business owners ask first, and how to try it free or talk to the team. Nothing is collected or submitted anywhere — `get_started` is documentation about signing up, not a signup.

## The tools

- **`about`** — What the service is and does: an AI receptionist for a business's callers and an assistant for its owner, covering features, who it's for, setup, privacy, the free test and how to sign up. No arguments.
- **`industries`** — The kinds of business the service answers calls for, what it does for each, and the page for the ones with their own page. No arguments.
- **`industry`** — One kind of business in detail, with an example call and that industry's common questions. Takes a required `industry` argument: `dental`, `home-services`, `clinics`, `law-firms` or `salons`.
- **`faq`** — The service's answers to the questions business owners ask first: AI disclosure, phone numbers, the pricing approach, and putting callers through to a person. No arguments.
- **`get_started`** — How to try it free (set up in about 2 minutes, then a 5-minute test call), sign up, or talk to the team. No arguments.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/deskreply mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "deskreply": {
      "command": "npx",
      "args": ["-y", "@reneturcios/deskreply", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/deskreply
docker run --rm ghcr.io/cloudcorprecords/deskreply about '{}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
deskreply about '{}'
deskreply industry '{"industry":"dental"}'
deskreply faq '{}'
deskreply --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `LARKELLA_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://larkella.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The Larkella registry record lists no headers for its remote, so deskreply never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `deskreply error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, deskreply repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

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
