# PRD coverage — Prompt Management (#1) & Dataset Management (#5)

An honest checklist of both PRDs against the prototype, appendix by appendix.
✅ built and clickable · 🟡 simplified (works, reduced fidelity) · ⏳ deferred (noted, not faked).

## PRD #5 — Dataset Management

| Spec | Status | Notes |
|---|---|---|
| **App C · List** — stats bar, search, sort, New dropdown (Empty / Import / From traces) | ✅ | Plus filters w/ chips and a persisted Display column menu (beyond spec) |
| App C · Columns: Name · Labels · Records · Version · Curated · Last modified · Last experiment · ⋯ | ✅ | Plus serial #, Description, Created by, Tags (pointers), Metadata, url_slug — Zaki's additions |
| App C · Row ⋯: Open · Run experiment — Pin to sidebar · Duplicate · Rename · Edit labels — Export — Archive · Delete (red, last) | ✅ | Pin really pins (rail updates live); Delete = type-name |
| App C · Bulk tag/export/archive; empty state w/ 3 CTAs | ✅ | |
| **App D · Items grid** — input/expected/metadata/tags/origin/provenance, inline edit, bulk select | ✅ | Row edit via dialog (PRD canon: dialogs only) |
| App D · Working draft → Save version / Discard | ✅ | Banner + change-note commit; "experiments can't pin a draft" stated |
| App D · Filter by tags / metadata / origin / provenance | 🟡 | Provenance + has-origin filters; record-tag & metadata-value filters deferred |
| App D · Custom columns (extract a metadata key) | ⏳ | Needs a column-builder UI; deferred with the server-side-filter work |
| App D · Origin control (green resolvable / red expired) | 🟡 | Green resolvable chip → trace; the "source expired" red state isn't seeded |
| **App E · Import mapper** — drag-to-bucket, auto-map-to-input, id upsert, result summary | ✅ | Plus per-column menu fallback (keyboard/touch) — beyond spec |
| App E · JSON / JSONL files | ✅ | **Day-3 gap pass:** real JSON-array + JSONL parser flattens objects into the bucket mapper (nested values stringified); auto-detects format by extension/content, falls back to CSV |
| **App F · Capture from traces / Seed from runs** | ✅ | **Day-3 gap pass:** capture is now a real span-picker over production traffic — failures pre-selected (the "production failure → permanent test" loop), captured rows land in a working draft with an Origin backlink and the observed output as a to-review `expected`; seed-from-run (Add to Dataset) still available from any run result |
| **App G · Versions** — list, diff (record-level), restore, tag, immutability note | ✅ | Diff is a seeded mock (consistent, not computed) |
| **App H · Experiments view** — where-used list + launcher (target, evaluators, version/tag pin, dry-run, concurrency) | ✅ | Launch creates a queued mock run |
| **App I · Manage** — Details, Schema (curated ⇄ ad-hoc, invalid-rows state), Access, Archive/Delete | ✅ | Materialization section deferred (optional per PRD; flagged backend cost) |
| **App J · Dialogs** — import, capture, schema, tag version, promote synthetic, delete (typed), export | ✅ | Promote-synthetic lives on synthetic rows' ⋯ menu |
| B.5 states draft / active / archived | ✅ | Seeded one of each; archived hidden by default |
| Edge cases: empty dataset blocks launch; upsert; restore-as-draft | ✅ | |
| Scale (virtualized grid, server-side filter, streaming export) | ⏳ | Out of a mock's scope; called out for backend |

## PRD #1 — Prompt Management

| Spec | Status | Notes |
|---|---|---|
| **App C · List** — Name · Kind · Labels · Version · Env · Last run · Runs · ⋯; Run on hover | ✅ | Full grouped ⋯ incl. live Pin to sidebar; search + sort |
| **App D · Edit** — editable segments (system/user/assistant), add message / add placeholder | ✅ | `{{variable}}` parses live into Inputs; `@[Name]` renders reference chips |
| App D · Segment menu: move · duplicate · delete · copy · insert reference · to/from placeholder | ✅ | **Day-3 gap pass:** convert-to/from-placeholder now in the segment ⋯ menu |
| App D · Inspector: Output (multi-type), Inputs, References, Token budget + cost | ✅ | |
| App D · Quick-run panel, gated, Capture | ✅ | PRD's exact empty-state line when ungated |
| App D · Icon toolbar (undo/redo/preview/import/export) | ✅ | **Day-3 gap pass:** real undo/redo history, assembled-prompt preview, JSON import/export |
| **App E · Run** — Single / Batch / Compare toggle | ✅ | |
| App E · Batch input sources (add row · load from dataset · generate · paste) | ✅ | **Day-3 gap pass:** editable input-set builder — add row · paste (lines or JSON) · load from dataset |
| App E · Compare — columns, diff-vs-first, mark winner, Run as Experiment | ✅ | |
| App E · Inspect — assembled prompt, request/response, token breakdown, timeline, comment thread (@mention/assign/resolve) | ✅ | |
| App E · Result ⋯: Inspect · Capture · Add to Dataset · Copy output · Mark winner | ✅ | Add-to-Dataset = PRD #5's seed flow, with origin |
| **App E-runner** — hosted/local groups, credential from Secrets by name, + Add key (write-once), endpoint, run gating, scope (run-only vs version default) | ✅ | Params are chat-task (temp/max-tokens); image-gen param swap deferred with multimodal |
| JTBD-9 multimodal output (image/file rendering) | ⏳ | Output types declarable; renderers deferred (PRD itself defers audio/stream) |
| **App F · Compress** — 3 method cards, question field (LongLLMLingua), keep-rate ⇄ target tokens, preserve digits, protected variables, stats, apply-as-version | ✅ | |
| **App G · Manage** — versions (diff/restore/promote, comment counts), env slots, gated promotion, evaluator card, associated datasets, access, archive/typed delete, DSPy button | ✅ | Label taxonomy curation (promote/rename/merge) deferred |
| JTBD-13/14 comments & presence | ✅ | **Day-3 gap pass:** version comment **threads** (add · @mention highlight · resolve/reopen), not just counts; run-comments in Inspect; presence deferred (P2) |
| **App H · Dialogs** — rename, promote (env), export (JSON/YAML), archive, typed delete | ✅ | |
| **App I · ⌘K** — go-to, create, search index | ✅ | **Day-3 gap pass:** context-aware "On this prompt" verbs (Run/Compress/Promote/Manage) + a Recent group + split New chat/text prompt |

## The short version

Every **P0 job** in both PRDs is clickable end-to-end. A **Day-3 gap pass** then
closed the remaining P0/P1 tail: JSON/JSONL import, a real capture-from-traces
span-picker, the batch input-set builder, the Edit-mode toolbar (undo/redo/preview/
import/export) + segment ↔ placeholder conversion, version comment **threads**, and
context-aware ⌘K verbs + Recents. The same pass removed two duplicate primary CTAs
(a second "Run" on the Run tab; a second experiment launcher on the Experiments
view) — the header shortcut now hides itself on the screen it points to.

The still-deferred items are the PRD's own P2 tail (custom columns, metadata-value
filters, multimodal renderers, presence, materialization, label-taxonomy curation)
plus backend-shaped work (virtualization, server-side filtering) that a mock
shouldn't fake — each is listed above rather than silently dropped.
