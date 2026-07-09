# Day 3 — Live Datasets teardown: Braintrust vs Arize AX (2026-07-09)

Live walkthrough on both platforms (Esberi/AskJolly accounts), same GST FAQ sample
CSV (12 rows) imported into each, then deleted. Screenshots in `braintrust/` and
`arize/`. Goal: ground Keystone's Day-3 Datasets build (PRD #5 + Zaki's feedback)
in what each actually ships — copy / adapt / avoid.

**Headline:** Zaki's asks map 1:1 onto what these two ship — the dataset-list
columns and serial # come from Arize; the Display column menu and drag-mapper come
from Braintrust; the "Upload CSV / Create via skill / Create via code (CLI · TS ·
Python · REST) + Create API key" creation dialog is Arize's New Dataset dialog
almost verbatim. He's asking us to combine the best of both — and both leave gaps
Keystone can beat.

## 1 · Dataset LIST

| Facet | Braintrust | Arize AX | Keystone verdict |
|---|---|---|---|
| Columns | ☑ · star/pin · Name · Description · Updated · Examples · Metadata (raw JSON!) · url_slug · Tags; drag-reorder headers | # serial · Name · #Examples · #Experiments · Created By (avatar) · Last Updated↓ · Tags · row-run icon · ⋯ | **Combine**: serial + name + description + examples + created-by + updated + tags + labels + metadata + slug; hide noisy ones by default |
| Column control | **Display → Columns ("1 hidden") + Group by** | none | **Copy BT** Display menu (+ persistence); Group-by = later |
| Search / filter | "Add filter" combobox (field-based) + saved **views** ("All datasets view") | Search-by-name only | **Copy BT** filterbar idea (simplified: label/state/creator + chips); saved views = later |
| Bulk | header + row checkboxes (no visible action bar until select) | none | **Beat**: selection toolbar w/ Tag/Export/Archive |
| Row ⋯ | (list) none seen — actions live on detail | **just "Delete"** | **Beat**: full PRD menu (Open · Run experiment · Duplicate · Rename · Edit labels · Export · Archive · Delete) |
| Stats bar | none | none | **Beat**: PRD App. C stats bar (datasets · curated · records · experiments) |
| Metadata col | raw `{"__schemas":…}` string — unreadable | n/a | **Beat**: `{…}` chip → key:value popover |

## 2 · CREATE flow

**Braintrust:** `+ Dataset` → dialog with ONE field (name) → Create → lands in empty
dataset → populate there ("create a row / upload a CSV/JSON file / insert data
programmatically(docs link)" + drop zone). No method choice up front; programmatic
path just links to docs; **no API-key affordance in-flow**.

**Arize:** `New Dataset` → one dialog, left method list: **Upload CSV · Create Via
Skill · Create Via Code**:
- *Upload CSV:* Dataset Name + drop zone + Download Sample.csv → after parse: a
  "Dataset Schema" section with a single **reference** column select ("typically
  ground truth… use {dataset.reference} in eval templates") + 12-row preview + Clear
  file → Create.
- *Create Via Skill:* "Copy and paste into your terminal" + **Create API Key** +
  terminal block: `npx skills add Arize-ai/arize-skills` then *ask your AI coding
  agent: "Create a dataset named my_dataset with examples from data.csv"*.
- *Create Via Code:* inner tabs **CLI · TypeScript SDK · Python SDK · REST API**,
  each stepped blocks (install → auth/profile → create) with Copy All + **Create
  API Key**. CLI: `pip install arize-ax-cli` · `ax profiles create` · `ax datasets
  create --name … --file data.csv`. TS: `npm install @arizeai/ax-client`,
  `client.datasets.create({spaceId, name, examples})`. REST: python-requests POST
  `https://api.arize.com/v2/datasets`.
- Post-create ⋯ menu keeps the pattern alive: "Get Dataset Via Agent (Skills)" ·
  "Get Dataset Via Code".

**Verdicts:** Copy Arize's 3-method dialog shape + API-key-in-flow + sample-file
link (Zaki's spec). Beat both on the mapper (below) and add a result summary.
Adopt Arize's "via skill/code persists on the detail ⋯" idea.

## 3 · CSV column mapping

**Braintrust** (the reference): full-page import surface — left: raw source preview
+ live import-preview table (ID/Input/Expected/Tags/Metadata); right: buckets
**Input · Expected · Metadata · Tags · ID · Do not import** with draggable column
chips; auto: `id`→ID, everything else→Input ("Move all to input" btn); toggles
"Flatten single-column values" / "Auto-parse objects in strings"; header "Upload
all 12 rows ▾" + Import. **Friction found live:** chips move ONLY by pointer drag —
no keyboard/click alternative (synthetic drag failed; selects text instead), and
import completes with **no summary** (rows just appear; dataset silently renamed
itself from the filename). Version after import = opaque txid `1000197481081881572`.

**Arize:** no buckets at all — columns import as-is; single **reference** select
designates ground truth; preview table. Fast but no metadata/tags mapping, no id
dedup story surfaced.

**Keystone:** drag-to-bucket like BT **plus a per-chip assign dropdown** (keyboard/
mobile accessible), auto-detect question/answer, explicit **id-upsert notice**, and
a **result summary ("N inserted · M updated · K skipped")** — beats both.

## 4 · DETAIL surface

| Facet | Braintrust | Arize | Keystone |
|---|---|---|---|
| Frame | name + Saved + VERSION txid + Tag · Field schemas · Review · Snapshots · **Evaluate in ▾**; toolbar Import · +Row · views · filter · Display; right rail Details/Runs (desc + auto-inferred `__schemas` YAML + "Recently used in") | back + name + Info · tag · **Version = timestamp dropdown** · Add Evaluator · **+ New Experiment** · ⋯; tabs **Examples · Experiments**; "Example Query" condition filter | PRD pills **Items · Versions · Experiments · Manage** + icon cluster |
| Versions | Snapshots (named), plus txid per write | timestamped auto-versions dropdown | **Beat**: human v1/v2/v3 + change notes + tags-as-pointers + diff/restore (PRD) |
| Row detail | side panel: Fields/Runs/Views, YAML editors, Assign, per-row **Activity comments**, prev/next | (not deeply inspected; grid has Status/Example ID/Kind/Name/Input…) | dialog-based edit (PRD: dialogs, no drawers) + origin chip + provenance |
| Delete | plain confirm ("Are you sure…") | plain confirm | **Beat**: type-name-to-confirm (PRD) |
| Experiments link | "Evaluate in ▾" / Runs tab | Experiments tab + New Experiment | Experiments pill + launcher pinning a version |

## 5 · Shell notes (Zaki's nav feedback traced to sources)

- **Arize sidebar**: has **Collapse navigation** (icons-only rail) — exactly what he
  wants ours to add. Groups: Observe / Evaluate / Improve + More; Settings·Help·
  account at bottom.
- **Braintrust sidebar**: org switcher top, project section, Settings bottom; has a
  collapse icon top-left as well.
- **Top bar**: both put the page/section title in the top bar (BT: "Datasets" +
  Docs/Resources/avatar right; Arize: big page title + description + page CTAs
  top-right). **Neither shows breadcrumbs on the list**; BT's detail shows
  "Datasets" as a single back-style link in the top bar — matches Zaki's
  "no breadcrumb trail, title+CTAs in the top bar" direction.
- **Theme toggle**: in neither top bar. (BT: appearance in account menu; Arize:
  dark-only aesthetic.) Supports moving ours into the account menu w/
  Light/Dark/System.
- Both ship an **AI assistant** (BT "Loop agent" pill bottom-right; Arize "Ask
  Alyx" panel + homepage chat). Out of scope today; note for Zaki.

## 6 · What Keystone will do better (the beat-list for docs)

1. Full row ⋯ action menu on the list (Arize: Delete-only; BT: none).
2. Stats bar + filters + Display columns together (each has at most one).
3. Accessible mapper (drag AND per-chip dropdown) + id-upsert notice + import
   result summary (BT: drag-only, silent import; Arize: no mapping at all).
4. Human versioning: v1/v2/v3 + change notes + movable tags (`golden→v3`) + diff +
   restore (BT: txids/snapshots; Arize: timestamps).
5. Provenance & origin on every example (neither surfaces where a row came from).
6. Type-name-to-confirm delete (both: plain confirm).
7. Working-draft → Save version commit model (neither separates edits from commits).
8. Readable metadata (key:value popover vs BT's raw JSON string in a cell).
