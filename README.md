# danely-claude-marketplace

Claude Code **marketplace** for the Danely agent experience.

This repo is the client-side packaging layer (skills, later hooks/scripts/agents).  
**Platform truth** for every MCP client (including source-less ones) remains the generated Danely MCP surface and [ADR-256](https://github.com/cyconx/Cyconx) conventions — this marketplace must not become a second source of truth.

## Install (Claude Code)

```text
/plugin marketplace add cyconx/danely-claude-marketplace
/plugin install danely-platform@danely-claude-marketplace
```

Enable auto-update for the marketplace under `/plugin` → Marketplaces if you want pulls on new tags.

## Layout

```text
.claude-plugin/marketplace.json     # registry Claude Code reads
plugins/
  danely-platform/                  # shared conventions (v0.1)
    .claude-plugin/plugin.json
    skills/platform-conventions/SKILL.md
```

## Roadmap (experiment)

Aligned with the Danely agent-experience plan:

| Step | Plugin / content |
|------|------------------|
| Now | `danely-platform` — conventions skill |
| Next | Per-bundle skills (`danely-decision`, …) |
| Later | Hooks + scripts (inspect→write pre-flight, Latest/Fixed retry, wait/poll) |
| Later | Optional `.mcp.json` profiles for local Aspire / deployed MCP |

## MCP servers

This marketplace does **not** yet ship MCP connection config. Point Claude Code at your existing `danely-*` MCP servers (Aspire local or deployed) as you do today; the skill assumes those tools are available.

## Licence

Proprietary — Cyconx / Danely. Public for Claude Code marketplace fetch only.
