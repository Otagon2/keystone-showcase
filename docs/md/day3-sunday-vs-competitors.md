# Keystone vs Braintrust · Arize AX · Phoenix — Datasets & Prompt Management

Grounded in a hands-on teardown of all three (see `keystone-competitor-review/`), mapped to
the axes Zaki set: **ease of use · new customer · power customer · experience over time (large
datasets) · flows.** ✅ = does it well · 🟡 = partial/awkward · ❌ = missing.

## First-run / time-to-first-dataset  (new customer)

| Capability | Keystone | Braintrust | Arize AX | Phoenix |
|---|---|---|---|---|
| One-click sample dataset | ✅ bundled GST sample, opens in ~0.4s | ❌ | ❌ | ❌ |
| Column mapping | ✅ labeled buckets + live preview + drag, defaults to **your** names | ✅ drag buckets + preview (~2 min) | 🟡 single `reference` field, OTel default | 🟡 same as Arize |
| Guided empty state | ✅ "New here?" onboarding + 3-step framing | 🟡 checklist, nav hidden | 🟡 dense home | 🟡 |
| Time-to-first-dataset shown | ✅ prints `… in 0.4s` | ❌ | ❌ | ❌ |

## Large datasets  (power customer · experience over time)

| Capability | Keystone | Braintrust | Arize AX | Phoenix |
|---|---|---|---|---|
| Real scale in the grid | ✅ 10,000 rows, instant | 🟡 not demonstrated | 🟡 | 🟡 |
| Instant search over all rows | ✅ debounced, "filtered from 10,000" | 🟡 | 🟡 | 🟡 |
| Structured filter builder | ✅ field·op·value chips | ✅ add-filter | 🟡 | 🟡 |
| Column show/hide | ✅ persisted | ✅ Display | 🟡 | 🟡 |
| Select-all-across-pages | ✅ explicit escalation ("select all N matching") | 🟡 | 🟡 | ❌ |
| Sample / quick-scan mode | ✅ Sample 100 | ❌ | ❌ | ❌ |
| Honest counts | ✅ "Showing 1–50 of 10,000 (filtered from …)" | 🟡 | 🟡 | 🟡 |

## Data quality / "all instances"  (ease of use)

| Capability | Keystone | Braintrust | Arize AX | Phoenix |
|---|---|---|---|---|
| Pre-commit issue panel | ✅ missing-Expected · empty-input · dupe-id upsert · ignored cols · huge-file | 🟡 | ❌ silent | ❌ silent |
| Duplicate IDs → upsert (no dupes) | ✅ explained inline | ✅ | 🟡 | 🟡 |
| Malformed / messy import demo | ✅ one-click "messy export" sample | ❌ | ❌ | ❌ |
| Archived = read-only banner | ✅ | 🟡 | 🟡 | 🟡 |
| Empty-filter / empty-state copy | ✅ "no rows match — clear filters" | 🟡 | 🟡 | 🟡 |

## Governance over time

| Capability | Keystone | Braintrust | Arize AX | Phoenix |
|---|---|---|---|---|
| Numbered versions + diff | ✅ | ✅ | 🟡 | 🟡 |
| Tags as movable version pointers | ✅ golden → v3 | 🟡 | ❌ | ❌ |
| Labels (key:value classifiers) | ✅ | 🟡 | 🟡 | 🟡 |
| Per-row provenance | ✅ human/imported/captured/synthetic | ❌ | 🟡 | 🟡 |
| Origin link back to source trace | ✅ green chip → trace | 🟡 secondary | ❌ | 🟡 |
| Capture "prod failure → permanent test" | ✅ span-picker loop | 🟡 | 🟡 | 🟡 |
| Experiments pin a version (immutable results) | ✅ | 🟡 ephemeral runs | ✅ run≡experiment | 🟡 |

## Prompt Management  (both areas, equal depth)

| Capability | Keystone | Braintrust | Arize AX | Phoenix |
|---|---|---|---|---|
| Library with search/filter/saved-views/paging | ✅ | ✅ | 🟡 | 🟡 |
| Playground: inline evaluators + per-cell score | ✅ | ✅ 👍👎 + scorers | ✅ | 🟡 |
| Per-cell cost / latency / tokens | ✅ | ✅ | ✅ | 🟡 |
| Save-as-version from a run | ✅ | ✅ | 🟡 | 🟡 |
| Promote-to-experiment | ✅ explicit (not "save or lose it") | 🟡 ephemeral footgun | ✅ auto | 🟡 |
| Version comment threads | ✅ @mention + resolve | 🟡 | 🟡 | ❌ |
| Cross-run regression trend | ✅ pass-rate-over-runs chart | ✅ | 🟡 | 🟡 |
| Compress / optimize prompt | ✅ 3 methods | ❌ | 🟡 DSPy | ❌ |

## The one-paragraph verdict
Braintrust is clean and summary-first but hides its nav and has ephemeral-run footguns.
Arize AX and Phoenix are complete and enterprise-grade but dense, raw-data-first, and cryptic
at import. **Keystone matches their power while staying legible (the loop framing), calm at
the edges (no silent failures), governed (versions/tags/labels/provenance/origin), and — the
headline — genuinely usable from row one to row ten-thousand.** Every ✅ above is clickable in
the live prototype today.
