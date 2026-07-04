# Keystone — Design Review (lo-fi complete)

**Date:** 2026-07-04 · **Stage:** lo-fi wireframes done, verified, reconciled to the repo. Next is hi-fi.
**File:** `Keystone-Loop-Wireframes.pen` · **Full journal:** `build-log.md` · **Decisions:** `decisions/decision-log.md`
**Proof screenshots:** `decisions/assets/after-*.png`

> One-line: the LLM-evaluation **loop** — Dataset → Prompt → Evaluator → Experiment → Trace — designed as one
> coherent product that is summary-first, honest about the numbers it actually has, and usable at any width;
> demonstrably cleaner than Braintrust and Arize.

---

## 1. What we're building & who it's for

Keystone (for Zaki / Esberi) turns the eval loop into a tool that serves **two users at once**:
- **Adel** — customer-0, ex-Arize/Braintrust power user. Wants speed, keyboard, density-on-demand, trace depth.
- **A goal-clear first-timer** — knows they want "is my prompt good enough?", not the jargon.

The design is grounded in the **actual `esberi/keystone` repo** (cloned read-only), not a greenfield guess — so
every screen binds to real entities and the shipped `ks` CLI has parity. Where the shipped app has UX gaps, we
**designed the better product** and flagged anything that needs backend work (⚑) rather than mirroring the gap.

---

## 2. What's designed (inventory)

| Area | Screens / boards | Status |
|---|---|---|
| **Prompt sub-app (PRD #1)** | List · Edit · Run (batch) · Compress · Manage · Dialogs · ⌘K palette | ✅ 7/7 built + **adversarially audited** + fixed |
| **The loop** | Overview · Dataset · Evaluator · Experiment · Trace | ✅ 5/5 realigned to decisions |
| **Edge cases** | `Board-States`: empty · loading · running · error (×6) | ✅ built |
| **Responsive** | `Board-Responsive`: breakpoint rules + mobile List + mobile Edit | ✅ built |
| Shell | Loop-staged rail (Build / Evaluate / Observe), header, ⌘K, loop ribbon | ✅ consistent across all screens |

Everything is on one corrected shell, **amber-free**, **single-brace `{variable}`**, real-field-grounded, and
carries the **GST QnA / AskJolly** worked example end-to-end.

---

## 3. The decisions that shaped it (and why)

These are the calls that make Keystone *Keystone* — all logged in `decision-log.md` with before/after proof.

1. **Token-spend, never dollars.** The repo has **no cost concept** — only token counts. So we retired every
   `$`-cost tile/column/chip (they were fabricated numbers) and lead with **token-spend** + latency. Honest, and
   it removed a class of wrong-number/credibility bugs. *(The one place cost belongs — org budgets — is a flagged
   backend plan, not invented in the UI.)*
2. **Single-brace `{variable}`.** Matches the shipped parser `/\{(\w+)\}/` and the app's own example. `{{mustache}}`
   would silently break at runtime. Small detail, real correctness.
3. **Dark leads, light co-equal — but in hi-fi.** Decided dark-first; *not* built in lo-fi because Pencil's
   renderer can't resolve themed colours. It lands in the shadcn/code step where theming is real.
4. **shadcn New York + neutral slate.** Read from the repo's own components (button geometry, lucide, CHANGELOG
   removing the indigo). Monochrome slate with green/red only for pass/fail — calm, not carnival.
5. **IA = the loop itself.** Rail groups are **Build · Evaluate · Observe** (the loop stages), not the shipped
   Build/Operate/Project. Simpler than competitors, and it teaches the loop by navigating it.
6. **Evaluator is first-class in the UI (⚑ backend).** Today an evaluator is just a metric-string on a run; we
   designed it as a real, testable object and flagged the backend binding — because "how is my output judged" is
   the heart of the loop and deserves a home.
7. **Summary-first, flat-over-drawers.** Every results view opens on the scorecard, not a raw table; overlays are
   centered dialogs, not the shipped side-drawers. (Zaki's #1 principle.)

---

## 4. How it beats Braintrust & Arize

From the hands-on teardown (`keystone-competitor-review/`). Verdicts are scoped — no screen claims more than it shows.

| Dimension | Braintrust | Arize | **Keystone** |
|---|---|---|---|
| First-run / empty states | bare | bare | **Guided empty states** with one clear CTA each |
| Results default | raw-table-ish | dense | **Summary-first** scorecard, drill on click |
| Experiment → Trace link | good | buried | **One-click** row → span tree → evaluator reasoning |
| Running a batch | wait for whole run | wait | **Rows stream in as they finish** |
| Errors | often silent/blank | opaque | **`n/a` + reason (429)** + retry-only-failures |
| Command palette | generic ⌘K | generic ⌘K | **Context-aware** "On GST QnA" — 10 on-prompt verbs |
| Responsive | unusable < ~1000px | unusable | **Rail collapses, panes stack, tables → cards** |
| Vocabulary | jargon | jargon | Plain, consistent lexicon (Run/Promote/Compress/Score) |

---

## 5. Edge cases & responsive — the differentiators

- **`Board-States`** — the states competitors under-design, each mapping to a shadcn Empty / Skeleton / Progress /
  Alert: empty (List/Experiments/Dataset with CTAs), loading (skeleton, not spinner-on-blank), running (streaming
  progress + Cancel), error (banner + errored row showing `n/a` + reason + retry-errored).
- **`Board-Responsive`** — breakpoint rules + two live mobile frames (List **table→cards**, Edit
  **two-pane→stack** with collapsible inspector). ⌘K, the loop ribbon, and summary-first are invariant at every
  width; nothing is desktop-only.

---

## 6. Verified, not asserted

The lo-fi wasn't just drawn — it was **attacked**:
- **Prompt sub-app audit** (8-way, adversarial): 1 blocker + 7 majors — all confirmed against screenshots and
  fixed (name inconsistency, stale version, fake `$`-cost, missing Capture, WCAG colour-only status, redundant
  Sort, seam wording).
- **Two audits that crashed were re-run and closed** — Palette (context group was showing 3 of 10 verbs → fixed
  to the full set) and the Delete dialog (conformant; two overstated claims corrected honestly).
- **Stale-backdrop class of bug** caught in re-verify: overlay backdrops were pre-fix *copies* re-exposing retired
  issues; grep proved the real screens were clean, and both backdrops were re-pointed to canon.
- Method throughout: `export_html → grep for regressions → screenshot → read`. Every fix has a refreshed proof PNG.

---

## 7. What carries into hi-fi (shadcn New York → Claude Code `prompt-management-v1`)

- **Dark-leads theme** (co-equal light) — the correct place for it, with real CSS variables.
- **Component-level minors** that are trivial with real shadcn and were consciously deferred: Edit header
  env-badge/pin/version `Select`; reference chip `@version/@tag`; Manage owner-static + member-remove +
  promote-disabled-on-production; Run Tokens-before-Latency + manual "Add row"; two-co-primary hierarchy tuning.
- **Backend-flagged (⚑) items** to raise with Zaki before/at build: Evaluator as a real object; insert-reference
  version/tag resolution; prompt compression (LLMLingua) as a fast-follow.
- Full responsive breakpoints (real CSS, not representative frames).

---

## 8. Open questions to confirm (Zaki / Adel)

1. **Evaluator object** — OK to design it first-class in v1 UI binding to today's metric model, with the entity as
   a flagged backend fast-follow?
2. **Compression** — in the v1 cut (UI now, LLMLingua backend later), or defer entirely?
3. **Org cost/budgets** — the only legitimate `$` surface. In scope for v1, or later? (Keeps the "no fake cost"
   rule intact.)
4. **Sample project** ("Load a sample project" on Overview) — do we ship a canned GST demo dataset+prompt for
   first-run?

---

### Appendix — proof index (`decisions/assets/`)
`after-prompt-{list,edit,run,compress,manage,dialogs,palette}.png` ·
`after-loop-{overview,evaluator,experiment,trace}.png` ·
`after-states-board.png` · `after-responsive-board.png`
