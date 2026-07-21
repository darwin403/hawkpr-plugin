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

1. **Gather the pitch.** From the pasted press release extract `brandName` and
   `brandDomain` (the brand's own site, e.g. `luxoliving.com.au`). If a media
   contact email appears in the footer or contact block (`Media contact`, `For
   media enquiries`, `Press contact`), note it for step 7 but do not act on it
   yet. Confirm brand and domain in one line, and add that you'll sweep for any
   coverage the release has **already** picked up as well as watch for new pickups
   going forward. Never invent facts not in the pitch. Do **not** extract or
   confirm a pitch date — the sweep finds past coverage without one.
2. **Name the campaign — always ask.** The `name` is a **crisp internal label**
   that identifies this release at a glance, *not* the full headline or a long
   thematic angle. Shape: **3–6 words**, the core subject of the release, no
   colon/subtitle tails, no trailing "— …" clauses. E.g. *No safe threshold sperm
   discovery* — not *No safe threshold: how everyday lifestyle choices impact male
   fertility*. Derive **two** candidate labels from the headline/dek, then use
   `AskQuestion` (not trailing prose) with prompt *"What should I call this
   campaign? Pick one or type your own."*:
   - Option 1 — your best recommended label (marked *(recommended)*).
   - Option 2 — a shorter or alternative phrasing.
   - The built-in **Other** lets the user type their own name for reference.

   Use the chosen text verbatim as `name`.
3. **Create** — call `create_campaign` with `brandName`, `brandDomain`, `name`,
   and `pressRelease` only (omit `notificationEmails`); keep `campaignId`, `slug`,
   `url`. If `existed` is true, say you're resuming rather than duplicating.
4. **Hunt** — call `start_hunt` with `campaignId`. It returns immediately; the
   agent generates tracked queries and runs the initial scan in the background.
5. **Wait for queries only.** Poll `get_campaign` every ~10–15s until
   `searchQueries` is non-empty (or `hasSavedQueries` is true). Skip polling if
   the campaign already has queries (e.g. `existed: true`). If queries still have
   not appeared after ~2–3 minutes, proceed to step 6 anyway (queries may still
   be generating).
6. **Hand off tracked queries** (do not wait for placements or `lastSweptAt`):
   - Open with one line: *Tracking **{name}** for **{brandName}** (`{brandDomain}`).*
   - **Tracked queries** — bullet list from `searchQueries` (agent-generated).
     Note the sweep may refine this list when it finishes.
   - **Google Alerts** — when `alertQueries` is non-empty, one line: *Also
     monitoring via Google Alerts: {alertQueries, comma-separated} + brand
     mentions of {brandName}.* Omit the line if `alertQueries` is empty.
   - Do not include Status yet. Do not label or refer to the campaign as an "angle".
7. **Ask which email should get alerts (not a yes/no).** Unless resuming a campaign
   that already has `notificationEmails` set, use `AskQuestion` — not trailing prose —
   with prompt *"Which email should get new-coverage alerts?"* and a suitable default
   pre-filled as the recommended option:
   - Option 1 *(recommended)* — the default address shown bare (e.g. *`{email}`*, not
     "Yes, use …"): the PR's media contact when the release has one, otherwise the
     current user's email if known.
   - Option 2 — *Don't send alerts*.
   - The built-in **Other** lets the user type one or more comma-separated addresses.
   - On an address → `set_notification_emails(campaignId, emails)`. This also emails
     each recipient a one-time setup confirmation listing the tracked search queries +
     Google Alerts (observability into what's being watched); the response reports
     `confirmationSent`/`confirmedTo`. On *Don't send alerts* → leave off.
   - If notifications already set, skip the question — one line in the status
     handoff is enough (e.g. *Alerts go to `{email}` when new coverage is confirmed.*).
8. **Status + link** — show a status table, then a prominent **[Open campaign](url)** link:

   | | |
   |---|---|
   | **Campaign** | `{name}` |
   | **Brand** | `{brandName}` |
   | **Tracked since** | `{createdAt}` formatted |
   | **Notifications** | Off, or comma-separated `notificationEmails` if set in step 7 |
   | **Last scan** | Relative time from `lastCheckedAt`; if null → `In progress` |
   | **Frequency** | Every 15 minutes |

   Never use "First scan", "Next scan", or "first scan in the background" in
   user-facing copy.

9. **Close with what happens next** — one short declarative line after the link.
   No questions, no menu of optional next steps (do not ask about editing queries,
   sample emails, or on-demand checks).

   - Notifications on: *A confirmation email listing the tracked queries and
     Google Alerts is on its way to `{notificationEmails}`. Existing and new
     coverage will show on the dashboard, and those addresses are emailed
     automatically as placements are confirmed.*
   - Notifications off: *Existing and new coverage will show on the dashboard —
     checks run every 15 minutes.*

   Setup is complete; the user can bookmark the link and leave it.

The expensive sweep runs once and reaches back through everything already
published; ongoing checks run every 15 minutes.

## B. Check a campaign (on-demand)

1. **Resolve.** If the user gave a slug or URL, use it. Otherwise call
   `list_campaigns` with `search` set to the brand or campaign name. Several matches → ask
   which; none → there's no campaign yet, offer **A**.
2. **Snapshot** — `get_campaign` and `list_placements`; note placement count and
   `lastCheckedAt`.
3. **Trigger check** — call `start_hunt` with `campaignId` if the user wants a
   fresh check. It returns immediately; do not poll for updated counts.
4. **Report** — summarize the snapshot in markdown: total placements, links vs.
   mentions, notable domains, last scan (`lastCheckedAt` or `In progress`), frequency
   (Every 15 minutes), and the `url`. Note the check runs in the background if you
   just triggered one. Close declaratively — no optional-setup questions.

## C. List campaigns

Call `list_campaigns` (optionally with `search`). Show brand, campaign name,
last scan (`lastCheckedAt` or `In progress`), frequency (Every 15 minutes), and
each `url`. Offer to check one (**B**) or start a new one (**A**).

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

- **Campaign name:** a crisp internal label (3–6 words) identifying the release at a
  glance — e.g. *No safe threshold sperm discovery*, not *No safe threshold — male
  fertility lifestyle impact* and never the full headline. Always let the user pick or
  rename it via `AskQuestion` (step A.2) — recommend two candidates and rely on the
  built-in **Other** for a custom name. Never use "angle" in user-facing copy.
- Notification emails: ask **after** showing tracked queries and **before** the status
  table, via `AskQuestion`, framed as *which address* should receive alerts (with a
  default pre-filled) — not a yes/no. Default the recommended option to the PR media
  contact when present, else the current user's email; the user can pick **Other** to
  type another or *Don't send alerts* to skip. Configure with `set_notification_emails`,
  not at `create_campaign`, and never set it silently (the question is the consent).
  Skip when resuming with notifications already set.
- **Scan status:** last scan from `lastCheckedAt` (relative time, or `In progress`
  when null). Frequency is always **Every 15 minutes** — hardcoded, not computed.
  Never say "First scan" or "Next scan".
- **Handoff closing:** after onboarding or a check, state expected outcomes in one
  line — never end with questions like "Want to edit queries, send a sample email, …?"
  Optional actions (sections **D**, **E**) are only when the user asks.
- Everything is idempotent; re-running on the same brand + campaign name resumes the
  existing campaign rather than duplicating.
- You never need to re-run `start_hunt` for routine monitoring — checks run every
  15 minutes automatically. Use it only for the initial sweep (**A**) or an
  explicit on-demand check (**B**).
