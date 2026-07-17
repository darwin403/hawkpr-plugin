---
name: coverage
description: Track and check third-party news coverage of a pitched press release — the single entry point for all coverage actions. Use when the user says things like "/coverage", "track coverage for this press release", "start monitoring coverage for a brand", "any new coverage for a brand", "check coverage", "check coverage for a brand", "send me a sample coverage email", or pastes a press release and asks to watch for pickup. Handles setup, on-demand checks, listing campaigns, editing tracked queries, and sample emails — no web UI needed.
---

# Coverage

The single entry point for coverage tracking within the HawkPR toolkit.
Everything runs through the HawkPR MCP server (`hawkpr`): `create_campaign`,
`list_campaigns`, `get_campaign`, `list_placements`, `set_queries`,
`send_sample_email`, `start_hunt`.

The heavy SERP sweep + verification run asynchronously in the coverage agent;
these tools are fast and deterministic. `start_hunt` returns immediately — poll
`get_campaign` / `list_placements` for progress.

## Routing — pick the intent

Read the user's message and pick the matching section. If invoked bare (just
`/coverage` with no clear intent), briefly list what you can do — start tracking a
new press release, check an existing campaign, or list campaigns — and ask which,
then proceed.

- Pasted a press release, or "track / start monitoring coverage" → **A. Start tracking**
- "any new coverage", "check coverage", "run a check" for a brand → **B. Check a campaign**
- "list / show my campaigns" → **C. List campaigns**
- "send a sample email", "what does the alert look like" → **D. Sample email**
- "change / edit the tracked queries" → **E. Edit queries**

Always end by giving the campaign `url` — it's the one thing to bookmark.

## A. Start tracking (new campaign onboarding)

1. **Gather the pitch.** From the pasted press release extract `brandName`,
   `brandDomain` (the brand's own site, e.g. `luxoliving.com.au`), a short `name`
   (the pitch angle), and `pitchDate` (`YYYY-MM-DD`, when the pitch went out).
   Confirm them back in one line. Never invent facts not in the pitch.
2. **Create** — call `create_campaign`; keep `campaignId`, `slug`, `url`. If
   `existed` is true, say you're resuming rather than duplicating.
3. **Hunt** — call `start_hunt` with `campaignId`. It returns immediately; tell the
   user the first scan runs in the background (a few minutes) and is resumable.
   Poll `get_campaign` every ~20–30s until `hasSavedQueries` is true (the sweep
   saved its winning queries); placements usually appear by then.
4. **Summarize as an Artifact.** Call `get_campaign` + `list_placements`, then
   render one self-contained HTML page (consult the `artifact-design` skill for
   calibration) with, in order:
   1. **Tracked queries** (`searchQueries`) — labelled as what will be monitored.
      This is the alignment check.
   2. **Sample alert preview** — a compact rendering of the coverage-alert email
      from the found placements (title, domain, author, link type).
   3. **What was found** — placement count, links vs. mentions, `lastCheckedAt`,
      and `nextCheckAt`.
   4. A prominent **"Open campaign"** button linking to `url`.
5. **Confirm & hand off.** Ask the user to confirm or edit the tracked queries
   (→ **E** if they change them). Offer to send a real sample email (→ **D**).
   Restate that tracking is live and scheduled — new coverage is found
   automatically and emailed — and give the `url`.

Do not skip the query confirmation in step 5; it is the alignment moment. The
expensive sweep runs once; ongoing checks are cheap and scheduled.

## B. Check a campaign (on-demand)

1. **Resolve.** If the user gave a slug or URL, use it. Otherwise call
   `list_campaigns` with `search` set to the brand/angle. Several matches → ask
   which; none → there's no campaign yet, offer **A**.
2. **Snapshot** — `get_campaign`; note placement count and `lastCheckedAt`.
3. **Incremental check** — call `start_hunt` with `campaignId` (it enforces its
   own cooldown, so just call it). It returns immediately; poll `get_campaign` a
   few times to see if `lastCheckedAt` advances and the count grows. If state
   doesn't advance (cooldown), just report current placements — don't force it.
4. **Report** — `list_placements`, then summarize: new vs. the snapshot, total,
   links vs. mentions, notable domains, and the `url`. For more than a couple of
   new placements, render a compact Artifact table instead of a long chat list.

## C. List campaigns

Call `list_campaigns` (optionally with `search`). Show brand, angle, placement
status, last/next check, and each `url`. Offer to check one (**B**) or start a new
one (**A**).

## D. Sample email

Resolve the campaign (as in **B.1**). Ask for the recipient address (default to
the user's own if known), then call `send_sample_email` with `to`. It uses the
campaign's real placements when available, otherwise a representative sample.
Confirm what was sent and to whom.

## E. Edit queries

Resolve the campaign. Show the current `searchQueries` from `get_campaign`, take
the user's edits, and call `set_queries` with the final list (deduplicated on
save). Confirm the new set — it's what ongoing checks will monitor.

## Notes

- Everything is idempotent; re-running on the same brand + angle resumes the
  existing campaign rather than duplicating.
- You never need to re-run `start_hunt` for routine monitoring — the schedule
  handles incremental checks. Use it only for the initial sweep (**A**) or an
  explicit on-demand check (**B**).
