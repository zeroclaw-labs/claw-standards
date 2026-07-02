# Claw Interop Standards — Rationale

**From:** Jordan (ZeroClaw)
**For:** Peter Steinberger (OpenClaw), Gavriel Cohen (NanoClaw)
**Status:** DRAFT for your redlines. Companion spec: `spec/cip-1.0-draft.md`.

Method note: every claim below about OpenClaw or NanoClaw is an outside reading of pinned checkouts — openclaw@203a896, nanoclaw@aecad86, zeroclaw@c6f49ca, pinned 2026-07-02 — and stands corrected the moment either of you says otherwise. Citations are `repo@sha:path#Lx-Ly`; empirical transcripts live in `evidence/`. One correction has already happened and is preserved there (§4).

## §0 The ask

Three items:

1. **Adopt three existing standards as per-unit conformance badges.** Agent Skills (agentskills.io), ACP (Agent Client Protocol), A2A (Agent2Agent) — five units: U-SKILLS, U-ACP-SERVER, U-ACP-CLIENT, U-A2A-SERVER, U-A2A-CLIENT. Conformance is claimed per unit; there is no aggregate word, so no badge is hostage to a unit you scoped out — a deliberate non-implementation is recorded as `out-of-scope`, not `fail`. Phases (skills → servers → clients) are opted into separately, in writing; Phase 1 commits nobody to Phases 2–3. The destination is still all five units on every harness — that is what "our agents can all communicate" cashes out to; the phases are the pathway there, not a place to stop, and the opt-ins exist so nobody is dragged rather than so anybody camps. Any unit MAY be satisfied by a first-party component or a designated external one (e.g. a ClawHub-vetted plugin), verified in its own CI against a pinned harness release (spec §3.4).
2. **Conformance claims are CI-verified only.** A claim is a link to a passing run of the official tooling. Self-attestation is banned (§6).
3. **Redline it; sign-off is scoped.** The draft sits at zeroclaw-labs/claw-standards (name/org negotiable — I hold no attachment). Your approval is required for exactly two things: normative text that binds your harness, and any published gap/conformance claim naming it. Nothing additive binds a harness whose maintainer didn't approve it, and either of you can walk at any time — walking pulls every claim about your harness from the repo (§7).

Where we stand is measured, not asserted. On 2026-07-02 I ran the official validator, skills-ref@0.1.5, against all three repos. OpenClaw's bundled catalog fails today (33/52 skills, sole cause top-level `homepage` — SKV-OPENCLAW); ZeroClaw's native schema would hard-fail if emitted, and three of its dev skills fail now (SKV-ZEROCLAW); NanoClaw's eight shipped skills pass 8/8 — my first sample said otherwise because it validated the wrong plane, and the correction is preserved in `evidence/` (§4). ZeroClaw's gap list, §5, is the longest.

## §1 The layers, and why these three

Four layers, one surviving candidate each; the market already picked them.

- **Skills** — portable knowledge and procedures. agentskills.io, official validator `skills-ref` (run here at 0.1.5 — SKV-TOOLING).
- **MCP** — agent↔tools. Out of scope here.
- **ACP** — UI/editor↔agent, JSON-RPC 2.0 over stdio (protocol v1). No network surface required. (Disambiguation: ACP here means the Agent Client Protocol, agentclientprotocol.com — not the similarly named "Agent Communication Protocol".)
- **A2A** — agent↔agent over HTTP JSON-RPC, card at `/.well-known/agent-card.json` (v1.0.0, official TCK a2aproject/a2a-tck, pinned in PINS.md).

The standards-body work is done; none of this asks us to design a protocol. The repo's scope clause (§7) forbids it.

## §2 Why now

Three inputs matured within the last two quarters: the skills spec shipped a reference validator; A2A reached v1.0.0 with an official TCK; ACP stabilized at v1. Everyone claims interop; nearly nobody publishes verified evidence of it — I couldn't have either: before running the TCK I would have attested "A2A server: yes" for ZeroClaw, and the measured answer was 57.7% of MUSTs. Three harnesses publishing CI-verified per-unit conformance would, as far as I can tell, be the first. I give that window about two quarters.

## §3 Peter: the plugin path is the proposal — your plugin path, not mine

I am not asking you to reverse #6842, and I am not asking you to staff anything. Your close was a plugin-scope decision — "better suited for ClawHub/community plugin work" — grounded in VISION.md's lean-core lines, including the two that matter most: "If you build a plugin, host and maintain it in your own repository" (openclaw@203a896:VISION.md#L78) and plugin promotion "belongs in ClawHub, preferably under vetted org publishers for official plugins" (#L82). The spec's plugin provision (§3.4) now says exactly that: a unit is satisfied by a first-party component **or a designated external one** — and pointing `claw-conformance.json` at a passing TCK run of, say, @a2anet/openclaw-a2a-plugin (linked in your own #6842 thread, alongside openclaw-a2a-gateway) is an explicitly conformant route. Package name, hosting, maintenance: your calls, not spec text.

On ACP I have no ask: both roles are in-tree at my pin — acpx drives external ACP agents (extensions/acpx/README.md#L1-L5) and `openclaw acp` serves the agent-side session set over stdio (src/cli/acp-cli.ts#L14; src/acp/translator.ts#L250). Your whole Phase 1 is one afternoon PR, and I'll open it (§8): fix the stale "single-line keys only" docs note (contradicted by the shipped `YAML.parse` parser — docs/tools/skills.md#L248-L253 vs packages/markdown-core/src/frontmatter.ts#L56-L58), add a skills-ref CI job, and resolve `homepage` — preferably proposed upstream for the allowlist, with fallback to the `metadata.openclaw.homepage` form your docs already document as supported (docs/tools/skills.md#L257-L260). Your `metadata.openclaw` namespacing is already the vendor-extension pattern the spec adopts as the norm (SKV-OPENCLAW).

## §4 Gavriel: the sidecar shape, stated honestly

Reading your tree corrected my desk notes twice, and the second correction is the headline: my first validator sample took `.claude/skills/` — your operator plane, which your own code never loads (NC-SKILLS-1) — and called it your skills. The eight skills you actually ship in container/skills/ pass skills-ref **8/8** (evidence/skills-ref-nanoclaw-container-skills.txt). Your U-SKILLS gap is a CI job, nothing else. The two failing operator-plane files are vendored Qodo skills whose `triggers` field is load-bearing for Qodo's own tooling — the spec's remedy is upstream-or-exclude (SK-6), not "fix 2 skills."

On protocols, the honest topology — since ChannelAdapter registration is in-process in your always-running host and an editor cannot spawn the host (NC-ARCH-1): an editor-spawned stdio shim relaying to a host-side registered adapter over a named local transport, the same paired-bridge shape the spec already tolerates for ZeroClaw (AS-5). Sidecars enter at the channel-ingress plane; your session DBs stay bilateral host↔container and are not an integration surface for any other process — the single-writer invariant (src/session-manager.ts#L5-L11) is spec text, and §3.4(d) adds that a sidecar owns its own state: zero schema changes to `v2.db` or the session DBs. AS-2 now explicitly accepts turn-boundary `session/update` emission, because your agent-runner writes `messages_out` only on result (container/agent-runner/src/poll-loop.ts#L80) and nothing here proposes surgery inside your container image. What I won't answer for you: who answers `request_permission` (OQ-7 — including the admin-vs-editor-user identity mismatch and your by-design bypassPermissions), how an ACP session maps onto (agent_group, messaging_group, thread) (OQ-11), and what an A2A task even is on an async DM pipeline (OQ-12).

## §5 The gap ledger — ZeroClaw first

ZeroClaw's numbers, plainly:

- **A2A server is message/send-only.** Every other method returns -32601 — the server's own words: "Method not found: only message/send is supported on this build" (zeroclaw@c6f49ca:crates/zeroclaw-gateway/src/a2a.rs#L492-L502; probe E2). No task store; each send mints a pre-completed Task.
- **Official a2a-tck, MUST level: 15 pass / 11 fail of 26 exercisable requirements — 57.7%** (E3). Failures: tasks/get, tasks/cancel, A2A error mappings, task history, multi-turn context. Honesty notes: no model key was available, so the send path was not exercised end-to-end; the catalog card at `/.well-known/agents-card.json` is deliberately non-spec (a2a.rs#L45-L58).
- **ACP is a paired bridge to a running daemon, not a native stdio agent** (src/bin/zeroclaw-acp-bridge.rs#L126-L153; handshake transcript E4). No ACP client exists (ZC-ACP-2). No A2A client exists (ZC-A2A-2).
- **Native skills schema would hard-fail the validator if emitted** (crates/zeroclaw-runtime/src/skills/frontmatter.rs#L33-L61; SKV-ZEROCLAW).

My work items: frontmatter migration plus registry rewrite; a real task store to pass the TCK; an ACP client from zero; the catalog card proposed upstream as a formal A2A extension (ZC-3 — co-signing invited, never assumed). OpenClaw's and NanoClaw's lists are §3 and §4 — both PR-sized in Phase 1, both my outside reading; correct them.

## §6 Verified or it didn't happen

A conformance badge is a link to a passing CI run: `skills-ref validate` for U-SKILLS, a2a-tck for U-A2A-SERVER. The ban on self-attestation is argued entirely from our own house: before running the TCK I would have attested "A2A server: yes" for ZeroClaw. The measured answer was 57.7% of MUSTs.

ACP has no verifier I could find, and the spec doesn't pretend otherwise: U-ACP-* is **provisional** — no MUST hangs off `acp-handshake` until it exists at a pinned version and has run green against at least two independent implementations (candidates: `openclaw acp`, `zeroclaw-acp-bridge`). ZeroClaw writes v0.1; you two review, nothing more (W-N1). Second tooling gap, found empirically: a2a-tck cannot test auth-mandatory servers — no auth-header mechanism exists in `run_tck.py` (evidence/a2a-tck-recon.md) — so we propose that fix upstream too (W-N5).

## §7 Governance, deliberately boring

Scope clause: the repo pins versions and defines conformance; it never invents protocols — gaps go upstream. Sign-off is scoped to what actually needs you: normative text binding your harness, and published gap/conformance claims naming it. No additive change binds a harness whose maintainer didn't approve it — SHOULDs cannot ratchet into MUSTs around anyone. Exit clause: walk anytime; every claim about your harness leaves the repo with you. Lazy consensus, one-week comment windows, redlines on PRs, no meetings. Registry and upstream-or-sunset machinery were cut from v1: process arrives the day a second vendor extension actually collides (spec §5.3).

## §8 Timeline and next step

Phase 1 ~T+6 weeks, Phase 2 ~T+16, Phase 3 ~T+24 — every phase opted into separately, every date negotiable; the only fixed ordering is within the units you choose (skills before servers, servers before clients). The proposal arrives as code: at T+0 I open the Phase 1 PRs against all three repos myself — for you two that's a docs fix, a CI job, and (Peter) the `homepage` resolution. If the standard is real, it survives being a PR first. Next step: redline anything, async; corrections turn around same day.

<!-- NEEDS-VERIFICATION (updated 2026-07-02, post-reconciliation). Still to re-verify against primary sources before publishing: "no ACP verifier found" is asserted from absence; A2A "v1.0.0" as current version rests on the pinned TCK. Resolved this pass: the "April 2026 survey ~0%" figure cut everywhere (argument now rests on ZeroClaw's own measured 57.7%); steward attributions cut; NanoClaw skills restated from the fresh container/skills 8/8 run (evidence/skills-ref-nanoclaw-container-skills.txt — supersedes ledger SKV-NANOCLAW's "no separate harness skills dir" note, which NC-SKILLS-1 already contradicted); VISION.md#L78/#L82, docs/tools/skills.md#L257-L260, poll-loop.ts#L80, and the #6842 community-plugin links verified against pinned checkouts/evidence. Recipient identities (Peter Steinberger, Gavriel Cohen) confirmed directly by Jordan from the prior conversation. The uncited "MCP already converged across all three" claim was softened to "out of scope". -->
