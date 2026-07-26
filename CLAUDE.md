# Maintaining danely-claude-marketplace

- **Branching:** GitFlow-style — see [docs/git-flow.md](docs/git-flow.md). Cut `feature/*` from `develop`; open PRs to `develop`; promote with `develop` → `main`. Do not push directly to `main` or `develop`.
- Skills are **thin**: point at MCP `about` / `inspect_actions` / help corpus; do not fork platform docs.
- When shipping a change, bump `version` in **all three** places to the same value:
  1. `.claude-plugin/marketplace.json` (top-level `version`)
  2. `.claude-plugin/marketplace.json` → `plugins[].version` for the plugin entry
  3. `plugins/*/…/plugin.json` → `version`
- Prefer conventional commits (`feat:`, `fix:`) if/when Release Please CI is added; a CI assert that those three strings match is the natural lockstep guard.
- `.mcp.json` points at **prod APIM only** (`dnly-apim.azure-api.net`) — not Aspire, not a per-env switch.
- Never put secrets or tenant-specific data in skills.
