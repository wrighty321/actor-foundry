# 014 — Listing congruency and daily-test alignment, generated and linted from the fixture
Status: proposed (quality-score pass and its Kun pass, 2026-10-18)
Adds to: 002 (the fixture is the vocabulary), 004 (lint joins the PR gate), 009 (the preflight page is the prefill page); amends B14 and B15

## Context
- Apify scores eight categories; its docs say reliability and feedback weigh most, and
  congruency of texts is one of the eight (docs.apify.com/actors/publishing/quality-score,
  read 2026-10-18).
- Apify's daily test runs the actor with its `prefill` input and expects `Succeeded` with a
  non-empty default dataset within 5 minutes (docs.apify.com/actors/publishing/test, read
  2026-10-18). The input-schema spec says `prefill` is used only in the UI; `default` is what the
  platform passes when a caller omits the field (reviewer's read, 2026-10-18). An agent calling
  through MCP with minimal input would otherwise run a different job from the one Apify tests.
- A task cannot be published without a dataset view (API reference `actor-tasks-post`, read
  2026-10-18); the dataset schema also feeds congruency.
- The fixture already defines the vocabulary: `fields`, `live_pages`, `min_rows`; L1 already
  asserts that `extract` output keys equal `fields` (ADR 002). The README, dataset schema and
  input schema have no deterministic link to it yet.
- The README doc says "aim for at least 300 words" and names no mechanism; a word-count gate
  rewards padding, which is the kind of proxy this project refuses to optimise.
- `title`, `description`, `seoTitle`, `seoDescription`, `categories` and `name` are Actor
  properties settable by API or Console; because `name` (the slug) is in the Update Actor body,
  the only guard against an accidental slug change is that nothing automated ever sends it.

## Decision
1. **One generator.** `.actor/dataset_schema.json` is generated from the fixture's `fields`:
   one view, `overview`, whose fields and types are exactly `fields`. Nothing else is generated;
   the README stays human-written.
2. **Prefill and default from the fixture.** The input schema's start-URL field and `maxItems`
   carry both `prefill` and `default`, set to `live_pages[0]` and `min_rows`. Apify's daily test,
   a bare API call with `{}` and our own L2 run therefore run the same job. The stub week
   measures that run: target under 2 minutes, non-empty dataset, recorded in the build plan.
3. **One lint, `foundry lint`,** in the PR CI job beside `pytest` and `ruff`, and again at deploy:
   - README: only H2/H3 headings (the actor name is the H1 Apify renders); required sections
     present in order — what it does and the fastest way to try it; a step-by-step tutorial;
     "How much does it cost to …" with a worked example at the current price per 1k results
     plus the start fee; Input example; Output example; links to our other actors; FAQ with
     legality (the lawful-basis line from the candidate note), troubleshooting, integrations;
     Output example keys equal `fields`; the cost example's price equals `pricingInfos` on the
     FREE tier (checked at deploy, where the read-back already exists).
   - Input schema: every field has a description; `prefill` and `default` validate against the
     schema and equal the fixture-derived values; `required`, `enum` and ranges are present
     where the fixture's `fields` types imply them.
   - Dataset schema: fields equal `fields`.
   - Listing record (`actors/<name>/listing.md`, below): title 40–50 characters matching
     `{Site} {Service} Scraper` or `{Site} Scraper`; description ≤ 300 characters; SEO title
     ≤ 60; SEO description 145–155; slug unchanged once `published_on` is set.
   - Lint never counts words or keywords.
4. **A listing record, not a listing writer.** `actors/<name>/listing.md` records what the user
   set in the Console at publish: `title`, `slug`, `description`, `seo_title`, `seo_description`,
   `categories`, `published_on`, `kyc_passed_on`. `.actor/actor.json` carries `title` and
   `description` and rides `apify push` (**assumed**; verified on the stub). Deploy never sends
   `name`, `isDeprecated`, `isPublic`, `pricingInfos` or the SEO fields — it reads them back and
   reports mismatches with the record.

## Alternatives
- **Generate README fragments from `fields`.** Rejected: a writer, template zones and a build
  step to do what one equality check in lint does.
- **Word-count and keyword lint.** Rejected: metric theatre; a padded README is worse for the
  buyer and no better for the score's stated mechanics.
- **`listing.yaml` applied by deploy through Update Actor.** Rejected: an automated listing
  writer holding the unscoped token for fields that change once; a Console edit at publish and a
  record in the repo cost less and keep `name` out of any automated body.
- **`prefill` only.** Rejected: UI-only; API and MCP callers would run a different job.
- **No dataset schema.** Rejected: a task cannot be published without a view, and the schema is
  one of the texts congruency scores.

## Consequences
- One generator and one lint; the README contract moves from B14 into `wiki/playbook/` as a
  checklist the lint enforces.
- The lint job must be named in the PR CI workflow (flow trace, 2026-10-18); deploy runs it again.
- The stub week verifies that `actor.json` title/description ride `apify push`; if not, the
  record notes a Console step for them too.
- A fixture change (new `fields`) regenerates the dataset schema and fails lint until the
  README's Output example is updated by hand — which is the point.
