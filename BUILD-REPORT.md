# BUILD REPORT — deskreply

Clean-room rebuild of `com.larkella/larkella` v1.0.0
(remote-only MCP server at `https://larkella.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/deskreply`
0.1.0, registry name `io.github.CloudCorpRecords/deskreply`.

## Files (19-file factory set)

`package.json`, `server.json`, `Dockerfile`, `tsconfig.json`,
`.gitignore`, `package-lock.json`, `README.md`, `LICENSE`,
`CHANGELOG.md`, `src/index.ts`, `src/cli.ts`, `src/mcp.ts`,
`src/types.ts`, `src/forward.ts` (core), `test/forward.test.ts`,
`.github/workflows/ci.yml`, `.github/workflows/publish.yml`,
`.github/workflows/publish-container.yml`,
`.github/workflows/registry-publish.yml` — plus this report and the
`parity-out/` directory.

## Surface built

Full surface: **5 of 5 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `about` | *(none — no `required` key in the capture, served verbatim)* |
| `industries` | *(none — no `required` key in the capture, served verbatim)* |
| `industry` | `industry` |
| `faq` | *(none — no `required` key in the capture, served verbatim)* |
| `get_started` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema` (every tool in the capture carries a
`title`, served verbatim), and every kept `inputSchema` carries
`additionalProperties: false` exactly as captured. `industry.industry`
is the captured string enum (`dental`, `home-services`, `clinics`,
`law-firms`, `salons`) with the captured description "Which industry.".
The tool literals were generated programmatically from
`harness/fixtures/larkella-tools-original.json` by the script at
`/tmp/laneb6-deskreply-scratch/gen.py` (parse + re-serialise, splice
into `src/forward.ts`) — never retyped — and the generator's own
post-generation check, the registration-integrity test suite, and
the parity harness each confirmed the served literals deep-equal
the fixture's 5 tool objects, in order. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod. All five tools
are pure content reads; `get_started` is a documentation read about
signing up, not a signup, and nothing in the surface collects or
submits anything.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the absence of a `required` key on `about`,
`industries`, `faq` and `get_started`, the `industry` required field
on `industry`, and the `additionalProperties: false` markers.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://larkella.com/mcp`.
- Env override `LARKELLA_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LARKELLA_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` / `LARKELLA_TOKEN`
  variables present.
- Error rendering `deskreply error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed.

## Deviations

One, environment-caused, affecting how `dist/` and `parity-out/`
were produced (the artifacts themselves are verified faithful):

- The VM's exec write guard (the failure mode documented in
  AGENTS.md: exec writes intermittently fail `EPERM` and truncate
  the target to 0 bytes) blocked **every** `tsc` write to `dist/`
  across repeated `npm run build` attempts over ~25 minutes
  (TS5033 on all 15 outputs, dist files left at 0 bytes), and also
  blocked `parity.py`'s own writes of `parity.json` / `parity.md`.
  Other exec writes (the lockfile via npm, small probes)
  intermittently succeeded in the same window; tsc's writes to
  `dist/` never did.
- Workaround used: `dist/` was emitted with the same TypeScript
  compiler (5.9.3) and the project's own `tsconfig.json` through
  the compiler API, printed to stdout by
  `/tmp/laneb6-deskreply-scratch/emit.mjs` and placed with the file
  tool (which the guard does not affect). Every `dist/*.js` and
  `dist/*.d.ts` was then verified **byte-equal** (sha256) to the
  compiler's emit, and every `dist/*.js.map` verified **deep-equal**
  as parsed JSON (the file tool stores JSON with sorted keys, so
  the maps' key order differs from raw tsc emit; byte lengths and
  all values, including the `mappings` payloads, are identical).
  A final read-only re-check confirmed all 15 outputs match a
  fresh in-memory emit, and `node dist/cli.js --version` prints
  `0.1.0` from the restored dist.
- Parity was computed by the harness's own `build_report` /
  `render_md` code (imported from
  `/home/hatch/workspace/mcp-rebuild-100/harness/parity.py` by
  `/tmp/laneb6-deskreply-scratch/parity_drive.py`, same arguments
  as the prescribed CLI invocation) and the resulting report data
  was placed into `parity-out/` with the file tool; `parity.json`
  on disk holds exactly the harness report data (stored in the
  file tool's compact JSON form rather than `json.dump`'s
  pretty-printed form), and `parity.md` is the harness's rendered
  markdown verbatim. The restored `dist/` is byte-identical to
  the one the parity run measured.
- Because of the guard, a literal `npm run build` exit 0 could not
  be obtained in this environment; `npx tsc --noEmit` exit 0 plus
  the byte-verified compiler emit stand in for it. Everything else
  in the verification list below ran exactly as prescribed.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/deskreply`.
- Package entries: **128**.
- sha256: `8f1516a858ee264c3ec8b2079a3616f9d301468abfa5a05ed0662da54916b9d6`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "What an AI receptionist service does: overview, industries served, example calls and owner FAQs." — **96 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/deskreply:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/deskreply"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/deskreply`
  and its smoke test asserts the server name `deskreply`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. Build: `npm run build` blocked by the VM exec write guard (see
   Deviations); `dist/` produced via the TypeScript compiler API
   emit of the project tsconfig and verified — all 15 outputs
   (.js, .d.ts, .js.map × 5 modules) match a fresh in-memory emit
   (byte-equal for .js/.d.ts, deep-equal for .js.map).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **62 tests, 62 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 5
   tools at both the registry level and through an in-memory MCP
   client; plus session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity (harness `build_report` with the prescribed arguments —
   original fixture
   `/home/hatch/workspace/mcp-rebuild-100/harness/fixtures/larkella-tools-original.json`,
   rebuild `stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/deskreply/dist/cli.js mcp`,
   out `/home/hatch/workspace/mcp-rebuild-100/builds/deskreply/parity-out`)
   → **verdict PASS, parity 100.0** (5/5 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
