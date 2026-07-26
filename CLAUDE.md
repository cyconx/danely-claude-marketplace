# Maintaining danely-claude-marketplace

- Skills are **thin**: point at MCP `about` / `inspect_actions` / help corpus; do not fork platform docs.
- Bump `version` in both `plugins/*/…/plugin.json` and the matching entry in `.claude-plugin/marketplace.json` when shipping a change.
- Prefer conventional commits (`feat:`, `fix:`) if/when Release Please CI is added.
- Never put secrets or tenant-specific data in skills.
