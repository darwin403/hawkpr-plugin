# HawkPR plugin

A Claude Code marketplace for **HawkPR** — a suite of digital PR tools that run
inside a Claude conversation, no web UI required. It ships a single plugin,
`hawkpr`, a consolidated toolkit whose first skill is coverage tracking.

## Install

```
/plugin marketplace add darwin403/hawkpr-plugin
/plugin install hawkpr@hawkpr-plugin
```

Then use it — e.g. `/coverage`, or just paste a press release and ask to track
its coverage.

## What's inside

| Plugin | Skills | What it does |
| ------ | ------ | ------------ |
| `hawkpr` | `/coverage` | Track third-party news coverage of a pitched press release end to end. More DPR skills will be added here over time. |

The plugin is metadata + skill instructions + a pointer to the hosted HawkPR MCP
server (`https://hawkpr.vercel.app/api/mcp`). The server and app are maintained
separately; this repo contains no application code or secrets.

To point the plugin at a different backend (staging, local), set `HAWKPR_MCP_URL`
in your environment before launching Claude.
