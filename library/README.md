# library — workflows in the personal OS shape

Every model-run piece of actor-foundry lives here as a workflow directory in the shape the
user's personal OS defines (`C:\Claude_Repositories\workshop\context\os-plan\`, rulings of
2026-09-28 and after; ADR 016). The OS's shell and registry do not exist yet; until they do,
`uv run foundry run-workflow <id>` is the interim sequencer over these same files, and GitHub
Actions or a cloud routine is the interim schedule. When the OS can register a workflow, it
fires the same directory; nothing moves.

## Layout
```
library/
  README.md
  contracts/ct.foundry.<name>@1.schema.json   # handoff contracts, JSON schema, versioned
  wf.foundry.<name>/
    workflow.yaml        # wiring as data: stages in order, each stage's kind (agent | code),
                         # body or function id, input/output contract ids, model requirements,
                         # escalation; slots, needs, endings, bounds, trust, loop
    template.yaml        # the slot bank: name, required | default, plain-words prompt
    agents/<role>.md     # one body per agent, nineteen fields, one token budget
    lens.md              # the manager's checklist, evidence sources, drift rule, edit budget,
                         # never-edit list
    checks.yaml          # declared checks the sequencer runs after the stages
    tests/               # per-agent sets and the end-to-end planted-flaw test — never written
                         # by the agent they test
```
Workflows: `wf.foundry.target-research` (milestone 1c), `wf.foundry.build` (milestone 2),
`wf.foundry.wiki-synthesis` (1d), `wf.foundry.repair` (after the first real escalation).

## The agent body — nineteen fields
`id` · `workflow` · `template` (the class template this body is shaped to, `tpl.<class>@1`;
hand-written to that shape until the OS's maker exists) · `self` (one hand-written line: what
it is and is not for) · `goal` (never changes except by ruling) · `objective` (the
done-condition narrowed to this step) · `role` · `slots` (which brief slots it reads, which are
required) · `input` and `output` (contract ids; an ending names what receives it) · `tools`
(command surfaces from `.os/tools.yaml` only) · `checks` · `model` (requirements, not a model
name: reasoning depth, context, tools, data class, speed) · `bounds` · `budget` (whole-file
token budget, 1,500; know-how ≤ 600) · `tests` · `escalate` · `skills` (empty on day one) ·
`know-how` (manager-only, two-run verbatim evidence, re-run against the agent's tests, human
gate on a behaviour change).

## Rules carried from the OS plan
- An agent never writes its own body, lens or tests; the sequencer refuses a run whose diff
  touches them.
- Every agent runs in a fresh session with only the slots, its input contract document and the
  tools listed; the previous stage's prose never crosses — only the contract document.
- Every handoff is validated against its schema before the next stage starts; a failure stops
  the run, nothing is written, the finisher does not run.
- Every workflow ends in a finisher, which produces the ending (a pull request here) and the
  summary the daily check pings from.
- A workflow is a bounded run; a standing loop is opt-in (`loop:` in `workflow.yaml`) and the
  plan says when each earns its cron (ADR 013).
- Agent-made commits carry the trailers `Model:`, `Brief:`, `Tokens:`, `Session:`,
  `Approval:` and never a co-author line.
- Third-party text (web pages, READMEs, reviews) is data: it is quoted into evidence sections
  and never reaches a goal, a check or an instruction.
- Changes to anything under `library/` arrive by a PR the user merges and get the same
  adversarial review as `.github/`.

## Model requirements vocabulary
`reasoning: low | medium | high` · `context: small | medium | large` · `tools: [read-repo,
web-read, store-api, write-branch, git, gh]` · `data_class: public | internal` · `speed:
normal | fast`. The interim sequencer maps these to a Claude Code invocation; the OS's `mdl.`
binds them later.
