# Competitive teardown — Arize · Braintrust · Keystone

*Live, logged-in walkthroughs of Arize AX and Braintrust, run side-by-side with the Keystone prototype — July 10, 2026. The full screen-by-screen audit (every finding, every screenshot reference) is in the companion doc “Live walkthrough log.”*

## The shared mental model

All three products are the same machine — the evaluation loop: **Observe (logs/traces) → Curate (datasets) → Evaluate (evaluators + experiments) → Improve (prompts) → Monitor → repeat.** They differ in *who can operate it*.

## Scorecard — 10 usability heuristics, scored live

| Heuristic | Arize AX | Braintrust | Keystone |
|---|---|---|---|
| Time-to-first-value | 3 | 3 | **5** |
| Wayfinding — where am I in the loop? | 4 | 2 | **5** |
| First-run & empty states | 4 | 3 | **5** |
| Cognitive load & jargon | 2 | 4 | **4** |
| Discoverability of the loop | 3 | 3 | **5** |
| Consistency | 3 | 4 | **4** |
| Feedback & recoverability | 3 | 4 | **5** |
| Non-engineer accessibility | 2 | 4 | **5** |
| Flow friction across loop hops | 3 | 4 | **4** |
| Polish & accessibility | 4 | 4 | **5** |
| **Total /50** | **31** | **35** | **47** |

*Keystone is a prototype scored on design intent; treat totals as directional. Arize was scored across two accounts (a fresh free account and a populated workspace); Braintrust on a fresh starter org.*

## What each product is

- **Arize AX** — an engineer's console. Deepest tracing (the Agent Graph is the single best agent-native view on the market), loop-staged navigation, the Alyx copilot everywhere. But: jargon at every first contact (`attributes.output.value` on your first CSV), dev-terminal onboarding, cost dead until you configure pricing tables, and real interaction bugs we hit live.
- **Braintrust** — the friendliest. Plain language, the best CSV column-mapper, an A/B/n playground, “unlocks with your first trace” onboarding. But: a flat 14-item nav with zero wayfinding, a dead-end Review screen, a marketing popup owning first-run, and metering by *score* that surprises teams.
- **Keystone** — the loop **is** the navigation (Build → Evaluate → Observe), a guided tour on live sample data from minute one, plain language throughout, and token/cost visible on every surface.

## Verified live on the competitors (receipts, not vibes)

- **Arize:** the default 15-minute time window rendered a 355-trace demo project as “No Data” — while the KPI strip simultaneously said “1 trace.” A nav flyout left an invisible overlay that swallowed every click until Escape. The floating copilot button blocked the Create-Dataset button in the import dialog. Tracing shows “Cost —” until you configure cost tables (yet the playground shows $ with zero config).
- **Braintrust:** Review’s empty state is one line of text pointing into Settings — no button, no preview. A GLM-5.2 promo card sat over the nav on every screen of a brand-new account. Datasets are never named during import (you get “Test Dataset Gst Qa” from the filename). Imported rows lose their CSV order (Arize too).

## The parity test — the same 10-row CSV, uploaded to all three

| | Arize | Braintrust | Keystone |
|---|---|---|---|
| Steps | 6 | 4 | 4 |
| Names the dataset | required field | ✗ (filename) | pre-filled + live slug |
| Column mapping | one “reference” dropdown, span-attribute jargon | semantic drag buckets, auto-inference | semantic buckets **with plain-English definitions + keyboard support** |
| Data-quality warnings before commit | ✗ | ✗ | ✓ (blank inputs · duplicate ids · missing ground truth) |
| After import | → detail ✓ | → detail ✓ | → detail ✓ **with upsert math (“10 inserted · 0 updated”) and rows in CSV order** |

## Keystone's three moats (all clickable in the prototype)

1. **The loop as navigation** — Build → Evaluate → Observe stages, a guided 5-step tour on live sample data, and a copilot that *routes* you to the right stage instead of just chatting.
2. **Cost as design material** — task-vs-judge token split on every row, verdict bar and experiment; code evaluators shown as “free” in the trace graph; **Compress** (three LLMLingua methods, keep-rate control, protected variables) turns prompt cost into a one-click saving applied as a new version. Neither competitor has anything in this category.
3. **Judge calibration inline** — Agree / “Judge is wrong” lives on the trace itself, and Review resolves straight into datasets — not a separate labeling product.

## What we adopted from the teardown (shipped same-day in the prototype)

- **From the findings on ourselves:** import now lands you inside the new dataset (upsert banner, CSV order kept); descriptor columns (category/topic/notes…) auto-map to Metadata; path deep-links resolve correctly.
- **From Arize:** a **Graph** tab on trace detail — the run's topology (input → task → parallel judges → verdict) with token cost on every node; saved filter views on Traces.
- **From Braintrust:** “Evaluate in ▾” on every dataset (experiment or playground, one click from the data); plain-words verdicts on experiments (“Beats baseline #6 by 16 pts · n=6, directional”); composable Monitor tiles — with cost-led presets neither competitor ships.

## Enterprise & pricing posture (for the roadmap conversation)

Arize cliffs from $50/mo to ~$60k/yr with SSO/audit gated to enterprise. Braintrust meters by *score* (several scorers per output silently multiply cost — a documented trust complaint) and gates SSO/RBAC/self-host behind sales. Keystone's stance: per-trace, predictable pricing, with enterprise controls pushed further down-tier.
