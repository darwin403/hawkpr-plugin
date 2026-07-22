# HawkPR — Team Guide

A plain-English guide to using HawkPR inside Claude. No technical knowledge needed. You talk to
Claude in normal English — the examples below are things you can literally type.

**What it does:** HawkPR watches the web for third-party news coverage of your pitched press
releases. Point it at a release and it sweeps for coverage that's already gone live **and** keeps
watching for new pickups, all from a single chat command.

**The golden rule:** HawkPR never emails a client or changes anything on its own. It sets up
coverage tracking for you, and only turns on alert emails if **you** ask it to and confirm the
address. You're always in control of what goes out.

---

## What you can do with it

The plugin adds one skill to Claude, with more digital-PR tools coming over time.

| Skill | What it does for you |
| --- | --- |
| **Coverage** (`/coverage`) | Tracks third-party news coverage of a pitched press release — finds coverage already published and watches for new pickups a few times a day, across the Australian news cycle. Gives you a shareable dashboard link, and optional email alerts when new coverage lands. |

---

## Install

Do this once. Everything happens in the **Claude desktop app** — no typing of commands.

HawkPR has **two** pieces to add: the **connector** (the HawkPR service Claude talks to) and the
**plugin** (the `/coverage` skill itself). Add the connector first.

### Step 1 — Add the HawkPR custom connector

HawkPR runs on its own hosted service. Because it's a StudioHawk service (not one of Claude's
built-in connectors like Gmail), you add it as a **custom connector** with its address.

1. In the left sidebar, open **Customize → Connectors**.
2. Click **Add custom connector** (near the bottom of the connector list).
3. Fill in:
   - **Name:** `HawkPR`
   - **Remote MCP server URL:**

     ```
     https://hawkpr.vercel.app/api/mcp
     ```

4. Click **Add**. HawkPR shows **Connected** — no sign-in or password needed.
5. Open the **HawkPR** connector and set its tools to **Always allow**, so you're not asked
   for permission on every use. (No per-tool settings? Just click **Always allow** the first time
   each tool runs.)

That's the connector done. It's what lets Claude create campaigns and check coverage for you.

### Step 2 — Add the HawkPR plugin

1. In the left sidebar, open **Customize → Plugins**.
2. In the **Personal plugins** section, click the **"+"** → **Add marketplace** → **Add from a
   repository**.
3. Paste this and let it sync:

   ```
   https://github.com/darwin403/hawkpr-plugin
   ```

4. Click **Browse plugins**, find **hawkpr**, and click **Install**.

**That's it — installation is done.** You now have the `/coverage` skill available in your chat.
Head to **Usage** below to start using it.

> **On Claude Code (CLI) or Cursor instead?** The plugin bundles the HawkPR connector for you, so
> you can skip Step 1. In Claude Code run `/plugin marketplace add darwin403/hawkpr-plugin`
> then `/plugin install hawkpr@hawkpr-plugin` — and to skip the per-tool prompts, add
> `"mcp__hawkpr"` to `permissions.allow` in your `.claude/settings.json`. In Cursor, add the
> marketplace `darwin403/hawkpr-plugin` under **Dashboard → Plugins**, then enable **hawkpr** in
> **Customize**.

---

## Usage

Everything you do runs in a **Cowork chat**. On the home screen, in the message box, select
**Cowork** (the toggle next to **Chat**), then start your message with the skill's slash command:

- **Coverage** — `/coverage`

Starting with the slash command makes sure Claude runs the right skill. Everything below is typed
straight into that Cowork chat.

### Start tracking a press release

Paste the press release (or just name the brand and release) after the command:

```
/coverage track coverage for this press release

<paste the full press release here>
```

That's all you do. HawkPR reads the brand and website out of the release, asks you to pick a short
**campaign name** (it suggests two — pick one or type your own), then kicks off the sweep.

**What you get**

- A quick handoff in chat showing the **tracked search queries** it will monitor (plus any Google
  Alerts it set up).
- A one-time question: **want email alerts when new coverage is confirmed?** If the release lists a
  media contact, it offers that address — you can accept it, use a different one, or say no.
- A status summary and a **shareable dashboard link** to bookmark. New and existing coverage shows
  up there, and checks keep running **a few times a day** (across the Australian news cycle) in the background.

**Good to know**

- **The first sweep reaches back** through everything already published, so you'll often see
  coverage right away — not just future pickups.
- **Emails are opt-in.** No alerts go out unless you turn them on and confirm the address. When on,
  the recipients also get a one-time note listing exactly what's being watched.
- **Re-running is safe.** Point it at the same brand and campaign again and it resumes the existing
  campaign instead of creating a duplicate.

### Check, list, and tweak

Once a campaign exists, you can talk to it in plain English — all through `/coverage`:

- **Check for new coverage now** — `/coverage any new coverage for <brand>?`
- **List your campaigns** — `/coverage list my campaigns`
- **Preview the alert email** — `/coverage send me a sample coverage email for <brand>`
- **Edit what it watches** — `/coverage change the tracked queries for <brand>`

Each one finds the right campaign for you and reports back in chat, always with the dashboard link.

---

## Managing the plugin

**Update** — open **Customize → Plugins** in the left sidebar, find the **hawkpr-plugin**
marketplace, and click **Update** to sync the latest version. Your campaigns live on the HawkPR
service, so nothing is lost — you don't need to redo anything.

**Remove** — in **Customize → Plugins**, click **Uninstall** on the **hawkpr** plugin to turn it
off, or remove its marketplace to delete the source completely. To also disconnect the service,
remove the **HawkPR** connector under **Customize → Connectors**.

---

## Need a hand?

- **`/coverage` doesn't do anything?** Open **Customize → Connectors** and check that **HawkPR**
  shows **Connected**. If it's missing, redo **Install Step 1**.
- **Want to see your coverage anytime?** Open the dashboard link HawkPR gave you — it's the one
  thing worth bookmarking.
- **Something looks off?** Just tell Claude what you want — "check <brand> again", "add a query
  about <topic>", "who gets the alert emails?". It's built for that.
- **Stuck?** Ask in the team's internal channel, or ping whoever set up HawkPR.
