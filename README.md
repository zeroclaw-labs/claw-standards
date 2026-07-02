# claw-standards

**Status: DRAFT — under review by the maintainers of OpenClaw, NanoClaw, and ZeroClaw. Nothing here is agreed yet.**

Conformance profiles for agentic harnesses ("Claws"). This repo pins versions and subsets of three existing standards and defines how a harness proves conformance. It does not define protocols; protocol gaps go upstream.

| Standard | Layer | Governed by |
|---|---|---|
| [Agent Skills](https://agentskills.io) | packaged knowledge | Anthropic (open spec) |
| [ACP — Agent Client Protocol](https://agentclientprotocol.com) | client/UI ↔ agent | Zed Industries (Apache 2.0) |
| [A2A — Agent2Agent](https://a2a-protocol.org) | agent ↔ agent | Linux Foundation (Apache 2.0) |

## Documents

- [`RATIONALE.md`](./RATIONALE.md) — why these three standards, why now.
- [`spec/cip-1.0-draft.md`](./spec/cip-1.0-draft.md) — CIP-1.0 draft: conformance units, requirements, per-harness gap analysis, work items.

## How to respond

Redline anything: open an issue or PR against either document. The per-harness gap tables are outside readings of each codebase (SHA-pinned, evidence-linked) — corrections from maintainers supersede them.

The repo name and org are provisional. Moving to a neutral org, or renaming, is on the table — see RATIONALE §7.

## Licensing

Prose (RATIONALE.md, spec/): [CC-BY-4.0](./LICENSE). Schemas and test code: [Apache-2.0](./LICENSE-CODE).
