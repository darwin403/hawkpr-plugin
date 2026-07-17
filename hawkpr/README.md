# HawkPR plugin

A suite of digital PR tools for Claude, delivered as a single plugin with one
skill per tool. No web UI required. The plugin is a thin layer over the hosted
HawkPR MCP server; heavy work (e.g. the coverage SERP sweep) runs asynchronously
in the HawkPR backend.

## Skills

- **`/coverage`** — track third-party news coverage of a pitched press release.
  Guided onboarding (create a campaign, run the initial hunt, confirm the tracked
  queries, preview the alert email, hand off a shareable URL), plus on-demand
  checks, listing, sample emails, and query edits. Routes all coverage actions
  from one entry point; also triggers on natural language ("track coverage for
  this press release", "any new coverage for <brand>?").

More DPR skills will be added to this plugin over time.

## MCP server

`.mcp.json` registers one server, `hawkpr`, defaulting to the production endpoint
`https://hawkpr.vercel.app/api/mcp`. Tools: `create_campaign`, `list_campaigns`,
`get_campaign`, `list_placements`, `set_queries`, `send_sample_email`,
`start_hunt`.

Override the backend with `HAWKPR_MCP_URL` (e.g. `http://localhost:3000/api/mcp`
for local development).

The MCP server, its Supabase/Resend integration, and the eve `coverage` agent
live in the private HawkPR app repo; this plugin repo contains no server code or
secrets.
