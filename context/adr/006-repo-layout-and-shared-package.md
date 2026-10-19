# 006 — Repo layout: one monorepo, a shared `foundry` package, one directory per actor
Status: accepted — 2026-10-18, by the user
Amended 2026-10-18: the layout gains `library/` (OS-shaped workflow directories and contracts) and `.os/` (project truth); the shared package's deterministic steps are functions `fn.foundry.*@1` (016).

## Context
- Every actor repeats the same first steps: read the control record (ADR 003), charge through
  `push_data(..., charged_event_name=...)` and stop cleanly on the spending limit, persist on
  `PERSIST_STATE`, fail loudly with a status message. Duplicated per actor, these drift.
- The maintainer, the wiki pull and the deploy step are repo-level code, not actor code.
- The brief's stack: Python, Apify Python SDK, Crawlee for Python, `uv`, `pytest`, `ruff`; one
  actor per directory under `actors/<name>/` with its fixture under `actors/<name>/fixture/`.
- On this machine bare `python` resolves to an unrelated venv (vault
  `knowledge/bare-python-resolves-to-notebooklm-venv.md`); every command must name its
  interpreter — `uv run` does.
- **Assumed:** `apify push` builds from the actor directory as the Docker context, so a
  Dockerfile there cannot `COPY` from the repo root. Verified on the first private stub push.

## Decision
```
actor-foundry/
  pyproject.toml            # uv workspace root: foundry + actors/*
  foundry/                  # shared package (pure Python, no actor code)
    control.py              #   kill-switch read/write + read-back (ADR 003)
    charge.py               #   push_data wrapper: charged event, ChargeResult, limit → clean stop
    fixture.py              #   L1/L2 runner (ADR 002)
    store.py                #   public Store API client for the wiki pull (paging, local ranking)
    telegram.py             #   send-only client
    cli.py                  #   `foundry check|pause|unpause|pull|summary|deploy`
  actors/<name>/
    .actor/actor.json       #   PPE events, limited permissions, memory default
    Dockerfile
    src/main.py             #   control check → input → crawl → extract → charged push
    src/extract.py          #   pure: response → rows (L1 runs this offline)
    fixture/                #   ADR 002
  maintainer/log/<actor>.md # ADR 005
  wiki/                     # the Obsidian vault
  routines/                 # self-contained prompt files for cloud routines (ADR 007)
  .github/workflows/        # maintainer-check, maintainer-repair, deploy, wiki-pull, notify
```
`foundry` is built as a wheel by the deploy workflow and copied into
`actors/<name>/.vendor/` (gitignored) before `apify push`, so each actor image installs it
from a local file. Actors are open source, so the wheel is public by construction.

## Alternatives
- **One repository per actor.** Rejected: the maintainer workflow, fixture rules and shared
  helpers would be copied per repo and drift; the Store listing does not care where the code
  lives.
- **No shared package — copy the helpers into each actor.** Rejected: the first fix to
  `charge.py` would have to land N times, and the brief's "each fix lands in the template"
  rule would become a copy-paste chore.
- **Publish `foundry` to PyPI and `pip install` it in the Dockerfile.** Deferred: a release
  step before the first actor exists is ceremony; the local wheel does the same job. Revisit
  when a third actor exists.
- **Apify's Git integration (build on push).** Rejected for now: `apify push` from the deploy
  workflow keeps the publish gate (ADR 004) in one place; Git integration would rebuild on
  every merge regardless of the ruling.

## Consequences
- Easier: one `uv run pytest` runs every actor's L1 and the package tests; one `ruff` config;
  the template is a directory to copy.
- Harder: the vendored-wheel step is one more thing in the deploy workflow; the Docker-context
  premise is unverified until the stub push.
- Commits us to: `foundry` as the only place actor plumbing lives, `uv run` in every verify
  step, and `actors/<name>/src/extract.py` as a pure function.

Trivial choices made without an ADR: `httpx` for Telegram and Store calls, JSON Lines for
rows and snapshots, Python 3.12 (the machine's install), `pytest` and `ruff` as given.
