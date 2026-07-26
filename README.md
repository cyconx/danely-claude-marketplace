# danely-claude-marketplace

Claude Code **marketplace** for the Danely agent experience (internal staff testing against the live Azure APIM perimeter).

This repo is the client-side packaging layer (skills + MCP wiring).  
**Platform truth** for every MCP client remains the generated Danely MCP surface and ADR-256 conventions — this marketplace must not become a second source of truth.

## Install (Claude Code)

```text
/plugin marketplace add cyconx/danely-claude-marketplace
/plugin install danely-platform@danely-claude-marketplace
```

That installs:

1. The `platform-conventions` skill  
2. Six HTTP MCP servers pointing at **APIM** (tenant perimeter) — OAuth via the DCR shim (no static bearer in the config)

On first use, Claude Code should prompt for CIAM (Entra External ID) login per connector. Use a **provisioned customer identity**, not workforce — workforce hits `/mcp/operator/{bundle}` (read-only) and 401s on the tenant authoring routes.

Enable marketplace auto-update under `/plugin` → Marketplaces if you want pulls on new tags.

## Claude settings — custom connectors (same six URLs)

If you prefer (or need) to register connectors manually — Claude **Settings → Connectors → Add custom connector** — add one connector per bundle:

| Bundle | URL |
|--------|-----|
| Content | `https://dnly-apim.azure-api.net/mcp/content` |
| Decision | `https://dnly-apim.azure-api.net/mcp/decision` |
| Policies | `https://dnly-apim.azure-api.net/mcp/policy` |
| Risks | `https://dnly-apim.azure-api.net/mcp/risk` |
| Workflow | `https://dnly-apim.azure-api.net/mcp/workflow` |
| Users | `https://dnly-apim.azure-api.net/mcp/user-admin` |

Do **not** paste a static API key/header for these URLs when using the live DCR/OAuth path — complete the browser login when prompted. After token refresh, reconnect via `/mcp` (or re-open the connector) so the session picks up new tokens.

### Identity prerequisite (admin, once per staff tester)

1. Create a **CIAM customer** user in External ID (`danely.ciamlogin.com`).  
2. Provision the Danely `Account` (`create_account` with that `oid`) and any needed grants via `/mcp/user-admin` as an admin.  
3. Staff sign in with that customer account when the connector OAuth runs.

Details: Cyconx repo `docs/operations/mcp-claude-code-onboarding.md` and `docs/operations/mcp-dcr-shim-configuration-guide.md`.

## Layout

```text
.claude-plugin/marketplace.json
plugins/
  danely-platform/
    .claude-plugin/plugin.json
    .mcp.json                          # six APIM bundle URLs
    skills/platform-conventions/SKILL.md
```

## MCP server names (plugin)

| Name in Claude | Path |
|----------------|------|
| `danely-content` | `/mcp/content` |
| `danely-decision` | `/mcp/decision` |
| `danely-policy` | `/mcp/policy` |
| `danely-risk` | `/mcp/risk` |
| `danely-workflow` | `/mcp/workflow` |
| `danely-user-admin` | `/mcp/user-admin` |

## Roadmap (experiment)

| Step | Content |
|------|---------|
| Now | `danely-platform` — conventions skill + APIM MCP wiring |
| Next | Per-bundle skills |
| Later | Hooks/scripts (pre-flight, Latest/Fixed retry, wait/poll) |

## Licence

Proprietary — Cyconx / Danely. Public so Claude Code can fetch the marketplace.
