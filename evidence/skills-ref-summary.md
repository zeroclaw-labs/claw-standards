# skills-ref validator — empirical runs (2026-07-02)

Validator: `skills-ref@0.1.5` (npm package `skills-ref`, the Agent Skills reference library).
Obtained via install path (a): `npm exec --yes skills-ref@latest -- --help` (worked on first try; exit 0).
For repeated runs, the same package was pinned locally: `npm install --prefix tools/skills-ref-install skills-ref@latest`, executed as `node tools/skills-ref-install/node_modules/skills-ref/dist/cli.js validate <path>`.
Options (b)/(c)/(d) were not needed.

Allowed top-level frontmatter fields per the validator's own error message:
`allowed-tools, compatibility, description, license, metadata, name`
Everything else is a HARD FAIL (exit 1), not a warning.

Pinned checkouts: openclaw@203a896, nanoclaw@aecad86, zeroclaw workspace@c6f49ca.

Skill-selection notes:
- OpenClaw: repo-root `skills/` are harness-shipped skills; richest frontmatter is 4 fields (`name`, `description`, `homepage`, `metadata` — vendor extensions nested under `metadata.openclaw`).
- NanoClaw: only `.claude/skills/` exists (no separate registry dir); richest are the two Qodo skills with `version`/`triggers`/`allowed-tools`.
- ZeroClaw: the repo ships NO harness-native skill files. Harness-native skills live in the user workspace (`~/.zeroclaw/workspace/skills/`, per `docs/book/src/tools/skills.md`); the runtime's own frontmatter schema (`crates/zeroclaw-runtime/src/skills/frontmatter.rs`) supports `license`, `author`, `version`, `category`, `tags`, but no skill file in the repo uses `author`/`version`/`category`/`tags` (grep over all SKILL.md: no hits). The skills validated here are the repo's `.claude/skills/` (Claude-Code dev-tooling skills, not ZeroClaw-harness skills). `github-pr` was deliberately included as a zero-frontmatter example.

## Results

| harness | skill path | pass/fail (exit) | key errors/warnings |
|---|---|---|---|
| openclaw | checkouts/openclaw/skills/weather | FAIL (1) | `Unexpected fields in frontmatter: homepage. Only allowed-tools, compatibility, description, license, metadata, name are allowed.` |
| openclaw | checkouts/openclaw/skills/1password | FAIL (1) | same: `Unexpected fields in frontmatter: homepage.` |
| openclaw | checkouts/openclaw/skills/peekaboo | FAIL (1) | same: `Unexpected fields in frontmatter: homepage.` |
| nanoclaw | checkouts/nanoclaw/.claude/skills/get-qodo-rules | FAIL (1) | `Unexpected fields in frontmatter: triggers, version.` (`allowed-tools` accepted) |
| nanoclaw | checkouts/nanoclaw/.claude/skills/qodo-pr-resolver | FAIL (1) | `Unexpected fields in frontmatter: triggers, version.` |
| nanoclaw | checkouts/nanoclaw/.claude/skills/setup | PASS (0) | `Valid skill` (name + description only) |
| zeroclaw | .claude/skills/zeroclaw | PASS (0) | `Valid skill` (name + description only) |
| zeroclaw | .claude/skills/changelog-generation | PASS (0) | `Valid skill` (name + description only) |
| zeroclaw | .claude/skills/github-pr | FAIL (1) | `SKILL.md must start with YAML frontmatter (---)` (file has no frontmatter at all) |
| synthetic | evidence/synthetic-skill-test/unknown-field-probe | FAIL (1) | `Unexpected fields in frontmatter: author.` |
| synthetic (control) | evidence/synthetic-skill-test/unknown-field-probe-control | PASS (0) | identical file minus `author: test` → `Valid skill` |

## Key findings

1. **Unknown top-level fields are a hard FAIL, not a warning.** The synthetic probe (valid `name` + `description` + `author: test`) exits 1 with `Unexpected fields in frontmatter: author.`; the control without `author` exits 0. There is no warning mode.
2. **OpenClaw's vendor-extension strategy (`metadata.openclaw`) passes** — `metadata` is on the allowlist — but its separate top-level `homepage` field fails every skill sampled (33 of the 52 repo skills carry `homepage` and would fail skills-ref as-is; the other 19, e.g. `github`, use only allowlisted fields).
3. **NanoClaw** skills that stick to `name`/`description` pass; the two skills carrying `version` + `triggers` fail (note `allowed-tools` itself is fine — it is spec-allowed).
4. **ZeroClaw** `.claude/skills` with plain `name`/`description` frontmatter pass; skills authored without any YAML frontmatter (`github-pr`, `github-issue`, `squash-merge` style) fail with `SKILL.md must start with YAML frontmatter (---)`. ZeroClaw's harness-native optional fields (`author`, `version`, `category`, `tags`) would all hard-fail skills-ref if placed top-level; only `license` overlaps with the spec allowlist. To be portable they must nest under `metadata`.

Per-run verbatim logs: `skills-ref-<harness>-<skill>.txt` in this directory.
