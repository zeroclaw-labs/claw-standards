# CIP-1.0 draft: Claw Interop Profiles — conformance requirements and gap analysis

**Status:** DRAFT — for review by the OpenClaw, NanoClaw, and ZeroClaw maintainers. Nothing here is final until all three sign off.
**Repo:** `zeroclaw-labs/claw-standards` (placeholder URL; rename/move to a neutral org is on the table — see OQ-8).
**Editor of this draft:** Jordan (ZeroClaw). Corrections welcome as PRs or redlines.

A note on evidence discipline: every factual claim in this document about any harness's code is anchored to a pinned commit and cited as `repo@7sha:path#Lx-Ly`, or to an empirical run whose transcript is preserved. Claims about OpenClaw and NanoClaw are my outside reading of their trees as of the pin date (2026-07-02) and stand corrected the moment their maintainers say otherwise. ZeroClaw's gaps are listed first and are the longest list in this document — deliberately. Two of the three harnesses have a real skills gap against the official validator today (OpenClaw's bundled catalog; ZeroClaw's native schema and dev skills); an earlier revision of this draft claimed all three did, on a sample taken from NanoClaw's operator plane rather than the eight agent-facing skills it actually ships — which pass 8/8 (§8.1). The claim was corrected the way this document says claims must be: with a rerun whose transcript is preserved.

---

## 1. Status and versioning

1.1. This document defines **CIP-1.0** (Claw Interop Profiles, working name), a conformance profile over three existing external standards. CIP pins versions and defines conformance; it does not invent protocols. Gaps discovered in the underlying standards MUST be proposed upstream (see ZC-3, W-N5), not patched locally.

1.2. Versioning: CIP versions are `MAJOR.MINOR`. A normative change — MUST or SHOULD, original or additive — binds only the harnesses whose maintainers approved it; no accumulation of SHOULDs can ratchet into a MUST for a maintainer who did not sign the SHOULD. Any maintainer MAY withdraw at any time; withdrawal removes every published conformance claim and gap table naming that harness from the repo. (Governance details live in RATIONALE.md §7, not here.)

1.3. The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

## 2. Normative references (pinned)

| Ref | Standard | Pinned version | Verification tool |
|---|---|---|---|
| [SKILLS] | Agent Skills, agentskills.io | spec as enforced by `skills-ref@0.1.5` | `skills-ref validate` (npm `skills-ref`) |
| [ACP] | Agent Client Protocol, agentclientprotocol.com | protocol v1 (v2 pin is OQ-1) | `acp-handshake` (W-N1 — **provisional**; no official TCK found as of the pin date) |
| [A2A] | Agent2Agent Protocol, a2a-protocol.org | v1.0.0 | `a2a-tck` (pinned `a2aproject/a2a-tck@5996b79`) |
| [RFC2119], [RFC8174] | Requirement keywords | — | — |
| [RFC8615] | Well-Known URIs | — | — |

Pinned harness checkouts for all citations (full table in PINS.md):

| Harness | Pin |
|---|---|
| ZeroClaw | `zeroclaw@c6f49ca` (c6f49ca6ee6828a8a5866e583432f2ec978e5866) |
| OpenClaw | `openclaw@203a896` (203a896b27c5c5191823138a4365578e0e77352d) |
| NanoClaw | `nanoclaw@aecad86` (aecad864e6371cb2a77ceaff8a38f9c4a8b71774) |

Layering (context, non-normative): Skills = knowledge packaging; MCP = agent↔tools (out of scope — already converged); ACP = UI/editor↔agent over stdio; A2A = agent↔agent over HTTP.

## 3. Conformance model

### 3.1 Units

CIP defines five conformance units:

| Unit | One-line definition |
|---|---|
| **U-SKILLS** | Ships and consumes Agent Skills that pass `skills-ref validate`; vendor extensions nest under `metadata` |
| **U-ACP-SERVER** | Exposes the harness as an ACP v1 agent to editors/clients |
| **U-ACP-CLIENT** | Drives an external conforming ACP agent end-to-end |
| **U-A2A-SERVER** | Serves a spec agent card + the A2A core method set with a real task lifecycle |
| **U-A2A-CLIENT** | Consumes a remote agent card and completes send/poll/cancel against it |

### 3.2 Conformance is claimed per unit

A conformance claim names its unit: **"CIP-1.0 conformant: U-SKILLS"** (or any set of units). Per-unit claims are first-class and complete — a harness that chooses three of five units and passes them is fully conformant to the units it claims, not a partial anything. CIP-1.0 defines **no aggregate designation**: no word is reserved for five-of-five, there is no "core vs full" split, and no unit holds another unit's badge hostage. A deliberate decision not to implement a unit is recorded in the manifest as `out-of-scope` (§3.5, Appendix A) and reads as a scope decision, not a deficiency. If a protocol is worth adopting, it wins on merit.

### 3.3 Phased pathway (scheduling, not obligation)

The pathway orders the work for the units a harness chooses; it obligates no harness to any unit. Its destination, stated so nobody mistakes the shape of the effort: all five units on every participating harness. The phases sequence that outcome; the per-phase opt-ins (below) govern consent, not the goal.

| Phase | Units | Rough target | Exit criterion |
|---|---|---|---|
| 1 | U-SKILLS (participating harnesses) | ~T+6 weeks | `skills-ref validate` green in CI on every agent-facing skill (SK-6 scope); manifests published |
| 2 | U-ACP-SERVER + U-A2A-SERVER | ~T+16 weeks | `a2a-tck --level must` green in CI; `acp-handshake` green once it exists (AS-6, provisional) |
| 3 | U-ACP-CLIENT + U-A2A-CLIENT | ~T+24 weeks | Cross-harness matrix green: each client unit verified against another harness's server |

**Each phase is opted into separately, in writing (a one-line PR to the repo's participation file). Participating in Phase 1 carries zero commitment to Phases 2–3.** Dates are negotiable. The only fixed thing is ordering *within a harness's chosen units*: skills before servers, servers before clients (clients need servers to matrix-test against).

### 3.4 Plugin/sidecar provision

A unit MAY be satisfied by an optional, separately installable component (plugin, adapter, or sidecar) rather than core code, IF all of the following hold:

- (a) the component is **first-party** — maintained by the harness project itself (the same org or owner account as the harness repo; a personal-account repo qualifies) — **or designated**: an external component (e.g. a ClawHub-vetted or official-publisher plugin, per OpenClaw's own VISION.md: "plugin promotion belongs in ClawHub, preferably under vetted org publishers for official plugins" — openclaw@203a896:VISION.md#L82) that the harness's docs or manifest names as a supported way to get the capability;
- (b) it is documented in the harness's install docs as a supported way to get the capability;
- (c) it is CI-verified against a pinned harness release — either in the harness's own CI, **or in the component's own CI pinned against the harness release tag** — and the passing run URL feeds the same manifest (§3.5). Trunk-CI integration in the harness repo is NOT required;
- (d) the component owns its own state exclusively: it MUST NOT create tables in, or otherwise change the schema of, the harness's own data stores.

This provision is load-bearing, not a loophole. It is how NanoClaw's core stays small enough to read (~20.4k non-test lines — nanoclaw@aecad86, counted per NC-ARCH-1), and it is OpenClaw's stated architecture: "Core stays lean; optional capabilities should usually ship as plugins" (openclaw@203a896:VISION.md#L63-L66), followed one section later by "If you build a plugin, host and maintain it in your own repository" (#L78) and the ClawHub promotion path (#L82). When Peter closed openclaw#6842 as "better suited for ClawHub/community plugin work" (steipete, 2026-04-26, evidence/openclaw-issue-6842-comments.json), the thread already linked working external implementations — openclaw-a2a-gateway and @a2anet/openclaw-a2a-plugin (both in evidence/openclaw-issue-6842-comments.json). Under (a)–(d), designating such a component and pointing `claw-conformance.json` at its passing TCK run is a fully conformant route to U-A2A-SERVER. Whether the route is first-party or designated, and the component's name, hosting, and maintenance, are the maintainer's calls — not spec text.

### 3.5 The manifest: `claw-conformance.json`

A harness claiming any unit MUST publish a `claw-conformance.json` at a documented, discoverable path of its choosing — repo root, `.github/`, `docs/`, or a per-harness page in claw-standards — regenerated per release, recording per-unit status, the satisfying component (core/plugin/sidecar), the suite version, and a **public CI run URL** as evidence. The badge action (W-N2) takes the path as configuration. Schema sketch in Appendix A.

### 3.6 Self-attestation is banned

A conformance claim without a resolvable CI-run URL MUST be treated as non-conformant. One data point motivates this rule, from our own house: before running the official TCK, ZeroClaw would have attested "A2A server: yes." The measured answer was 57.7% of MUST-level requirements (§8.2). Advertised ≠ conformant, starting with us.

## 4. Unit requirements (normative)

### 4.1 U-SKILLS

| ID | Requirement |
|---|---|
| SK-1 | Every shipped/bundled SKILL.md MUST begin with a YAML frontmatter block. |
| SK-2 | Top-level frontmatter keys MUST be a subset of exactly: `allowed-tools`, `compatibility`, `description`, `license`, `metadata`, `name`. This is the allowlist `skills-ref@0.1.5` enforces with a hard fail — its own error text: "Only allowed-tools, compatibility, description, license, metadata, name are allowed." (evidence/skills-ref-synthetic-unknown-field-probe.txt; ledger SKV-UNKNOWN-FIELDS). |
| SK-3 | Vendor/harness-specific fields MUST nest under `metadata.<harness>` (e.g. `metadata.openclaw`, `metadata.zeroclaw`). OpenClaw's existing `metadata.openclaw` namespacing IS this pattern already — 48/52 of its bundled skills use it and the payloads pass the validator (openclaw@203a896:skills/github/SKILL.md#L4-L13; SKV-OPENCLAW). CIP §5 adopts OpenClaw's existing practice as the norm rather than inventing a new one. |
| SK-4 | Skill readers SHOULD ignore unknown `metadata` namespaces and SHOULD NOT fail on another vendor's keys. |
| SK-5 | Skill readers SHOULD NOT promote keys nested under `metadata` (or any ignored block) to top level. This is a real failure mode: ZeroClaw's flat parsers trim indentation, so an `author:` nested under an ignored `metadata:` block can be misread as a top-level field (zeroclaw@c6f49ca:crates/zeroclaw-runtime/src/skills/document.rs#L79-L82 with the misparse caveat per ledger ZC-SKILLS-1). |
| SK-6 | CI MUST run `skills-ref validate` (pinned version) over every **agent-facing** skill the harness ships — bundled catalogs, container-mounted skill planes, template skills — and fail the build on any nonzero exit. Repo-development/operator skill planes (e.g. a repo's own `.claude/skills/` used by maintainers' tooling, not loaded by the harness — ledger NC-SKILLS-1) are OUT of unit scope and MAY be validated separately without affecting unit status. Vendored third-party skills whose extra fields are load-bearing for the third party's own tooling follow **upstream-or-exclude**: propose the frontmatter fix to the vendor, or exclude the skill from the validated set — never rewrite another project's files in place. |

Note on SK-4/SK-5's keyword level: these are reader-side behaviors with no verifier in §8.4 today, and a MUST that can only be self-attested would violate §3.6's own rule — so they are SHOULDs. A harness that delegates all frontmatter reading to a runtime that ignores unknown fields (NanoClaw's design: core opens no SKILL.md — NC-SKILLS-1) satisfies both by construction. If reader-behavior fixtures land in the shared suite (a skill carrying foreign `metadata` namespaces that must load cleanly), these upgrade to MUST at the next MINOR.

### 4.2 U-ACP-SERVER

Reference points: the two in-tree implementations among us. ZeroClaw's server handles `initialize`, `session/new`, `session/load`, `session/resume`, `session/close`, `session/prompt`, `session/stop`, `session/cancel`, with `session/event`/`session/update` as notifications, and returns METHOD_NOT_FOUND otherwise (zeroclaw@c6f49ca:crates/zeroclaw-channels/src/orchestrator/acp_server.rs#L398-L422). OpenClaw's `openclaw acp` bridge implements `initialize`, `authenticate`, `session/new`, `session/load`, `session/list`, `session/resume`, `session/close`, `session/set_mode`, `session/prompt`, `session/cancel`, plus `session/update` notifications (openclaw@203a896:src/acp/translator.ts#L250-L819; wire names for initialize and the session/{new,load,list,resume,close,prompt,update} set per src/acp/protocol-schema.test.ts#L26-L124 — `authenticate`/`session/set_mode`/`session/cancel` rest on the translator's SDK method handlers, OC-ACP-1). The MUST set below is a core subset of the intersection; the SHOULD set draws from the union.

| ID | Requirement |
|---|---|
| AS-1 | JSON-RPC 2.0; `initialize` MUST advertise protocol v1 and capability flags that match implemented behavior. |
| AS-2 | MUST handle: `initialize`, `session/new`, `session/prompt`, `session/cancel`. MUST emit at least one `session/update` notification carrying the agent's output before the prompt turn completes; **turn/message-boundary emission satisfies this MUST.** Finer-grained streaming (chunk- or tool-call-level updates mid-turn) is a SHOULD. Rationale: a message-granularity harness must be able to conform via a boundary component without core surgery — NanoClaw's agent-runner writes `messages_out` only on result (nanoclaw@aecad86:container/agent-runner/src/poll-loop.ts#L80), and requiring mid-turn events would reach inside its container image. |
| AS-3 | MUST return JSON-RPC -32601 (method not found) for unrecognized methods. |
| AS-4 | SHOULD handle: `session/load`, `session/list`, `session/resume`, `session/close`, `session/set_mode`, `authenticate`; SHOULD emit `session/request_permission` where the harness has a human-approval concept to map it to. |
| AS-5 | A native stdio entry point is RECOMMENDED. A bridge-to-daemon architecture MAY satisfy this unit IF starting it is a single documented command and any pairing/auth bootstrap is documented as part of the unit's install path. (ZeroClaw is the motivating case: its stdio path is `zeroclaw-acp-bridge`, a stdin/stdout↔WebSocket relay to the gateway's `/acp` endpoint, gated on gateway pairing — zeroclaw@c6f49ca:src/bin/zeroclaw-acp-bridge.rs#L126-L153 — not a self-contained stdio agent.) |
| AS-6 | **Provisional — no MUST here yet.** The intended verifier is the shared `acp-handshake` suite (W-N1), which does not exist at this draft's date; no official ACP TCK was found either. Until `acp-handshake` ships at a pinned version and has run green against at least two independent implementations that are not its own seed transcript (candidates: `openclaw acp`, `zeroclaw-acp-bridge`), U-ACP-* status in any manifest MUST be reported as `provisional` (Appendix A), and a `provisional` unit MUST NOT be presented as a conformance claim (§3.6). This row upgrades to a CI MUST at the next MINOR after the suite exists. |
| AS-7 | Vendor extensions in `initialize` results MUST nest under `_meta.<harness>` (ZeroClaw already does: `_meta.zeroclaw {maxSessions, sessionTimeoutSecs}` — evidence/acp-initialize-response.json; §5). |

### 4.3 U-ACP-CLIENT

| ID | Requirement |
|---|---|
| AC-1 | MUST spawn or attach to a conforming ACP v1 server, complete `initialize` negotiation, create a session, send a prompt, and consume the `session/update` stream to completion. |
| AC-2 | MUST respond to `session/request_permission` (a fixed policy such as documented auto-deny is acceptable; hanging is not). |
| AC-3 | MUST support `session/cancel`. |
| AC-4 | Verified by driving the `acp-handshake` reference transcripts (provisional until W-N1 ships — same terms as AS-6), and (Phase 3) at least one of the other two harnesses' U-ACP-SERVER implementations end-to-end. |

Existing reference: OpenClaw's first-party `acpx` plugin ("Official ACP runtime backend for OpenClaw… lets OpenClaw run external coding harnesses through the Agent Client Protocol" — openclaw@203a896:extensions/acpx/README.md#L1-L5), with `/acp` chat commands and `sessions_spawn({runtime:"acp"})` (src/agents/tools/sessions-spawn-tool.ts#L312). Neither ZeroClaw (ledger ZC-ACP-2: zero matches for any client-side ACP surface) nor NanoClaw (NC-ACP-1: zero ACP references in tree) has one.

### 4.4 U-A2A-SERVER

| ID | Requirement |
|---|---|
| A2S-1 | MUST serve an agent card at `<base>/.well-known/agent-card.json` [RFC8615] with valid structure and `supportedInterfaces`. |
| A2S-2 | Card capability flags MUST match implemented behavior. (ZeroClaw's flags do — `{streaming:false, pushNotifications:false, extendedAgentCard:false}`, honest to a fault; probe E1, evidence/a2a-card-probe-alias.json.) |
| A2S-3 | MUST implement `message/send`, `tasks/get`, and `tasks/cancel` with a persistent-enough task store to make `tasks/get` meaningful after `message/send` returns. |
| A2S-4 | MUST implement the A2A v1.0 8-state task lifecycle and the A2A-specific error mappings (e.g. TaskNotFound), not just generic JSON-RPC errors. |
| A2S-5 | SHOULD implement `message/stream` + `tasks/subscribe` (SSE). Push-notification config CRUD is MAY. |
| A2S-6 | CI MUST run `a2a-tck --level must` (pinned) with zero FAIL among exercisable MUST requirements, and publish the report. |
| A2S-7 | Deployments MAY require auth on the RPC endpoint; the discovery card SHOULD be fetchable unauthenticated. Until `a2a-tck` supports auth-header injection (it has none today — §8.2, W-N5), CI MAY use a loopback no-auth test profile, and the manifest MUST record that the auth layer was bypassed for measurement. |

### 4.5 U-A2A-CLIENT

| ID | Requirement |
|---|---|
| A2C-1 | MUST fetch and parse a remote agent card and select a transport by `protocolBinding`. |
| A2C-2 | MUST complete `message/send`, poll `tasks/get` to a terminal state, and issue `tasks/cancel`. |
| A2C-3 | MUST honor card capability flags (e.g. MUST NOT call `message/stream` against `streaming:false`). |
| A2C-4 | Phase 3 verification: green against at least one other harness's U-A2A-SERVER. |

No harness among the three has any A2A client today (ZC-A2A-2; OC-A2A-1; NC-A2A-1).

## 5. Extension policy

5.1. Vendor extensions MUST live in the designated slots: `metadata.<harness>` in skills frontmatter, `_meta.<harness>` in ACP payloads, card `extensions` in A2A. Unknown vendor namespaces MUST be ignored by readers.

5.2. This is codifying observed practice, not inventing it: OpenClaw already namespaces under `metadata.openclaw` in 48/52 bundled skills (OC-SKILLS-3), and ZeroClaw already emits `_meta.zeroclaw` in its ACP initialize result (evidence/acp-initialize-response.json). CIP adopts the counterpart's existing pattern as the norm in each slot.

5.3. **Deferred machinery.** An extension-key registry and an upstream-or-sunset rule were drafted here and cut from v1: process arrives the day a second vendor extension actually collides, not before. Until then, §5.1's slot rule is the only normative extension requirement. The one live case is ZeroClaw's own: its deliberately non-spec multi-agent catalog card at `/.well-known/agents-card.json` (plural; zeroclaw@c6f49ca:crates/zeroclaw-gateway/src/a2a.rs#L45-L58, marked "Deliberately NOT the spec's singular agent-card.json" in its own source) — ZeroClaw will propose it upstream to the A2A project as a formal extension (ZC-3) rather than keep a private fork of the spec. Co-signatures from the other maintainers are invited, not assumed.

## 6. Gap analysis

Sizes (S = days, M = weeks, L = a quarter-scale item) are given **for ZeroClaw only** — I don't estimate other people's repos. OpenClaw and NanoClaw tables are **outside readings** of pinned checkouts as of 2026-07-02, offered for correction by their maintainers — not authoritative statements about their roadmaps or effort.

### 6.1 ZeroClaw (first, and the longest list)

Stated plainly before the table: ZeroClaw's A2A server accepts `message/send` and nothing else — `tasks/get`, `tasks/cancel`, `tasks/list`, `message/stream`, `tasks/subscribe`, and `agent/getAuthenticatedExtendedCard` all return -32601 `"Method not found: only message/send is supported on this build"` (the server's own words; probe E2). On the official `a2a-tck` MUST level it scores **15 pass / 11 fail of 26 exercisable requirements = 57.7%**. Its ACP story is a paired bridge to a daemon, not a native stdio agent. It has no ACP client and no A2A client. Its native skills schema would hard-fail the official validator if ever emitted top-level. None of this is currently gated in CI.

| Unit | Current state | Evidence | Gap | Size |
|---|---|---|---|---|
| U-SKILLS | Native schema defines 8 top-level fields; `author`/`version`/`category`/`tags`/`slash_options` are non-spec top-level keys; `metadata`, `allowed-tools`, `compatibility` are unparsed (dropped via `_ => {}`). Two divergent flat parsers (loader vs document reader) with unequal field coverage. Misparse hazard: keys indented under an ignored `metadata:` block can be read as top-level. 3 of 9 repo `.claude/skills` have no frontmatter at all and fail the validator outright. | zeroclaw@c6f49ca:crates/zeroclaw-runtime/src/skills/frontmatter.rs#L33-L61, #L4-L9; mod.rs#L1438-L1457; document.rs#L176-L186 (ZC-SKILLS-1); SKV-ZEROCLAW | Single spec-compliant parser; parse `metadata`/`allowed-tools`/`compatibility`; migrate flat fields to `metadata.zeroclaw` (Appendix B); migrate the external skills registry; `skills-ref` in CI | M |
| U-ACP-SERVER | Server handles the v1 method set (§4.2) and advertises honest capabilities; but the stdio path is `zeroclaw-acp-bridge`, a stdin/stdout↔WebSocket relay to the gateway `/acp` endpoint, refusing to start without pairing token/pair-code. Confirmed live (E4). | zeroclaw@c6f49ca:crates/zeroclaw-channels/src/orchestrator/acp_server.rs#L398-L422; src/bin/zeroclaw-acp-bridge.rs#L126-L153 (ZC-ACP-1); evidence/acp-initialize-response.json | Meet AS-5 (native stdio entry, or one-command documented bridge bootstrap); `acp-handshake` in CI once it exists (AS-6 provisional) | S–M |
| U-ACP-CLIENT | None. No `agent-client-protocol` dependency, no ClientSideConnection, no ACP subprocess spawning; the bridge dials ZeroClaw's own gateway only. | ZC-ACP-2 (grep results, all empty) | Full client from zero | L |
| U-A2A-SERVER | `message/send` only; every other method -32601. No task store: each send runs one synchronous turn and returns a freshly minted Task (new UUID, state `"completed"`). TCK MUST level: 15/26 pass (57.7%). Failing requirement IDs (11): JSONRPC-ERR-001, JSONRPC-ERR-002, JSONRPC-ERR-003, JSONRPC-SSE-002, CORE-GET-001, CORE-CANCEL-001, CORE-CANCEL-002, CORE-SEND-002, CORE-HIST-002, CORE-MULTI-005, CORE-MULTI-006. Card discovery/structure and JSON-RPC envelope basics pass (15 IDs, evidence/probe-summary.md E3 and evidence/a2a-tck-reports/compatibility.json). Non-spec plural catalog path served alongside the spec card. | zeroclaw@c6f49ca:crates/zeroclaw-gateway/src/a2a.rs#L492-L502, #L540-L557, #L45-L58 (ZC-A2A-1); probes E1–E3 | Task store; `tasks/get`/`tasks/cancel`; 8-state lifecycle; A2A error mappings; multi-turn context; history; TCK in CI; catalog → upstream extension proposal (ZC-3) | L |
| U-A2A-CLIENT | None. All A2A code lives in gateway+config crates; config comment reserves "outbound client config" as future work. | ZC-A2A-2; zeroclaw@c6f49ca:crates/zeroclaw-config/src/multi_agent.rs#L198-L200 | Full client from zero | M–L |

Caveat recorded honestly: the `message/send` happy path was NOT exercised end-to-end in our probes (no model key in the probe environment; the send returned -32000 needs_quickstart — E2). The 57.7% figure is the TCK's own requirement-level aggregation; some send-path test failures are confounded by the no-model condition (noted in E3).

### 6.2 OpenClaw (outside reading — please correct)

OpenClaw's ACP position is the strongest of the three: **both roles are in-tree.** Its real gaps are skills validation, A2A, and conformance CI — not ACP.

| Unit | Current state | Evidence | Gap |
|---|---|---|---|
| U-SKILLS | Docs claim agentskills.io compliance but describe a "single-line keys only" parser that no longer exists — the shipped parser is full `YAML.parse` with multi-line support, and unknown top-level fields are silently ignored. 33/52 bundled skills carry top-level `homepage` and fail `skills-ref` as-is (3/3 sampled failed, sole cause `homepage`); the `metadata.openclaw` payloads pass. | openclaw@203a896:docs/tools/skills.md#L248-L253 (OC-SKILLS-1); packages/markdown-core/src/frontmatter.ts#L56-L58 (OC-SKILLS-2); OC-SKILLS-3; SKV-OPENCLAW | Preferred route: propose `homepage` upstream for the Agent Skills allowlist — a third of OpenClaw's catalog and at least one other harness clearly wants it, and §5.3's principle says upstream first. Fallback if upstream declines: migrate `homepage` (+ `user-invocable` where used) under `metadata.openclaw` — a path OpenClaw's docs already document as supported ("Also supported via `metadata.openclaw.homepage`", docs/tools/skills.md#L257-L260). Either way: fix the stale docs note; `skills-ref` in CI at a pinned version (inference: no validator gate can currently be in effect, since 33/52 shipped skills fail it) |
| U-ACP-SERVER | In-tree: `openclaw acp` serves an AgentSideConnection over stdio backed by the Gateway WS, with the full session method set (§4.2 list). | openclaw@203a896:src/cli/acp-cli.ts#L14; src/acp/translator.ts#L250-L819 (OC-ACP-1) | `acp-handshake` in CI once it exists (AS-6 provisional); likely near-pass on day one |
| U-ACP-CLIENT | In-tree: official `acpx` plugin drives external ACP agents (default Claude adapter; arbitrary agents via config); `/acp` commands; `sessions_spawn({runtime:"acp"})`. | openclaw@203a896:extensions/acpx/README.md#L1-L5; index.ts#L10-L21 (OC-ACP-1) | CI verification only |
| U-A2A-SERVER | None native — no agent card, no A2A JSON-RPC anywhere in src/packages/extensions; all "a2a" hits are the internal same-gateway `sessions_send` flow. #6842 closed 2026-04-26 as ClawHub/community plugin scope (quoted §3.4), and the thread links working external implementations (openclaw-a2a-gateway; @a2anet/openclaw-a2a-plugin, tested end-to-end per its author). The plugin API demonstrably covers the needed surface: `registerHttpRoute` + `registerTool`. | OC-A2A-1 (grep results); evidence/openclaw-issue-6842-comments.json; OC-PLUGIN-1 | A first-party **or designated external** plugin (§3.4). Designating an existing community plugin and pointing `claw-conformance.json` at its passing TCK run is an explicitly conformant route; so is building first-party. Which route — and whether at all — is Peter's call |
| U-A2A-CLIENT | None (same absence evidence). | OC-A2A-1 | Client side of the same (first-party or designated) component |

### 6.3 NanoClaw (outside reading — please correct)

NanoClaw's product plane passes U-SKILLS validation today — its Phase 1 is a CI job, nothing more. Its protocol route is sidecars at the channel-ingress plane.

| Unit | Current state | Evidence | Gap |
|---|---|---|---|
| U-SKILLS | Core parses ZERO frontmatter: discovery is directory-name-only (container/skills/ mounted RO, symlinks into `~/.claude/skills`); all SKILL.md semantics are delegated to the embedded Claude Code runtime in-container. Unknown fields are silently ignored because NanoClaw never opens the file. Validator, run against the plane NanoClaw actually ships to agents: **all 8 container/skills pass, 8/8 exit 0** (agent-browser, frontend-engineer, onecli-gateway, self-customize, slack-formatting, vercel-cli, welcome, whatsapp-formatting). An earlier draft sampled `.claude/skills/` — the repo-dev/operator plane NanoClaw's code never loads (NC-SKILLS-1) — and its 2 failures are vendored Qodo files whose `triggers` field is load-bearing for Qodo's own tooling: out of SK-6 scope (upstream-or-exclude), and not NanoClaw's to rewrite. | nanoclaw@aecad86:src/container-runner.ts#L346-L426; src/group-skills.ts#L4-L8 (NC-SKILLS-1); evidence/skills-ref-nanoclaw-container-skills.txt | Add `skills-ref` CI over container/skills/ (+ template skills). That is the whole Phase 1 |
| U-ACP-SERVER | None — zero ACP references in tree (clean absence; the only 'acp' matches are base64 noise in lockfiles). | NC-ACP-1 | Sidecar, with the topology stated plainly: an **editor-spawned stdio shim relaying to a host-side registered ChannelAdapter over a named local transport** — the same paired-bridge shape AS-5 already tolerates for ZeroClaw. (ChannelAdapter registration is in-process in the always-running host and the editor cannot spawn the host — NC-ARCH-1 — so a shim/relay pair is the honest shape, not "architecturally identical to existing adapters.") The sidecar enters at the channel-ingress plane; the session DBs stay bilateral host↔container and are NOT an integration surface for any other process. AS-2's turn-boundary allowance exists precisely so NanoClaw's result-time `messages_out` write conforms without agent-runner changes. Session→entity-model mapping is undesigned: OQ-11. `request_permission` mapping is a candidate pending OQ-7 |
| U-ACP-CLIENT | None. | NC-ACP-1 | Sidecar spawning external ACP agents; lowest priority (Phase 3) |
| U-A2A-SERVER | None — no agent card, no `/.well-known/agent.json`, no A2A JSON-RPC. Warning for readers: NanoClaw's branch names and code use "a2a" for its internal agent-to-agent SQLite routing module; that is unrelated to the A2A protocol and must not be cited as adoption. | NC-A2A-1 | A2A sidecar at the same ingress plane (ingest MUST present as a ChannelAdapter, never write `messages_in` directly — single-writer invariant, "concurrent writers corrupt the DB": nanoclaw@aecad86:src/session-manager.ts#L5-L11; NC-ARCH-1). Per §3.4(d) the sidecar owns its task store exclusively — its own SQLite file, zero schema changes to `v2.db` or the session DBs. The task↔session/message lifecycle mapping is unsketched: OQ-12 |
| U-A2A-CLIENT | None. | NC-A2A-1 | Client half of the sidecar |

## 7. Work items

### ZeroClaw (ZC-#)

| ID | Item | Unit | Phase |
|---|---|---|---|
| ZC-1 | Replace the two flat frontmatter parsers with one spec-compliant parser (parse `metadata`/`allowed-tools`/`compatibility`; kill the metadata-promotion misparse); migrate `author`/`version`/`category`/`tags`/`slash_options` to `metadata.zeroclaw` (Appendix B); migrate the zeroclaw-skills registry; `skills-ref` in CI | U-SKILLS | 1 |
| ZC-2 | A2A task store + `tasks/get` + `tasks/cancel` + 8-state lifecycle + A2A error mappings + history/multi-turn context — i.e., clear the 11 named failing TCK requirements; `a2a-tck --level must` in CI | U-A2A-SERVER | 2 |
| ZC-3 | Author the A2A multi-agent catalog extension proposal, submit it upstream to the A2A project, and move `/.well-known/agents-card.json` behind it (co-signatures from the other maintainers invited, not assumed) | U-A2A-SERVER / §5.3 | 2 |
| ZC-4 | Meet AS-5: native stdio ACP entry or one-command bridge bootstrap; write `acp-handshake` v0.1 (W-N1) and run it in CI | U-ACP-SERVER | 2 |
| ZC-5 | ACP client from zero | U-ACP-CLIENT | 3 |
| ZC-6 | A2A client (card fetch, send, poll, cancel) | U-A2A-CLIENT | 3 |
| ZC-7 | Publish `claw-conformance.json` + badge action | all | 1 |

### OpenClaw (OC-#) — proposed, Peter's call

| ID | Item | Unit | Phase |
|---|---|---|---|
| OC-1 | Fix the stale "single-line keys only" docs note (contradicted by the shipped parser and by the docs' own multi-line example) | U-SKILLS | 1 |
| OC-2 | Resolve the 33/52 top-level `homepage` skills: preferred, propose `homepage` upstream for the Agent Skills allowlist; fallback if upstream declines, migrate under `metadata.openclaw` (already documented as supported — docs/tools/skills.md#L257-L260). `skills-ref` in CI at a pinned version either way | U-SKILLS | 1 |
| OC-3 | A2A via §3.4: either designate an external plugin (the #6842 thread already links openclaw-a2a-gateway and @a2anet/openclaw-a2a-plugin) and point the manifest at its passing TCK run, or build first-party (`registerHttpRoute` card + JSON-RPC endpoint, `registerTool` for outbound). Route, naming, hosting: Peter's calls | U-A2A-* | 2–3 |
| OC-4 | `acp-handshake` in CI for `openclaw acp` (server) and `acpx` (client) — likely the cheapest green badges in this document | U-ACP-* | 2 |
| OC-5 | Publish `claw-conformance.json` | all | 1 |

### NanoClaw (NC-#) — proposed, Gavriel's call

| ID | Item | Unit | Phase |
|---|---|---|---|
| NC-1 | Add `skills-ref` CI over container/skills/ and template skills (8/8 pass today — evidence/skills-ref-nanoclaw-container-skills.txt). The vendored Qodo skills in the operator plane are out of SK-6 scope: upstream-or-exclude, not NanoClaw's to rewrite | U-SKILLS | 1 |
| NC-2 | ACP sidecar: editor-spawned stdio shim ↔ host-side registered ChannelAdapter over a named local transport (topology per §6.3). The approvals primitive as `request_permission` backend is a **candidate mapping pending OQ-7**, not settled; session→entity-model mapping pending OQ-11 | U-ACP-SERVER | 2 |
| NC-3 | A2A sidecar at the ingress plane; owns its own task store per §3.4(d) — zero schema changes to `v2.db` or the session DBs; ingest via adapter, never direct `messages_in` writes; lifecycle mapping pending OQ-12 | U-A2A-SERVER | 2 |
| NC-4 | Client halves of NC-2/NC-3 | U-*-CLIENT | 3 |
| NC-5 | Publish `claw-conformance.json` | all | 1 |

### Neutral repo (W-N#) — shared, in claw-standards

| ID | Item | Rationale |
|---|---|---|
| W-N1 | `acp-handshake` golden-transcript suite | No official ACP TCK found; U-ACP-* stays **provisional** (AS-6) until this ships pinned and runs green against ≥2 independent implementations. Seed transcripts: ZeroClaw's captured initialize (E4) + OpenClaw's protocol-schema test vectors. ZeroClaw writes v0.1; the others review, nothing more |
| W-N2 | `claw-conformance.json` schema (Appendix A) + badge GitHub Action | Makes §3.5/§3.6 mechanical |
| W-N5 | Contribute auth-header support upstream to `a2a-tck` | Empirical blocker: the official TCK has no mechanism to send auth headers (`run_tck.py` exposes only sut-host/transport/level plus verbosity/pytest passthrough — no auth flag; the card fetch is a bare `httpx.get` — evidence/a2a-tck-recon.md). It therefore cannot test any auth-mandatory A2A server as deployed. Shared upstream work, not a local workaround |

(W-N3, the extension-key registry, and W-N4, the co-signed catalog extension, were cut: W-N3 is deferred per §5.3 until a real collision exists, and W-N4 was a ZeroClaw upstream proposal wearing a shared label — it is now ZC-3, endorsable but not a deliverable of the standard.)

## 8. Verification and CI

This section records the exact commands run for this draft and their actual results. Everything below is reproducible from the pinned checkouts and the transcripts in `evidence/`.

### 8.1 Skills — `skills-ref@0.1.5` (2026-07-02)

Command (per skill directory):

```
npm exec --yes skills-ref@latest -- validate <skill-dir>
# pinned for batch runs: node tools/skills-ref-install/node_modules/skills-ref/dist/cli.js validate <skill-dir>
```

Results (full table in evidence/skills-ref-summary.md; verbatim logs per run):

| Harness | Sampled | Result |
|---|---|---|
| OpenClaw | skills/weather, 1password, peekaboo (bundled catalog — agent-facing) | 3/3 FAIL (exit 1) — sole cause top-level `homepage`; 33/52 of the catalog carries it. `metadata.openclaw` payloads pass |
| NanoClaw | container/skills/* — all 8 agent-facing skills (product plane) | **8/8 PASS (exit 0)** — evidence/skills-ref-nanoclaw-container-skills.txt. An earlier draft sampled `.claude/skills/` (operator/dev plane, not loaded by NanoClaw code — NC-SKILLS-1): 2 vendored-Qodo FAILs (`triggers`/`version`), 1 PASS. That sample is out of SK-6 scope and superseded for unit status |
| ZeroClaw | .claude/skills/zeroclaw, changelog-generation, github-pr (dev plane; no agent-facing skills live in-repo) | 2 PASS, 1 FAIL ("SKILL.md must start with YAML frontmatter (---)") |
| Synthetic probe | valid skill + `author: test` vs identical control | FAIL vs PASS — isolates the hard-fail behavior to a single unknown field |

Empirical findings folded into the spec: the validator HARD-FAILS unknown top-level fields (no warn mode), and the allowlist is exactly the six fields in SK-2. **Corrected headline:** OpenClaw's shipped catalog fails the validator today, and ZeroClaw's native schema (`author`/`version`/`category`/`tags` top-level — SKV-ZEROCLAW) would hard-fail if ever emitted; NanoClaw's shipped plane passes 8/8 and needs only the CI gate. The first version of this table sampled NanoClaw's operator plane and drew the wrong headline; the correction run is preserved, which is the point of §3.6.

### 8.2 A2A — official TCK + method probes (2026-07-02)

Target: ZeroClaw built from `c6f49ca`, isolated config, loopback port 43617.

```
./run_tck.py --sut-host http://127.0.0.1:43617/a2a/probe --level must    # a2a-tck@5996b79
```

Result: **15 PASS / 11 FAIL of 26 exercisable MUST requirements = 57.7%** (the TCK's own `summary.must_compatibility`); 54 skipped (absent capabilities), 34 not tested (TLS/gRPC/etc. out of scope). Failing IDs as listed in §6.1; the 15 passing IDs are recorded in evidence/probe-summary.md (E3) and evidence/a2a-tck-reports/compatibility.json. Reports preserved: evidence/a2a-tck-reports/.

Auth caveat, stated as run conditions: the TCK cannot send auth headers, so the run required `require_pairing = false` on loopback; production ZeroClaw defaults to bearer-gated task POSTs (401 unauthenticated — E1). This is the W-N5 blocker observed first-hand, and precisely the situation A2S-7 exists for.

Method probes (valid bearer, POST /a2a/probe — evidence/a2a-method-probes.txt): `tasks/get`, `tasks/cancel`, `tasks/list`, `message/stream`, `tasks/subscribe`, `agent/getAuthenticatedExtendedCard` each returned `{"error":{"code":-32601,"message":"Method not found: only message/send is supported on this build"}}`. `message/send` with no model configured returned -32000 needs_quickstart; the full send path was not exercised end-to-end in this environment.

### 8.3 ACP — initialize handshake (2026-07-02)

```
zeroclaw-acp-bridge --pair-code <code>     # stdio → ws://127.0.0.1:43617/acp
# then on stdin: {"jsonrpc":"2.0","id":0,"method":"initialize","params":{"protocolVersion":1}}
```

Response (verbatim, evidence/acp-initialize-response.json): `protocolVersion: 1`; `agentInfo {name:"zeroclaw-acp", title:"ZeroClaw ACP", version:"0.8.2"}`; `agentCapabilities {loadSession:true, mcpCapabilities{http:false,sse:false}, promptCapabilities{audio:false,embeddedContext:false,image:false}, sessionCapabilities{close:{},resume:{}}}`; `authMethods: []`; `_meta.zeroclaw {maxSessions:10, sessionTimeoutSecs:3600}`. The bridge refused to start without a pairing token/pair-code — first-hand confirmation of the paired-bridge design (AS-5 caveat). This transcript is the first golden transcript for W-N1.

### 8.4 CI obligations (normative summary)

| Suite | Gate | Publishes to manifest |
|---|---|---|
| `skills-ref validate` (pinned) over all agent-facing skills (SK-6 scope) | SK-6, per release | U-SKILLS |
| `acp-handshake` (W-N1 — **provisional**: suite does not exist yet; until it ships and runs green against ≥2 independent implementations, U-ACP-* reports `provisional` and carries no claim) | AS-6 / AC-4 | U-ACP-SERVER, U-ACP-CLIENT |
| `a2a-tck --level must` (pinned SHA) | A2S-6 | U-A2A-SERVER |
| Cross-harness client matrix | AC-4 / A2C-4, Phase 3 | U-*-CLIENT |

## 9. Milestones (non-binding)

The proposal ships as reviewable diffs, not homework: at T+0 ZeroClaw opens the Phase 1 PRs itself — OC-1+OC-2 against openclaw, NC-1 against nanoclaw, ZC-1 in zeroclaw — and the ask reduces to redlining or merging them. Phases bind only harnesses that opt in (§3.3).

| When | Milestone |
|---|---|
| T+0 | Maintainers redline this draft; Phase 1 PRs opened by ZeroClaw against all three repos; OQ list triaged; repo naming settled (OQ-8) |
| T+2w | W-N1 seed transcripts + W-N2 schema merged; Phase 1 PRs in review |
| T+6w | Phase 1 exit for participating harnesses: skills-ref green; manifests live |
| T+16w | Phase 2 exit (opt-in): a2a-tck green for participating U-A2A-SERVERs; acp-handshake green if W-N1 has shipped, else U-ACP stays provisional |
| T+24w | Phase 3 exit (opt-in): cross-harness client matrix green; first per-unit conformance claims at client level |

## 10. Open questions

| ID | Question | Status |
|---|---|---|
| OQ-1 | ACP v1 vs emerging v2 schema: when to bump the CIP pin? | OPEN — proposal: pin v1 for CIP-1.0; revisit at CIP-1.1 |
| OQ-2 | Skills `metadata` is spec-typed as a string map; structured vendor values (ZeroClaw `slash_options`, OpenClaw `command-dispatch`) don't fit natively. JSON-encoded string values (Appendix B) vs upstream ask for structured metadata? | OPEN — empirical note: skills-ref@0.1.5 accepted OpenClaw's structured-JSON-under-`metadata` blocks (SKV-OPENCLAW), so both encodings validate today; the string-encoded form is the conservative reading |
| OQ-3 | Exact a2a-tck mandatory test list? | **RESOLVED empirically** — at ZeroClaw's capability profile the MUST level exercised 26 requirements (15 pass/11 fail, IDs in §6.1); 54 skipped as capability-gated, 34 not tested (TLS/signing/gRPC). The mandatory set is capability-relative, which is why A2S-6 says "zero FAIL among exercisable MUST requirements" rather than a fixed count |
| OQ-4 | Does skills-ref fail or warn on unknown top-level fields? | **RESOLVED empirically** — hard FAIL, exit 1, no warn mode; allowlist exactly the six SK-2 fields (synthetic probe + control, SKV-UNKNOWN-FIELDS). This is what makes "all three of us fail today" a fact rather than a reading |
| OQ-5 | A2A card `extensions` mechanism semantics for the ZC-3 catalog proposal | OPEN — blocks ZC-3 drafting, not Phase 1/2 |
| OQ-6 | (Added post-plan, from the TCK run.) How should conformance runs treat auth-mandatory deployments until W-N5 lands upstream — loopback no-auth profile (A2S-7), header-injecting proxy, or wait? | OPEN — A2S-7 encodes the interim answer; maintainers should confirm |
| OQ-7 | Who answers `session/request_permission` in NanoClaw's headless containers? Two real difficulties, not one: (a) containers run `bypassPermissions` by design, so there is no tool-level event to forward in the first place (NC-PERM-1); (b) identity mismatch — the approvals primitive DMs an *admin* over a messaging channel at human latency, while ACP `request_permission` addresses the human *at the editor*, who may not be an admin at all. The approvals primitive is a candidate backend only; the NC-2 parenthetical is explicitly pending this OQ | OPEN — Gavriel's question to answer, not mine |
| OQ-8 | Repo/org naming: `zeroclaw-labs/claw-standards` is a placeholder; rename/move to a neutral org offered | OPEN — Peter's call on "Claw" adjacency |
| OQ-9 | Spec license: CC-BY-4.0 prose + Apache-2.0 schemas/test stubs? | OPEN — decide at repo creation |
| OQ-10 | NanoClaw's registry-branch distribution (`git show origin/<branch>:<path>`) is per-file and layout-correct only when SKILL.md enumerates every file (NC-SKILLS-2); does U-SKILLS need a distribution-integrity clause, or is that out of scope? | OPEN — current draft says out of scope (CIP validates the skill as installed) |
| OQ-11 | Define the ACP-session→NanoClaw entity-model mapping. NanoClaw sessions are (agent_group, messaging_group, thread) with a per-session container; `session/new` from an anonymous local editor names none of those. Which agent group does the editor get? Who authorizes it — unknown_sender_policy, a dedicated messaging_group per editor session, something else? This mapping is the actual design work behind NC-2; the gap table must not imply it exists | OPEN — blocks NC-2 design, Gavriel's call |
| OQ-12 | Define the A2A task↔NanoClaw session/message lifecycle mapping for NC-3: what is a task (one message round-trip? a session?), when does it reach `completed` (when the host's delivery poll drains `messages_out`?), and where the 8 states land on an async DM pipeline. Sidecar-owned state per §3.4(d) | OPEN — blocks NC-3 design, Gavriel's call |

---

## Appendix A — `claw-conformance.json` schema sketch

Draft JSON Schema (2020-12); to be finalized as W-N2.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/zeroclaw-labs/claw-standards/schema/claw-conformance.schema.json",
  "title": "claw-conformance.json",
  "type": "object",
  "required": ["cip_version", "harness", "generated_at", "units"],
  "properties": {
    "cip_version": { "type": "string", "examples": ["1.0-draft"] },
    "harness": {
      "type": "object",
      "required": ["name", "repo", "release"],
      "properties": {
        "name":    { "type": "string" },
        "repo":    { "type": "string", "format": "uri" },
        "release": { "type": "string", "description": "tag or commit SHA these results apply to" }
      }
    },
    "generated_at": { "type": "string", "format": "date-time" },
    "units": {
      "type": "object",
      "propertyNames": {
        "enum": ["U-SKILLS", "U-ACP-SERVER", "U-ACP-CLIENT", "U-A2A-SERVER", "U-A2A-CLIENT"]
      },
      "additionalProperties": { "$ref": "#/$defs/unit" }
    }
  },
  "$defs": {
    "unit": {
      "type": "object",
      "required": ["status"],
      "$comment": "mode, suite, and evidence are REQUIRED whenever status is \"pass\" or \"fail\" (a measured result needs its run); out-of-scope and not-implemented need only status.",
      "properties": {
        "status": {
          "enum": ["pass", "fail", "provisional", "not-implemented", "out-of-scope"],
          "description": "out-of-scope = deliberate maintainer scope decision, distinct from fail/not-implemented (§3.2); provisional = the unit's verifier has not shipped/pinned yet (AS-6) — not claimable per §3.6. There is no aggregate designation; per-unit status is the product."
        },
        "mode":   { "enum": ["core", "plugin", "sidecar"], "description": "§3.4 provision" },
        "component": { "type": "string", "description": "package/binary satisfying the unit when mode != core" },
        "suite":  { "type": "string", "examples": ["skills-ref@0.1.5", "a2a-tck@5996b79 --level must", "acp-handshake@0.1"] },
        "evidence": { "type": "string", "format": "uri", "description": "public CI run URL. REQUIRED — no URL, no claim (§3.6)" },
        "results": {
          "type": "object",
          "properties": {
            "pass": { "type": "integer" },
            "fail": { "type": "integer" },
            "skip": { "type": "integer" }
          }
        },
        "notes": { "type": "string", "description": "e.g. \"auth layer bypassed for measurement per A2S-7\"" }
      }
    }
  }
}
```

## Appendix B — worked frontmatter migration (ZeroClaw as the guinea pig)

ZeroClaw's native schema is the flattest and most divergent of the three, so it goes first. The "before" below is a synthetic skill in exactly the shape ZeroClaw's runtime parser defines (zeroclaw@c6f49ca:crates/zeroclaw-runtime/src/skills/frontmatter.rs#L33-L61) and its docs describe; no repo file currently carries these fields (SKV-ZEROCLAW), which is itself part of the finding — the divergent schema lives in the parser and the external registry, not in validated files.

**Before (ZeroClaw-native flat shape) — FAILS skills-ref:**

```yaml
---
name: weather-brief
description: Fetch and summarize today's weather for the configured city. Use when the user asks for a weather briefing.
license: MIT
author: jordanthejet
version: 1.2.0
category: utilities
tags:
  - weather
  - daily
---
```

Validator behavior (verified by the synthetic probe, SKV-UNKNOWN-FIELDS): exit 1 — `Unexpected fields in frontmatter: author, category, tags, version. Only allowed-tools, compatibility, description, license, metadata, name are allowed.` Note `license` survives: it is the only ZeroClaw-native optional field on the spec allowlist.

**After (spec shape, vendor fields under `metadata.zeroclaw`) — PASSES:**

```yaml
---
name: weather-brief
description: Fetch and summarize today's weather for the configured city. Use when the user asks for a weather briefing.
license: MIT
metadata:
  zeroclaw: '{"author":"jordanthejet","version":"1.2.0","category":"utilities","tags":["weather","daily"]}'
---
```

Migration notes:

1. `name`, `description`, `license` are unchanged — already spec fields.
2. `author`, `version`, `category`, `tags` move under `metadata.zeroclaw` as a JSON-encoded string value (the conservative reading of the spec's string-map type — OQ-2). Empirically, skills-ref@0.1.5 also accepts structured JSON nested under `metadata` (OpenClaw's 48 bundled `metadata.openclaw` blocks draw no validator complaint — SKV-OPENCLAW; its sampled failures cite only top-level `homepage`), so a structured block is a valid alternative encoding pending OQ-2 resolution.
3. `slash_options` (ZeroClaw's remaining non-spec field) follows the same route into `metadata.zeroclaw`, with its array JSON-encoded.
4. Reader-side obligations (ZC-1): the runtime must parse `metadata` (today it drops it via a `_ => {}` arm), must stop promoting indented keys out of ignored blocks (SK-5 — today an `author:` nested under `metadata:` can be misread as top-level), and must accept-and-ignore other vendors' namespaces (SK-4).
5. Rollout: dual-read (flat + `metadata.zeroclaw`) for one release with a deprecation warning on flat fields, migrate the zeroclaw-skills registry, then drop flat parsing. The external registry is in a separate repo and was not validated in this pass; its migration is part of ZC-1, not assumed done.

<!-- NEEDS-VERIFICATION (fact-check pass, 2026-07-02; updated after maintainer-objection reconciliation): claims that cannot be verified against the pinned checkouts, ledger, or evidence/ transcripts. Re-verify against primary sources before publishing:
- §2/§4.2/W-N1: "no official ACP TCK found" is asserted from absence — softened accordingly, but no local evidence can prove a negative about the ACP ecosystem.
- OQ-1: "emerging v2 schema" for ACP is an external-roadmap claim not locally verifiable.
- A2A "v1.0.0" as the current spec version rests on the pinned a2a-tck (card probes show protocolVersion "1.0").
Resolved in the reconciliation pass (2026-07-02): steward attributions dropped from §2; the "April 2026 survey ~0%" figure cut everywhere (§3.6 now argues from ZeroClaw's own 57.7%); NanoClaw §6.3/§8.1 restated from a fresh 8/8-pass run over container/skills (evidence/skills-ref-nanoclaw-container-skills.txt) — note this supersedes ledger SKV-NANOCLAW's "no separate harness skills dir" note, which NC-SKILLS-1 already contradicted; VISION.md#L78 and #L82 quotes verified verbatim in the pinned checkout; docs/tools/skills.md#L257-L260 "Also supported via metadata.openclaw.homepage" verified; @a2anet/openclaw-a2a-plugin and openclaw-a2a-gateway references verified in evidence/openclaw-issue-6842-comments.json; poll-loop.ts#L80 "On result: write messages_out" verified.
Previously verified (no flag needed): Appendix B "Before" fails pinned skills-ref exactly as quoted, "After" passes; the three no-frontmatter .claude/skills fail with "SKILL.md must start with YAML frontmatter (---)"; the 8-state lifecycle matches the pinned a2a-tck bindings.
-->

