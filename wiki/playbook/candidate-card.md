# Candidate card

The information needed before anything is built, and the template every note in
`../candidates/` follows. The target research agent fills it; you edit it and rule; the
builder reads the approved version as its brief. This page is playbook: it changes only
through a pull request the user approves.

## Exclusions — never scored, never sent
- A login wall, or data behind any account.
- Private or sensitive personal data. Public professional data (names, job titles, work
  emails) is allowed, with a lawful-basis line (ruled 2026-10-18).
- A site not in English.
- Work for a named client (scraping-as-a-service is out of scope).

A site whose terms forbid automated access is **not** excluded by the agent — it is sent with
the terms line quoted and flagged, because that is a ruling for you.

## Scoring — demand first (ruled 2026-10-18)
Growth adjusts a score; it never selects a candidate.

| Part | Range | How it is set |
|---|---|---|
| Demand | 0–3 | 0 no evidence anyone wants it · 1 one weak signal · 2 several independent signals (forum posts, GitHub scrapers with stars and open issues, data people copy by hand) · 3 demand people already pay for (an incumbent with paid users, Upwork jobs) |
| Gap | 0–3 | 0 a healthy incumbent we cannot undercut · 1 a healthy incumbent we can undercut on price · 2 an incumbent that is failing, under maintenance, proxy-walled or poorly rated · 3 no actor at all |
| **Base** | 0–9 | Demand × Gap. **Base 0 is never sent, whatever the growth.** |
| Growth | 0 to +2 | +1 a launch or rank rise in the last 90 days · +2 a rise across at least two readings plus launch coverage. Applied only when Base > 0. |
| Fetch | 0 to −3 | 0 all three pages return the data to a plain fetch · −1 the data needs a browser (more compute) · −3 any anti-bot challenge (Cloudflare, DataDome): the price must assume residential proxies, and most such candidates will not clear cost |
| **Total** | | Base + Growth + Fetch |

These weights are the starting point. They change only on evidence from
`../candidates/rulings-log.md`, through a pull request to this page.

## Template

```
---
candidate: <slug>
site: <https://...>
status: proposed        # proposed | changes-requested | approved | rejected | parked
proposed: YYYY-MM-DD
reviewed:               # YYYY-MM-DD, set by you
score_demand: 0
score_gap: 0
score_base: 0
score_growth: 0
score_fetch: 0
score_total: 0
flags: []               # e.g. [terms-forbid-scraping, public-professional-data]
ruling:                 # the date of the decisions.md line, once recorded
---

## What a buyer wants
The data, in one or two sentences, and the search a buyer would type to look for it.

## Demand evidence
Each signal with its link and the date it was read.

## Store gap evidence
Best existing actor (link to its page under evidence/actors/), its 30-day success rate,
rating, review count, notice, permission level, agentic-payments flag and price on the FREE
buyer tier (incumbents price per buyer plan; name the tier) — or "no actor", with the Store
searches tried.

## Growth signal
Launch posts, rank readings with their dates, or "none".

## Plain-fetch result
Three page URLs; for each: HTTP status, whether the wanted data is in the response, any
challenge page, and the IP class the fetch ran from (datacenter egress, as from a cloud
routine, is the right worst case; record which it was).

## Terms of service
The relevant line quoted, its URL and the date read.

## Data class and lawful basis
What kind of data each output field is. For any personal data (public professional only):
the lawful basis in one line.

## Proposed output
The fields per row, with one example row taken from the fetched pages.

## Price and compute estimate
Incumbent price per 1k results on the FREE buyer tier (incumbents price per buyer plan; name
the tier); our proposed price; the compute estimate behind it (pages per result, page weight,
browser or not, proxy type) — marked as an estimate until the fixture run measures it.

## Suggested fixture page
The page the builder would draft expected rows from, for you to approve or swap.

## Risks
What could make this lose money or break often.

## Your changes
You write here: what you changed and why. The agent never edits this section.
```
