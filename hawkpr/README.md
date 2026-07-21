# HawkPR plugin

A suite of digital PR tools, delivered as a single plugin with one skill per
tool. No web UI required. The plugin is a thin layer over the hosted HawkPR MCP
server; heavy work (e.g. the coverage SERP sweep) runs asynchronously in the
HawkPR backend.

## Skills

- **`/coverage`** — track third-party news coverage of a pitched press release:
  both coverage it has already received and new pickups going forward.
  Guided onboarding (create a campaign, start the hunt, poll briefly until the
  agent saves tracked queries, show queries, offer notification emails, then hand
  off status with a shareable URL while the sweep continues in the background),
  plus on-demand checks, listing, sample emails, and query edits. Routes all
  coverage actions from one entry point; also triggers on natural language
  ("track coverage for this press release", "any new coverage for <brand>?").

More DPR skills will be added to this plugin over time.

## MCP server

`mcp.json` / `.mcp.json` registers one server, `hawkpr`, defaulting to the
production endpoint `https://hawkpr.vercel.app/api/mcp`. Tools: `create_campaign`,
`list_campaigns`, `get_campaign`, `list_placements`, `set_queries`,
`set_notification_emails`, `send_sample_email`, `start_hunt`.

Campaign notification emails are an optional add-on, offered after tracked queries
are shown and before the status handoff, configured via `set_notification_emails`
(the plugin proposes the PR media contact with confirmation when present). When
set, new confirmed placements trigger automatic Resend alerts.

Override the backend with `HAWKPR_MCP_URL` (e.g. `http://localhost:3000/api/mcp`
for local development).

The MCP server, Supabase/Resend integration, and eve `coverage` agent live in
the parent hawkpr app repo; this directory is metadata and skill instructions
only — no server code or secrets.
