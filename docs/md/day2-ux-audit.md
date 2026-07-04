# Day 2 — UX audit (5 independent reviewers)

Before pushing the prototype further we ran a **fresh-eyes audit**: five independent reviewers, each walking every screen and the whole **Dataset → Prompt → Evaluator → Experiment → Trace** loop through a different user lens, then critiquing our design decisions and flagging what we missed.

**The five lenses:** first-time user · power user · accessibility + visual design (WCAG / Practical UI) · ML eval-engineer vs Arize & Braintrust · information-architecture / flow.

Findings are deduped and ranked by **severity × how many lenses independently raised each** (more lenses = higher confidence).

---

## The six things that mattered most (multi-lens consensus)

1. **Identity didn’t propagate.** Every prompt opened as *GST QnA* content and every experiment rendered as *#7* regardless of the id (the breadcrumb said #5, the heading said #7). "New" reopened existing records. *(flagged by flow, eval, first-time, power)*
2. **The loop ribbon was the weakest decision — unanimous.** Its links were hard-wired to the sample, so off the golden path it teleported you to GST; and it duplicated + disagreed with the left rail. *(all 5)*
3. **"Reusable evaluator" was claimed but not shown** — no Evaluators list and no Datasets list; both rail items dropped you into a single record. *(first-time, power, eval, flow)*
4. **Results were too shallow to act on** — one evaluator → one pass-rate, no per-topic slicing (the metadata was imported then discarded), no histogram, the "partial" band was mathematically dead, only 6/12 rows reachable. *(eval, power)*
5. **The core loop wasn’t keyboard-operable + real contrast failures** — table rows (the primary nav) were mouse-only; opacity-dimmed micro-text and the light theme failed AA. *(a11y)*
6. **Dynamic states + forward hand-offs were missing in-flow** — running/error only lived in the States gallery; Dataset had no "use in an experiment" CTA and Evaluator’s "Add to experiment" was dead. *(flow, power, eval)*

---

## Priority findings

### P0 — critical / systemic
- **Identity propagation** — load prompts/experiments by id; stop "New" reopening the sample.
- **Loop ribbon** — its clicks must not contradict the current object.
- **Keyboard-operable list rows** (WCAG 2.1.1, Level A) — rows are the primary control.

### P1 — major
- Evaluators & Datasets need real **list/library** views.
- **Filter / sort / search** on Prompts, Experiments, and the experiment Rows tab.
- A real **⌘K** fuzzy index (it was a hardcoded stub).
- Bring **running / loading / error** states into the real flow (+ `aria-live`).
- Fix the **Trace**: dead Raw-JSON/Messages/Timeline tabs, no row pager, no back-to-experiment.
- Wire the two dead hand-offs (Dataset → experiment; Evaluator → "Add to experiment").
- **Scope "Open traces"** to its experiment.
- **Multi-scorer** experiments + a per-metric scorecard; **per-topic** breakdown.
- **Contrast + focus** pass (both themes); non-colour distribution bar.
- Onboarding that doesn’t contradict a populated app; teach the loop + jargon tooltips.
- Remove internal/competitor language from product copy.

### P2 — polish
Dataset size shown 6/8/12; the dead "partial" band; task-vs-judge token split; unify the two Compare UIs; keyboard shortcuts; bulk actions; hit-target sizes; inconsistent Delete; a real 404; Compress orphaned from the tab set.

### Roadmap (bigger primitives — scoped, then built at prototype level)
Online eval over **production logs** + monitors; **regression / CI gating** on promotion; **human review / annotation** + judge calibration; first-class **code/assertion scorers**; **export / SDK**; dataset splits & richer schemas; N-way compare / version-trend.

---

## Design-decision report card (our 7 originals)

| Decision | Verdict |
|---|---|
| Token-spend, never $ | Right default; "never" too absolute → offer $ as a toggle; split task vs judge tokens |
| Single-brace `{variable}` | Correct, self-teaching → add a JSON-brace escape |
| Evaluator = first-class object | Right bet, under-delivered → needs a list + version pinning |
| Summary-first results | Best decision → deepen it (slices, histogram) |
| Dark-leads dual theme | Fine → fix the opacity greys + light-theme contrast |
| Status = icon + colour | A real a11y win → extend it to the colour-only bars |
| Loop ribbon duplicating the rail | Weakest — unanimous → make it contextual or cut it |

Everything above was implemented in the same iteration — see **[Design decisions & why](docs.html#day2-design-decisions)**.
