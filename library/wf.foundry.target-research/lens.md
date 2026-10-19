# Lens — wf.foundry.target-research

The manager reviews every run through this lens. Its evidence is the reviewer's verdict and the
failed checks, never the run's own account of itself. It edits only `know-how` fields, within
the budget, with verbatim evidence from at least two runs, re-running the agent's tests before
any edit reaches the human gate.

## Checklist (research class)
1. Every demand signal in a note has a URL and a read date; every Store-gap figure traces to a
   snapshot actor id.
2. The exclusions were applied before scoring: login wall, non-English, private or sensitive
   personal data, named-client work — none appears in a note.
3. A terms line that forbids automated access is quoted and flagged, not used to exclude.
4. Scores follow `candidate-card.md` exactly; Base = Demand × Gap; Growth only when Base > 0;
   the saturation band is a hand count of titles, never the Store `search=` total.
5. The frontmatter, **Proposed output**, **Suggested fixture page** and **Proposed listing** are
   the agent's words; every other section is quotation and data.
6. **Your changes** is empty.
7. Nothing outside `wiki/candidates/**` changed; the PR carries the five trailers.

## Evidence sources
The reviewer's verdict on the PR (a non-authoring session), the failed checks from
`checks.yaml`, the user's edits recorded in **Your changes** and `rulings-log.md` (what the
user changed is the strongest evidence of what the writer got wrong).

## Drift rule
A pattern is two runs with the same failed check or the same kind of user edit; one run is a
one-off and gets no edit.

## Edit budget
Know-how ≤ 600 tokens per body; at most two edits per review.

## Never edit
`goal`, `slots`, `input`, `output`, `tools`, `checks`, `tests`, the contracts, this lens.
