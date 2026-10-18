# 005 — Maintainer state and log: runtime state in the control store, the run log in the repo
Status: accepted — 2026-10-18, by the user
Amended 2026-10-18: `maintainer/STATUS.md` and the non-zero exit by 013.

## Context
- The brief's "done" needs one log entry per maintainer run, 7 then 30 consecutive days,
  provable later.
- The pause flag must be readable by the actor at run time (ADR 003), so at least that state
  lives on Apify, not only in the repo.
- Runs on GitHub Actions are ephemeral; artifacts expire (90 days by default — **assumed**).
- Logs and histories are append-only and nobody edits the past (vault `agents.md` V2).

## Decision
- **Runtime state** — pause flag, repair id, last check — lives in the `actor-foundry-control`
  key-value store (ADR 003). It is the only state an actor reads.
- **The run log** — one append-only file per actor in the repo, `maintainer/log/<actor>.md`,
  one line per run: `date | L1 pass/fail | L2 pass/fail | failed anchors | action
  (none | paused | escalated | repair-requested) | compute units | run url`. The check workflow
  appends and commits with `contents: write`. The 7- and 30-day proofs are counted from these
  lines; the weekly Telegram summary is generated from them plus the manual profit reading.
- **Compute units** per check are read from the run's usage and written on the line, so the
  1-CU limit is measured, not assumed.

## Alternatives
- **An Apify dataset as the log.** Rejected: the proof lives outside git and outside the repo
  the user reads; nothing else in the project is read from a dataset.
- **Actions artifacts.** Rejected: ephemeral, not greppable, not in the repo.
- **GitHub Issues, one per run.** Rejected: noisy, hard to count, outside the file-based record
  everything else uses.
- **One file per run.** Rejected: hundreds of files for the same information one appended line
  carries.

## Consequences
- About one bot commit per actor per day on `main`. The deploy workflow (ADR 004) must trigger
  on `actors/**` paths only, so log commits never deploy anything; the check workflow commits
  only under `maintainer/**`.
- The log is readable in Obsidian alongside `wiki/` and is the raw input for
  `wiki/first-party/`.
- Commits us to: the line format above (changing it is a new ADR), `maintainer/log/` as a
  write-once path, and `contents: write` on the check workflow — the one permission it needs.
