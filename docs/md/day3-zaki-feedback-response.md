# Day 3 — Zaki's review, answered point by point

Every item from the July 9 review, with what changed and why. All live in the
[prototype](../keystone/).

## 1 · "Breadcrumbs are a bad design choice"

**Before:** a `AskJolly / Datasets / GST FAQ` trail in the top bar, plus a
Dataset → Prompt → Evaluator → Experiment → Trace ribbon on every list page that
*read* as a second breadcrumb.

**After:** both removed. Detail pages carry a single **← back** control (one level
up, never a trail) in the top bar. The loop ribbon is gone entirely — the process
isn't linear, so nothing in the chrome should imply an order.

**Why:** breadcrumbs encode hierarchy the product doesn't have (the loop is a graph,
not a path), and they burned the most valuable row of the screen on wayfinding that
the sidebar already does.

## 2 · "Section headers and CTAs belong in the top bar"

**Before:** every page spent ~90px on an in-body `h1` + description + buttons; the
top bar held only breadcrumbs and a theme toggle.

**After:** the top bar IS the page header: `[collapse] [← back] [Title + version/
badges] … [page CTAs] [Search ⌘K] [account]`. Pages register their title and
actions; content starts at the data. Where a description carried real information it
survives as one muted line above the content.

**Why:** the page identity and its primary verbs are now always visible (no scroll),
and every screen gained ~90px of content height.

## 3 · "The theme toggle isn't used all the time — move it"

**Before:** a Sun/Moon button permanently occupying top-bar space, two states only.

**After:** the account menu carries a **Theme** submenu with **Light · Dark ·
System** (system tracks the OS preference live). Also reachable from ⌘K
("Theme: …").

**Why:** set-and-forget controls belong behind the account, not on the prime row —
and "System" was missing entirely.

## 4 · "The sidebar has no hide/open mechanism"

**Before:** fixed 240px rail.

**After:** collapses to a 56px icons-only rail (tooltips on hover), toggle button in
the top bar + the `[` shortcut, state persisted across sessions. The rail also
gained the Keystone brand block, a global **+ New** create button
(Databricks-inspired, per the shared reference), and the PRD's canonical stage
grouping — Build · Evaluate (Datasets · Evaluators · Experiments) · Observe, with
Operate/Learn/Manage marked as coming.

**Why:** power users reclaim ~184px for tables; Arize and Databricks both treat
collapse as table-stakes.

## 5 · "Datasets needs search, filters, the full table, and three create paths"

**After — the list:** stats bar (datasets · curated · records · experiments) ·
search by name/label · **Filters** (curated/ad-hoc, include-archived, created-by)
with removable chips · table with **# · Name (+curated badge) · Description ·
Examples · Created by · Last updated · Tags (version pointers) · Labels · Metadata ·
url_slug · Last experiment**, controlled by a persisted **Display** column menu ·
bulk select with Tag/Export/Archive · a full row **⋯** menu (Open · Run experiment ·
Duplicate · Rename · Edit labels · Export · Archive · Delete-with-typed-name).

**After — New dataset:** one dialog, three methods:
- **Upload CSV** — drop zone → drag-to-bucket mapper (Input · Expected · Metadata ·
  Tags · ID · Not imported) with a keyboard-accessible per-column menu, id-upsert
  notice, live preview, and an import summary ("8 inserted · 0 updated · 1 skipped").
- **Create via skill** — npx instructions for an AI coding agent + **Create API key**.
- **Create via code** — CLI · TypeScript · Python · REST snippets templated on the
  typed name, + **Create API key** (shown once, copy-only).

**Why:** this is the Arize dialog shape (which the review referenced) with
Braintrust's mapper — plus the parts neither ships (see the competitor coverage doc).

## 6 · "Improve the upload CSV flow"

**Before:** a one-shot mapping table with select dropdowns.

**After:** the drag-to-bucket mapper above, embedded in both create (new dataset)
and import-into-dataset (upsert, commits as a new version with a change note).
Braintrust's mapper is drag-*only* — ours adds a per-column menu so the same flow
works by keyboard and on touch, and ends with an explicit result summary instead of
rows silently appearing.

## Also shipped on the same review

- **Dataset detail rebuilt to PRD #5's editor frame:** pills **Items · Versions ·
  Experiments · Manage**; a working-draft banner (edits autosave, **Save version**
  commits v-next with a change note); version table with restore/tag/diff; an
  experiments launcher that **pins a version** (or follows a tag like `golden → v3`);
  Manage with schema (curated ⇄ ad-hoc + invalid-rows state), access and a typed-name
  delete. Every example carries **provenance** and, when captured, an **Origin** chip
  back to its source trace.
- **Prompt Management finished to PRD #1** (see the design-decisions doc): editable
  message segments with live `{{variable}}` parsing, segment ⋯ menus, the runner
  binding (model + Secret-by-name credential gating), Single/Batch/Compare run modes,
  Inspect with request/response + timeline + comment threads, Add-to-Dataset from any
  result, pin-to-sidebar that really pins.
- **Scorecard integrity:** experiment headlines now **derive from their own rows**
  (passed/total/pass-rate computed, not typed) — the 6-vs-12 row drift class of bug
  can't recur.
- **Inbox** (PRD #3) stub with the needs-action / FYI split, reachable from the
  account block.
