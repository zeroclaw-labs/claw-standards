# Empirical probe summary — 2026-07-02

All runs against ZeroClaw built from workspace @ c6f49ca (plain `cargo build`, default features), isolated config (`probe-home/config.toml`, `ZEROCLAW_CONFIG_DIR` override), loopback port 43617. The user's live ZeroClaw instance was never touched.

## E1 — A2A discovery cards (files: `a2a-card-probe-alias.json`, catalog captured in transcript)

- `GET /.well-known/agents-card.json` (non-spec ZeroClaw catalog): serves a "ZeroClaw agents" catalog card with `protocolBinding: "catalog"` for itself + one JSONRPC interface per published alias. Self-describes as "Not a runnable agent".
- `GET /a2a/probe/.well-known/agent-card.json` (spec path per alias): valid card: name, description, `supportedInterfaces` [{url, protocolBinding: JSONRPC, protocolVersion: "1.0"}], version 0.8.2, capabilities `{streaming:false, pushNotifications:false, extendedAgentCard:false}` — capability flags honestly match reality; `skills: []`.
- Discovery GETs require no auth. Task POST requires bearer: unauthenticated POST → HTTP **401** (with `require_pairing = true`, the default).

## E2 — A2A method surface (file: `a2a-method-probes.txt`)

With valid bearer, POST /a2a/probe:
- `tasks/get`, `tasks/cancel`, `tasks/list`, `message/stream`, `tasks/subscribe`, `agent/getAuthenticatedExtendedCard` → ALL return JSON-RPC error **-32601 "Method not found: only message/send is supported on this build"** (server's own words).
- `message/send` with no model configured → **-32000 "Agent task failed: needs_quickstart: gateway has no model configured…"** (file: `a2a-message-send-probe.json`). No LLM provider key was available in this environment, so the full send→Task path was NOT exercised end-to-end. State this in the docs.

## E3 — a2a-tck official conformance run (files: `a2a-tck-must-tail.txt`, `a2a-tck-reports/{compatibility.json,compatibility.html,tck_report.html,junitreport.xml}`)

- TCK: a2aproject/a2a-tck @ 5996b79, `./run_tck.py --sut-host http://127.0.0.1:43617/a2a/probe --level must` (JSON-RPC transport auto-selected from card).
- **Auth caveat (standards-gap finding in its own right)**: the official TCK has NO mechanism to send auth headers (`run_tck.py` flags: sut-host/transport/level only; card fetch is bare `httpx.get` — see `a2a-tck-recon.md`). ZeroClaw's task endpoint is bearer-gated by default, so the run required `require_pairing = false` on loopback. The official A2A TCK cannot currently test an auth-mandatory A2A server as-deployed.
- **Headline result (per_requirement, MUST level): 15 PASS / 11 FAIL of 26 exercisable requirements = 57.7% MUST compatibility** (TCK's own `summary.must_compatibility`). 54 requirements SKIPPED (absent capabilities: streaming, push, task ops downstream of missing task store; some send-path prerequisites unavailable without a model), 34 NOT TESTED (TLS/signing/gRPC/etc. out of scope for this setup).
- PASS (15): CARD-DISC-001, CARD-STRUCT-001, CARD-PROTO-001, CARD-PROTO-002, BIND-FIELD-001, CORE-ERR-001, CORE-ERR-002, CORE-CAP-002, VER-SERVER-002, VER-SERVER-003, JSONRPC-SVC-001, JSONRPC-SVC-002, HTTP_JSON-URL-001, HTTP_JSON-URL-002, HTTP_JSON-QP-001 → card discovery/structure + JSON-RPC envelope basics are solid.
- FAIL (11): JSONRPC-ERR-001/002/003, JSONRPC-SSE-002, CORE-GET-001, CORE-CANCEL-001, CORE-CANCEL-002, CORE-SEND-002, CORE-HIST-002, CORE-MULTI-005, CORE-MULTI-006 → missing tasks/get + tasks/cancel, A2A-specific error-code mappings (TaskNotFound etc.), task history, multi-turn context handling.
- Raw pytest tail: 45 failed / 17 passed / 167 skipped / 6 errors (test-level counts; requirement-level aggregation above is the citable number). Confound to note honestly: some send-path test failures stem from the no-model -32000, not protocol logic.

## E4 — ACP initialize handshake (files: `acp-initialize-response.json`, `acp-bridge-stderr.log`)

- Path exercised: `zeroclaw-acp-bridge --pair-code <code>` (stdio) → gateway `ws://127.0.0.1:43617/acp` (i.e., the bridge-to-daemon architecture, not a native stdio agent).
- Sent `initialize` (protocolVersion 1). Response (verbatim in file): `protocolVersion: 1`; `agentInfo {name: "zeroclaw-acp", title: "ZeroClaw ACP", version: "0.8.2"}`; `agentCapabilities { loadSession: true, mcpCapabilities {http:false, sse:false}, promptCapabilities {audio:false, embeddedContext:false, image:false}, sessionCapabilities {close:{}, resume:{}} }`; `authMethods: []`; vendor extension `_meta.zeroclaw {maxSessions: 10, sessionTimeoutSecs: 3600}`.
- Bridge requires gateway pairing (refuses to start without token/pair-code) — first-hand confirmation of the paired-bridge design.

## E5 — skills-ref validation (files: `skills-ref-*.txt`, `skills-ref-summary.md`, synthetic probe under `synthetic-skill-test/`)

See ledger SKV-* entries (findings/ledger.json|md). Headline: skills-ref@0.1.5 **hard-fails** unknown top-level frontmatter fields (allowlist exactly: allowed-tools, compatibility, description, license, metadata, name). OpenClaw: 3/3 sampled bundled skills FAIL (top-level `homepage`; 33/52 of catalog carries it). NanoClaw: 2 richest skills FAIL (`triggers`, `version`); minimal name+description skill passes. ZeroClaw: repo-shipped .claude/skills mostly pass (plain frontmatter); one (github-pr) fails for missing frontmatter entirely; ZeroClaw's native schema (author/version/category/tags top-level) would hard-fail if emitted.

## Cleanup

Probe gateway stopped (port 43617 freed). Nothing written outside `.context/standards/`.
