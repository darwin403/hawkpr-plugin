# HawkPR plugin (marketplace)

Claude/Cursor marketplace for **HawkPR** — skills + MCP config for the hosted
backend. Published to [darwin403/hawkpr-plugin](https://github.com/darwin403/hawkpr-plugin).

**Source of truth:** this `plugin/` directory in the [hawkpr](https://github.com/darwin403/hawkpr) app repo. Edit here; publish with `./scripts/publish-plugin.sh`.

## Install (end users)

### Claude Code

```
/plugin marketplace add darwin403/hawkpr-plugin
/plugin install hawkpr@hawkpr-plugin
```

### Cursor

Dashboard → Plugins → Add Marketplace → `darwin403/hawkpr-plugin`, then enable
the plugin in **Customize**.

## Local dev

From the hawkpr repo root:

```bash
./scripts/link-plugin-local.sh
export HAWKPR_MCP_URL=http://localhost:3000/api/mcp
npm run dev
```

Reload Cursor/Claude, enable the **hawkpr** skill and MCP server.

## Publish

```bash
./scripts/publish-plugin.sh
```

Pushes `plugin/` to the public marketplace repo via `git subtree`.
