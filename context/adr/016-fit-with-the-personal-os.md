# 016 — Fit with the personal OS: workflows in the library shape, deterministic steps as functions, `.os/project.yaml` as the project truth
Status: proposed (user instruction of 2026-10-18: follow the OS plan's agent and workflow templates; this venture becomes a workflow in that system)
Amends: 006 (layout gains `library/` and `.os/`), 007 and 013 (`routines/*.md` become library workflows), 011 (trailer fields)

## Context
- The user's personal operating system is planned in `C:\Claude_Repositories\workshop\context\os-plan\`
  (the connection map is the specification; rulings in `04-system-plan.md`) and built in
  `C:\Claude_Repositories\personal-os` (pull-only copy here; it builds only in its own
  container). Read 2026-10-18.
- The shape every capability shares there (ruled 2026-09-28 and after): a workflow is a
  directory `library/wf.<class>.<name>/` holding `workflow.yaml` (wiring as data: stages in
  order, each stage's kind — agent, code, call or advised — its model requirements, bounds,
  needs, endings, trust state), `agents/*.md` bodies, `lens.md` (the manager's checklist, drift
  rule, edit budget, never-edit list), `template.yaml` (the slot bank), `checks.yaml`,
  `contracts/` (JSON-schema handoffs `ct.<name>@n`) and `tests/` (per-agent sets and an
  end-to-end test, never written by the agent they test). Bodies come from a template bank
  `tpl.<class>@n` and carry nineteen fields under one token budget (1,500 tokens, know-how
  ≤ 600): `id`, `workflow`, `template`, `self`, `goal`, `objective`, `role`, `slots`, `input`,
  `output`, `tools`, `checks`, `model`, `bounds`, `budget`, `tests`, `escalate`, `skills`,
  `know-how`. Agents never write their own files; only the manager edits know-how, with two-run
  evidence; every workflow ends in a finisher; each agent runs in a fresh session; author and
  reviewer are isolated; a standing loop is opt-in and a workflow is a bounded run by default.
- Deterministic steps are functions `fn.<name>@n`: pure, contracted, tested, wired as code
  stages. Protocols (repo, scratch, document, file, draft, effect, connector, red line,
  schedule, child) are the shell's, never a workflow's.
- Every project carries `.os/project.yaml` (language, package manager, commands including the
  one end-to-end command, areas → paths, protected paths, key files, env var names),
  `.os/tools.yaml` (the command surfaces agents may use) and an ignored `.os/runs/`.
- Commits made by workflows carry trailers for model, brief id, token usage, session id and
  approval id, and never a co-author line (OS plan, ruled 2026-09-26/28).
- Schedules fire from the OS registry into intake. **The shell, registry and runners do not
  exist yet**: personal-os targets its first receipt on 2026-11-11. Nothing here can run on the
  OS today, and nothing here may import its internals (its own `CLAUDE.md`).
- The user's instruction: design to these templates so actor-foundry becomes part of the system
  as a workflow.

## Decision
1. **Every model-run piece of this venture is a workflow directory in the OS shape** under
   `library/`:
   - `wf.foundry.target-research` (milestone 1c) — reader (store-gaps and snapshot) → scout
     (demand outside the Store) → writer (candidate card) → finisher; ending: a pull request.
   - `wf.foundry.wiki-synthesis` (milestone 1d) — reader → extractor → writer → finisher;
     ending: pull requests.
   - `wf.foundry.repair` (C5, when built) — the coding-set shape with the reproducer replaced by
     the failing fixture: reproducer (runs L1/L2, names the failed assertions), implementer,
     fixer (two rounds), finisher; ending: a pull request with the fixture draft.
   - `wf.foundry.build` (the builder) — scout (candidate note's named sections, playbook),
     implementer, docs (README to the ADR 014 contract), finisher; **acceptance is the
     user-owned fixture**, so no stage writes a test; ending: a pull request.
   Bodies carry the nineteen fields and the budget; `know-how` starts empty and is manager-only;
   each has a per-agent test set the agent did not write; the planted-flaw sets of
   `architecture.md` are the end-to-end tests.
2. **Deterministic steps are functions** `fn.foundry.<name>@1` in the `foundry` package —
   `check`, `preflight`, `lint`, `deploy`, `pull`, `store_gaps`, `status`, `summary`,
   `telegram` — pure where the step allows, each with a contract and a test; the CLI is a thin
   wrapper and the workflows wire them as code stages.
3. **Handoff contracts** live as JSON schema under `library/contracts/`: `ct.foundry.candidate@1`
   (the candidate note's frontmatter), `ct.foundry.fixture@1`, `ct.foundry.control@1`,
   `ct.foundry.snapshot-header@1`, `ct.foundry.log-line@1`, `ct.foundry.listing@1`. Tests
   validate the wiki pages, the fixture and the log against them.
4. **Interim runner.** Until the OS shell exists, `uv run foundry run-workflow <id>` is the
   sequencer: it reads `workflow.yaml`, binds the slots from `template.yaml`, runs each agent
   body in a fresh Claude Code session in the wired order, validates every handoff against its
   contract, runs the declared checks, and ends with the finisher. GitHub Actions cron and
   `workflow_dispatch`, and the cloud routine, are the interim schedule and host. When the OS
   registry can register these workflows, its schedule protocol fires the same directories and
   the interim sequencer is retired; no file moves.
5. **`.os/project.yaml` is written now** and is the one truth `blueprint`, the build session and
   later the OS read: language, package manager, commands (`uv run pytest` as the end-to-end
   command, `uv run ruff check`, `uv run foundry lint`), areas → paths, protected paths
   (`wiki/playbook/**`, `.github/**`, `library/**`, `actors/*/fixture/**`, `context/adr/**`), key
   files, env var names (from `.env.example`). `.os/tools.yaml` lists the command surfaces
   agents may use. `.os/runs/` is gitignored.
6. **Agent-made commits carry the OS trailers** — `Model:`, `Brief:`, `Tokens:`, `Session:`,
   `Approval:` (the ruling or standing-approval id; for a candidate PR the merge is the
   approval) — replacing ADR 011's `Agent:`/`Run:` pair, and never a co-author line. Commits made
   by planning sessions keep the harness attribution until the user rules otherwise (open
   decision, below).

## Alternatives
- **Keep `routines/*.md` prompt files (ADR 007/013).** Rejected: a second shape that would be
  migrated later; the OS shape is already ruled and the cost of writing to it now is a directory
  per workflow instead of a file.
- **Wait for the OS before building any routine.** Rejected: 1c is how actor #1 is chosen, and
  the interim sequencer is a short function over files that already have to exist.
- **Import or vendor personal-os code.** Rejected: it builds only in its container; nothing here
  may depend on its internals; the boundary is the file shape, not code.
- **Write `VISION.md` now.** Deferred: the OS's project workflow derives it from the brief;
  `context/vision.md` serves the planning stage.

## Consequences
- A `library/` tree with four workflow directories (two written at blueprint time, two when
  their milestone arrives), `library/contracts/`, `.os/project.yaml`, `.os/tools.yaml`.
- `foundry run-workflow` as one more subcommand; retired when the OS registers the workflows.
- The OS's constraints apply from day one: agents never write their own files, know-how is
  manager-only, every workflow has a finisher, a loop is opt-in.
- Open decision for the user: whether this repo's planning-session commits should drop the
  harness's co-author line to match the OS rule.
