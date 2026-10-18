# actor-foundry
> **Aim:** Build and keep alive a portfolio of paid Apify Store actors, where each actor ships with a user-owned fixture and a machine loop checks, repairs or escalates against it, so the portfolio grows faster than one person can repair by hand. **Not in scope:** scraping personal data or login-walled sites; publishing to the Store without a human ruling in `context/decisions.md`; scraping-as-a-service for named clients.

## Conventions & gotchas
- Every actor is pay-per-event with limited permissions, charged through `push_data(..., charged_event_name=...)`, and persists progress on the `PERSIST_STATE` event. No exceptions: this is what keeps it eligible for agentic payments and off the negative-profit path.
- Every actor has a fixture under `actors/<name>/fixture/` that the user authored or approved. The builder never writes an actor's tests; the maintainer never publishes a change the fixture did not pass.
- No `apify push` to a public listing without a dated ruling in `context/decisions.md` and a PR that passed an adversarial review in a separate session.
- The Apify token lives in `.env` here and is never placed on the Grok Bot computer; the watcher is read-only.
- The maintainer runs on **GitHub Actions cron** (ruled 2026-10-18); in-app scheduled tasks fire only while the app is open and are not a runner. The Apify token and Telegram secrets are repository secrets there, nowhere else.
- Maintainer hard limits: never publishes, never changes a price, never touches another actor's fixture, never exceeds 1 compute unit per run (default until measured). An actor it cannot repair is **paused with a status message** — never left running broken, never unpublished.
- Escalations and ruling requests go to **Telegram**; the user reviews same day. A weekly Telegram summary covers profit per actor, runs, repairs and escalations. Escalation classes are a trust ladder: automate one only after it has matched the user's ruling 10 consecutive times (telos R5).
- A ruling request for a candidate site shows: the plain-fetch result, the terms-of-service line, the data class and lawful basis, the Store gap evidence, and a price with the compute estimate behind it. A publish request shows the fixture passing and a sample of real output against what the candidate request promised.
- Fixtures are **builder-proposed, user-approved**: the builder drafts expected rows from a page the user chose; nothing counts as a fixture until the user approves it.
- Actors are **open source** and **priced to undercut** the incumbent actor for the same data; the reliability claim is what the price is not.
- Public professional data (names, job titles, work emails) is an allowed target; private and sensitive data are not (ruled 2026-10-18, narrowing the brief's blanket exclusion). English-language sites only for now.
- Python: `uv` for environments and locking, `pytest`, `ruff` (format and lint); type hints on public functions, plain functions over class hierarchies, async only where the Apify SDK requires it.

## Key files
- `bank/` — the actor intelligence bank: `snapshots/` is the source of truth, `actors/` and `patterns/` are generated from it, `gaps.md` feeds ruling requests. Read `bank/README.md` before touching it; read `bank/patterns/` before drafting any actor.

## Context (read on demand — deliberately not loaded here)
- `context/project-brief.md` — frozen goal & scope (drift anchor).
- `context/project-plan.md` — living milestones/risks, maintained by /consolidate.
- `context/commitments.md` — frozen viability thesis & kill criteria.
- `context/vision.md` — why this route, rejected alternatives, tripwires, success gates (read before any directional change).
- `context/decisions.md` — decision log, maintained by /consolidate.
- `context/research-2026-10-18.md` — Apify platform facts, Store gap-check heuristics, watcher feeds, Grok Bot facts, each dated (read before milestone 2 target selection or any pricing).
- Vault (cross-repo substrate): `C:\Users\Alexc\Vault\` — read its `agents.md` only before writing there.
