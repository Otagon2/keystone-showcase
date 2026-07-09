# Day 3 — design decisions & why

The reasoning behind today's build, decision by decision. Companion to the
[feedback-response matrix](day3-zaki-feedback-response.md) and the
[competitor coverage table](day3-datasets-vs-competitors.md).

## Shell

- **Top bar = page header, no breadcrumbs.** The loop is non-linear; a trail
  implies order. Detail pages get exactly one ← (parent list). Grounded in the live
  teardown: neither Braintrust's list pages nor Arize show a trail; Braintrust's
  detail shows a single parent link — the same shape.
- **Portal slots, not a config store, for the header.** Pages render their title/
  CTAs straight into top-bar slots, so actions can never go stale and there's no
  shared state to desync.
- **Search: fixed width, fixed position.** Anchored beside the avatar so it sits at
  identical pixels on every screen (reviewed live: it previously shifted with each
  page's CTA width). The ⌘K hint is a uniform mono kbd chip.
- **Sidebar:** brand block (K · Keystone · Esberi/AskJolly) → org/workspace
  switchers → global **+ New** → stage nav (Build · Evaluate · Observe · soon:
  Operate/Learn/Manage) → collapsible to a 56px icon rail (`[`, persisted). The
  "+ New" and identity block follow the Databricks reference shared in review; the
  account card moved **top-right** instead (it now owns Inbox, Theme and
  Organization settings, so it earns the more discoverable corner).
- **Theme Light/Dark/System** lives in the account menu; System subscribes to
  `prefers-color-scheme` so it tracks the OS live.

## Datasets (PRD #5)

- **Three label-like concepts, three controls — never merged.** Labels
  (`team:support`) classify the dataset; **tags are movable version pointers**
  (`golden → v3`) that experiments can follow; record tags slice rows. The list
  renders tags as `name → vN` pills precisely so nobody mistakes them for labels.
- **Versions are human numbers with change notes.** Braintrust shows an opaque
  transaction id (`1000197481081881572`); Arize shows timestamps. An experiment
  pins `v3` — a number a reviewer can say out loud.
- **Working draft → Save version.** Grid edits autosave to a draft; a version is an
  explicit commit with a note. Keeps versions meaningful (not one per keystroke) and
  makes "experiments can't pin a draft" a visible rule.
- **Provenance + Origin on every example.** human / imported / captured / synthetic
  badges; captured rows keep a clickable Origin chip to their source trace, and
  synthetic rows are flagged until reviewed. Neither competitor surfaces where a row
  came from.
- **Delete = type the name.** Both competitors ship a plain confirm; the PRD's
  proportional-friction rule is stricter and we follow it.
- **Import ends in a sentence.** "8 inserted · 0 updated · 1 skipped (empty input)"
  — Braintrust's import completes silently and even renames the dataset after the
  file, which we deliberately do not.

## Prompt Management (PRD #1 completion)

- **The runner binding gates Run.** Model (Hosted/Local groups) + credential chosen
  from project **Secrets by name** (never a pasted key; "+ Add API key" creates a
  write-once secret) or an endpoint for local models. Run stays disabled until the
  gates pass, with the PRD's exact empty-state line.
- **Typing `{{variable}}` IS the schema.** Segments are editable textareas; inputs
  and `@[references]` parse live into the Inspector. No separate "define inputs"
  form to drift out of sync.
- **Single | Batch | Compare are one mode with a toggle,** per the PRD ("comparing
  is running more than one variant"). Compare highlights diffs vs the first column
  and lets you mark winners per cell.
- **Inspect answers "why did it do that":** the assembled prompt as sent,
  request/response JSON, token breakdown (input/output/cached), a step timeline, and
  a comment thread (@mention/assign/resolve) on the captured run.
- **Add to Dataset from any result** — the PRD #5 seed flow surfaced inside PRD #1:
  input→input, output→expected, origin backlink, upsert.

## Data integrity

- **Experiment stats derive from rows.** `passed / total / passRate` are computed
  from the row set at module load — the headline can no longer disagree with its own
  table (the Day-2 audit's #1 finding, now structurally impossible).

## Flagged inventions (naming questions for Zaki)

These exist in the prototype but aren't specified in any PRD — treat as proposals:

- CLI/SDK naming: `@keystone-ai/cli`, `@keystone-ai/sdk`, `keystone_ai` (PyPI),
  `npx skills add esberi/keystone-skills`, `api.keystone.dev/v1`.
- API key format `ks_live_…` and env var `$KEYSTONE_API_KEY`.
- The promote gate threshold (80% on the eval dataset) is a demo policy, not canon.
