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
| App E · JSON / JSONL files | 🟡 | Accepted by the picker; parser currently handles CSV (sample path covers demos) |
| **App F · Capture from traces / Seed from runs** | ✅/🟡 | Seed-from-run is fully clickable (Add to Dataset from any run result, origin + upsert); capture-from-traces is an explainer dialog → Traces (span multi-select mapping deferred) |
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
| App D · Segment menu: move · duplicate · delete · copy · insert reference · to/from placeholder | 🟡 | All but to/from-placeholder conversion (add/delete covers the demo) |
| App D · Inspector: Output (multi-type), Inputs, References, Token budget + cost | ✅ | |
| App D · Quick-run panel, gated, Capture | ✅ | PRD's exact empty-state line when ungated |
| App D · Icon toolbar (undo/redo/preview/import/export) | ⏳ | Chat/Text toggle + quick-run shipped; the rest deferred |
| **App E · Run** — Single / Batch / Compare toggle | ✅ | |
| App E · Batch input sources (add row · load from dataset · generate · paste) | 🟡 | Load-from-dataset shipped; add-row/paste/synthetic-generate deferred |
| App E · Compare — columns, diff-vs-first, mark winner, Run as Experiment | ✅ | |
| App E · Inspect — assembled prompt, request/response, token breakdown, timeline, comment thread (@mention/assign/resolve) | ✅ | |
| App E · Result ⋯: Inspect · Capture · Add to Dataset · Copy output · Mark winner | ✅ | Add-to-Dataset = PRD #5's seed flow, with origin |
| **App E-runner** — hosted/local groups, credential from Secrets by name, + Add key (write-once), endpoint, run gating, scope (run-only vs version default) | ✅ | Params are chat-task (temp/max-tokens); image-gen param swap deferred with multimodal |
| JTBD-9 multimodal output (image/file rendering) | ⏳ | Output types declarable; renderers deferred (PRD itself defers audio/stream) |
| **App F · Compress** — 3 method cards, question field (LongLLMLingua), keep-rate ⇄ target tokens, preserve digits, protected variables, stats, apply-as-version | ✅ | |
| **App G · Manage** — versions (diff/restore/promote, comment counts), env slots, gated promotion, evaluator card, associated datasets, access, archive/typed delete, DSPy button | ✅ | Label taxonomy curation (promote/rename/merge) deferred |
| JTBD-13/14 comments & presence | 🟡 | Run-comments shipped in Inspect; version threads shown as counts; presence deferred (P2) |
| **App H · Dialogs** — rename, promote (env), export (JSON/YAML), archive, typed delete | ✅ | |
| **App I · ⌘K** — go-to, create, search index | 🟡 | Go-to/create/theme + full entity search; context-aware "Do" verbs and Recents deferred |

## The short version

Every **P0 job** in both PRDs is clickable end-to-end. The deferred items are the
PRD's own P1/P2 tails (custom columns, metadata-value filters, multimodal renderers,
presence, materialization) plus backend-shaped work (virtualization, server-side
filtering) that a mock shouldn't fake — each is listed above rather than silently
dropped.
