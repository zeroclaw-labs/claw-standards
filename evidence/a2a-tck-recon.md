# a2a-tck recon (before running)

Pinned: a2a-tck@5996b79 (shallow clone, tool only — not cited as protocol evidence)

Facts established by reading the TCK source:

1. **Invocation**: `uv venv && uv pip install -e . && ./run_tck.py --sut-host URL [--level must|should|may] [--transport ...]`. Reports land in `reports/`. (README.md)
2. **Card discovery**: fetches `{sut-host}/.well-known/agent-card.json` (tests/compatibility/conftest.py:97-101). So `--sut-host http://localhost:<port>/a2a/<alias>` matches ZeroClaw's per-alias card route `GET /a2a/{alias}/.well-known/agent-card.json`.
3. **RPC target**: transport clients are built from the card's `supportedInterfaces[].protocolBinding` + per-interface `url` (conftest.py:107-127). Card with no `supportedInterfaces` → hard fail.
4. **NO CLIENT AUTH SUPPORT**: `run_tck.py` has no auth/header flag (only --sut-host/--transport/--level/-v/--verbose-log/pytest passthrough, run_tck.py:104-138). Card fetch is bare `httpx.get(url)` (conftest.py:101). No env var or config path found for injecting Authorization headers (`grep -rn "Authorization|auth_header|AUTH_TOKEN|bearer" tck/` → only tck/requirements/auth.py, which is requirement *definitions*, mostly tagged NOT_AUTOMATABLE).
   → **Standards-gap observation for the docs**: the official A2A TCK cannot exercise an auth-mandatory A2A server; conformance testing implicitly assumes an unauthenticated (or ambient-auth) endpoint. The April-2026 "~0% conformance" cohort and this limitation are related facts.
5. **Consequence for our run**: if ZeroClaw's A2A RPC endpoint cannot be configured unauthenticated, the run needs a local header-injecting forward proxy (or a ZeroClaw config that allowlists the TCK), and the report must state the auth layer was bypassed for measurement.
6. TCK levels map to RFC 2119: MUST = hard fail, SHOULD = xfail, MAY = skipped unless capability declared. Use `--level must` for the core conformance claim.
