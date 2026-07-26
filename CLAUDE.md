# Maintaining danely-claude-marketplace

- Skills are **thin**: point at MCP `about` / `inspect_actions` / help corpus; do not fork platform docs.
- When shipping a change, bump `version` in **all three** places to the same value:
  1. `.claude-plugin/marketplace.json` (top-level `version`)
  2. `.claude-plugin/marketplace.json` → `plugins[].version` for the plugin entry
  3. `plugins/*/…/plugin.json` → `version`
- Prefer conventional commits (`feat:`, `fix:`) if/when Release Please CI is added; a CI assert that those three strings match is the natural lockstep guard.
- `.mcp.json` points at **prod APIM only** (`dnly-apim.azure-api.net`) — not Aspire, not a per-env switch.
- Never put secrets or tenant-specific data in skills.
