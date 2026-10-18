# actor-foundry
> **Aim:** Build and keep alive a portfolio of paid Apify Store actors, where each actor ships with a user-owned fixture and a machine loop checks, repairs or escalates against it, so the portfolio grows faster than one person can repair by hand. **Not in scope:** scraping personal data or login-walled sites; publishing to the Store without a human ruling in `context/decisions.md`; scraping-as-a-service for named clients.

## Conventions & gotchas
- Every actor is pay-per-event with limited permissions, charged through `push_data(..., charged_event_name=...)`, and persists progress on the `PERSIST_STATE` event. No exceptions: this is what keeps it eligible for agentic payments and off the negative-profit path.
- Every actor has a fixture under `actors/<name>/fixture/` that the user authored or approved. The builder never writes an actor's tests; the maintainer never publishes a change the fixture did not pass.
- No `apify push` to a public listing without a dated ruling in `context/decisions.md` and a PR that passed an adversarial review in a separate session.
- The Apify token lives in `.env` here and is never placed on the Grok Bot computer; the watcher is read-only.
- In-app scheduled tasks fire only while the app is open — the maintainer's runner is GitHub Actions cron or an always-on machine, decided in milestone 1.

## Context (read on demand — deliberately not loaded here)
- `context/project-brief.md` — frozen goal & scope (drift anchor).
- `context/project-plan.md` — living milestones/risks, maintained by /consolidate.
- `context/commitments.md` — frozen viability thesis & kill criteria.
- `context/vision.md` — why this route, rejected alternatives, tripwires, success gates (read before any directional change).
- `context/decisions.md` — decision log, maintained by /consolidate.
- Vault (cross-repo substrate): `C:\Users\Alexc\Vault\` — read its `agents.md` only before writing there.
