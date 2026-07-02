# Pinned checkouts

Every citation in RATIONALE.md and spec/cip-1.0-draft.md is anchored to these SHAs, pinned 2026-07-02. Citation format: `repo@7sha:path#Lstart-Lend`.

| Repo | Remote | HEAD SHA at pin |
|---|---|---|
| ZeroClaw | github.com/zeroclaw-labs/zeroclaw | `c6f49ca6ee6828a8a5866e583432f2ec978e5866` |
| OpenClaw | github.com/openclaw/openclaw | `203a896b27c5c5191823138a4365578e0e77352d` |
| NanoClaw | github.com/nanocoai/nanoclaw | `aecad864e6371cb2a77ceaff8a38f9c4a8b71774` |
| a2a-tck (tool) | github.com/a2aproject/a2a-tck | `5996b79f9cefa6fc390980e383e358a66fb9e49e` |
| skills-ref (tool) | npm `skills-ref@0.1.5` | — |

Method:

- OpenClaw was read via a blobless clone with sparse checkout (`src packages docs skills extensions` + root files); NanoClaw and ZeroClaw as full checkouts. All code claims were verified against these checkouts, never against floating branch tips.
- GitHub issues are cited by number with captured JSON in `evidence/` (issues are not part of a git checkout).
- Empirical transcripts (validator runs, A2A probes, TCK reports, ACP handshake) are in `evidence/`, with the run methodology in `evidence/probe-summary.md`.
- The verified-claims ledger backing the docs' claim IDs (OC-*, NC-*, ZC-*, SKV-*) is `evidence/ledger.md`.

Claims about OpenClaw and NanoClaw are outside readings of these pins, pending maintainer correction — corrections supersede this material.
