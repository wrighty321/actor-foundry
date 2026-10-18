# Decisions Log

Append-only, chronological (oldest first, newest at the **bottom**). One entry per
decision. Maintained by `/consolidate` — do not hand-edit or reorder existing entries.

Format: `YYYY-MM-DD — <what was decided> — because <why>`

---

<!-- New entries are appended below, at the end of the file, newest last. -->
2026-10-18 — Public professional data (names, job titles, work emails) is an allowed target; private and sensitive data stay excluded, lawful basis recorded per actor — because the brief's blanket personal-data exclusion was ruled too strict at interview; UK GDPR still applies and was flagged once.
2026-10-18 — Actors are open source and priced to undercut the incumbent for the same data — because trust and discovery on the Store outweigh copy risk, and price plus reliability is the chosen wedge.
2026-10-18 — Fixtures are builder-proposed and user-approved, not user-authored — because the fixture ruling is about authority over the expected rows, not who types them.
2026-10-18 — Maintainer runs on GitHub Actions cron; escalations and a weekly summary go to Telegram; an unrepairable actor is paused with a status message — because in-app schedules need the app open, Telegram is checked same day, and pausing protects ratings where unpublishing loses users.
2026-10-18 — Maintainer never publishes, changes a price, touches another actor's fixture, or exceeds 1 CU per run (default until measured) — because these are the actions whose cost is external and the user accepted the draft list at interview.
2026-10-18 — Target selection starts from a competitor assessment and a self-updating actor intelligence bank; the human still rules before any build — because the user has no sites in mind and wants selection grounded in evidence (modifies the same-day "human picks by hand" ruling from the Kun pass).
2026-10-18 — Escalation classes form a trust ladder: same-day human review now, automated only after 10 consecutive agreements with the user's ruling (telos R5) — because everything is to be automated eventually, on evidence rather than confidence.
2026-10-18 — Add an actor intelligence bank under bank/ (weekly Store API pull on GitHub Actions, monthly synthesis on a Claude cloud routine with a manual /bank-refresh fallback) as milestone 1b alongside the maintainer — because the user wants target selection and builds grounded in measured competitor evidence; the Store API exposes 30-day run success counts, ratings and maintenance notices per actor.
