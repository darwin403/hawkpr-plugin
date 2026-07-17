---
name: coverage
description: Track and check third-party news coverage of a pitched press release — the single entry point for all coverage actions. Use when the user says things like "/coverage", "track coverage for this press release", "start monitoring coverage for a brand", "any new coverage for a brand", "check coverage", "check coverage for a brand", "send me a sample coverage email", or pastes a press release and asks to watch for pickup. Handles setup, on-demand checks, listing campaigns, editing tracked queries, and sample emails — no web UI needed.
---

# Coverage

The single entry point for coverage tracking within the HawkPR toolkit.
Everything runs through the HawkPR MCP server (`hawkpr`): `create_campaign`,
`list_campaigns`, `get_campaign`, `list_placements`, `set_queries`,
`set_notification_emails`, `send_sample_email`, `start_hunt`.

The heavy SERP sweep + verification run asynchronously in the coverage agent;
these MCP tools are fast and deterministic. `start_hunt` returns immediately.
During onboarding, poll `get_campaign` only until `searchQueries` is non-empty
— do not wait for placements or sweep completion.

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
   `brandDomain` (the brand's own site, e.g. `luxoliving.com.au`), and `name`
   (a **concise campaign title** — 3–8 words, derived from the release headline
   or dek; crisp and recognizable, not a long thematic angle). If a media contact
   email appears in the footer or contact block (`Media contact`, `For media
   enquiries`, `Press contact`), note it for step 5 but do not act on it yet.
   Confirm brand, domain, and campaign name in one line. Never invent facts not in
   the pitch. Do **not** extract or confirm a pitch date.
2. **Create** — call `create_campaign` with `brandName`, `brandDomain`, `name`,
   and `pressRelease` only (omit `notificationEmails`); keep `campaignId`, `slug`,
   `url`. If `existed` is true, say you're resuming rather than duplicating.
3. **Hunt** — call `start_hunt` with `campaignId`. It returns immediately; the
   agent generates tracked queries and runs the first scan in the background.
4. **Wait for queries only.** Poll `get_campaign` every ~10–15s until
   `searchQueries` is non-empty (or `hasSavedQueries` is true). Skip polling if
   the campaign already has queries (e.g. `existed: true`). If queries still have
   not appeared after ~2–3 minutes, proceed to step 5 anyway (queries may still
   be generating).
5. **Offer email alerts (optional, before handoff).** Always ask before step 6
   unless resuming a campaign that already has `notificationEmails` set. Use
   `AskQuestion` — not trailing prose — with prompt *"Want automatic emails when
   new coverage is confirmed?"*
   - **PR has media contact:** include an option to use `{email}` from the release;
     accept confirm, substitute, comma-separated addresses, or decline.
   - **No media contact:** yes/no; if yes, ask which address(es).
   - On yes → `set_notification_emails(campaignId, emails)`. On no → leave off.
6. **Hand off in markdown** (do not wait for placements or `lastSweptAt`):
   1. **Tracked queries** — bullet list from `searchQueries` (agent-generated).
      Note the sweep may refine this list when it finishes.
   2. **Status** — campaign name (`name`), brand (`brandName`), tracked since
      (`createdAt`), notifications (off, or list `notificationEmails` if set in
      step 5); first scan continues in the background.
   3. A prominent **[Open campaign](url)** link.

   Open with one line: *Tracking **{name}** for **{brandName}** (`{brandDomain}`).*
   Do not label or refer to the campaign as an "angle".
7. **Other optional follow-ups:** query edits (→ **E**), sample email (→ **D**).

The expensive sweep runs once; ongoing checks are cheap and scheduled.

## B. Check a campaign (on-demand)

1. **Resolve.** If the user gave a slug or URL, use it. Otherwise call
   `list_campaigns` with `search` set to the brand or campaign name. Several matches → ask
   which; none → there's no campaign yet, offer **A**.
2. **Snapshot** — `get_campaign` and `list_placements`; note placement count and
   `lastCheckedAt`.
3. **Trigger check** — call `start_hunt` with `campaignId` if the user wants a
   fresh check. It returns immediately; do not poll for updated counts.
4. **Report** — summarize the snapshot in markdown: total placements, links vs.
   mentions, notable domains, and the `url`. Note the check runs in the background
   if you just triggered one.

## C. List campaigns

Call `list_campaigns` (optionally with `search`). Show brand, campaign name,
status, last/next check, and each `url`. Offer to check one (**B**) or start a new
one (**A**).

## D. Sample email

Resolve the campaign (as in **B.1**). Call `send_sample_email` — omit `to` to use
the campaign's stored notification emails, or pass a comma-separated `to` override.
It uses the campaign's real placements when available, otherwise a representative
sample. Confirm what was sent and to whom.

## E. Edit queries

Resolve the campaign. Show the current `searchQueries` from `get_campaign`, take
the user's edits, and call `set_queries` with the final list (deduplicated on
save). Confirm the new set — it's what ongoing checks will monitor.

## Notes

- **Campaign name:** short title from the release headline — e.g. *No safe threshold
  sperm discovery*, not *No safe threshold — male fertility lifestyle impact*.
  Never use "angle" in user-facing copy.
- Notification emails are optional but ask via `AskQuestion` **before** the
  handoff, not after. Configure with `set_notification_emails`, not at
  `create_campaign`. Propose the PR media contact when present — never silently
  use a client email or default to the user's own address. Skip when resuming
  with notifications already set.
- Everything is idempotent; re-running on the same brand + campaign name resumes the
  existing campaign rather than duplicating.
- You never need to re-run `start_hunt` for routine monitoring — the schedule
  handles incremental checks. Use it only for the initial sweep (**A**) or an
  explicit on-demand check (**B**).
