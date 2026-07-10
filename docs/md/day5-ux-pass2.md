# UX Pass 2 — what we overlooked, and what shipped
*Day 5 · Jul 11, 2026 · follow-on to the competitive teardown (Arize 31 · Braintrust 35 · Keystone 47 of 50)*

## The question this pass answered
After beating both competitors on the scorecard, we turned the same audit lens on ourselves: **as a product, what did we overlook — any flow, UI, or information architecture?** A full audit of every screen in the prototype, a re-read of the teardown for every recommendation left open, plus a live walkthrough of the six screens the teardown never scored. The honest answer: **seven gaps** — all closed the same day, in eleven shipped changes.

## What was overlooked → what shipped

### 1 · There was no story for how data gets in
Braintrust's "one-line setup + things unlock as your first trace arrives" was the single best pattern of the whole teardown — and we had nothing but a dead "Save connection" button.
**→ New Connect page.** Connected state: SDK snippets (Python / TypeScript / curl), a write-only key, a live trace pulse. First-run state: a three-step wizard and a "Simulate first trace" moment where the locked Traces / Logs / Monitor / Online-evals cards light up and link. This is now the demo's 60-second opener.

### 2 · The loop silently ended at ship
Arize scores live traffic with online evaluators; our loop stopped at Review.
**→ Monitor › Online evals.** The same Q&A judge used in experiments now scores 5% of live production traffic — one toggle, plain words, add more judges from a picker. Also fixed while there: the time-range selector now actually changes the data (it only changed a label before), and "New alert" opens a real rule builder.

### 3 · We contradicted our own #1 cost principle
Design principle: token-spend, never a fabricated dollar figure. Yet the playground showed "$0.108 total cost" and a per-row $ column.
**→ Every dollar deleted.** Tokens everywhere, split task vs judge. Compress now reads "Saved 190k tokens per 1,000 runs" — a stronger, more honest line.

### 4 · Human verdicts vanished
Agreeing or disagreeing with the judge on a trace disappeared the moment you navigated away; Review's "Resolve" was just a toast.
**→ Verdicts persist**, Review really resolves (with per-person assignment and a fine-grained score slider), and the payoff is brand new: **the evaluator's Alignment view** — agreement rate, a judge-vs-human matrix that calls out where the judge is too lenient, and links to every disagreement. We don't just let humans review; we use the reviews to judge the judge.

### 5 · Three different filter systems across lists
**→ One shared toolbar** (structured filters + saveable views) on Experiments, Traces, and Logs — filtering now behaves identically everywhere.

### 6 · The agent-native metric was half-shipped
The trace Graph view existed; the convergence score didn't.
**→ Convergence** (how directly the agent reached its answer) on every experiment, in the experiments list with a delta vs baseline, and on the trace graph — the failing row visibly "wandered: 5 steps, 2 retries".

### 7 · Settings was 100% dead
Every account-menu item just did nothing; the badge count was hardcoded.
**→ Settings-lite:** members with working role selects, usage-by-project in token-spend, and an honest roadmap panel (billing · audit log · SSO/SCIM) instead of half-built enterprise UI. The badge is now computed from real inbox items.

## Plus: the import mapper, redesigned from live feedback
The drag-a-column-into-buckets board was hard to parse, and dragging could dismiss the whole dialog mid-mapping. **Redesigned:** every CSV column is one readable row — its name, a real sample value from your file, and a plain "Import as" dropdown — pre-filled by auto-detection with a ✓ readback line. And once a file is loaded, a stray click outside can't throw the mapping away.

## Also shipped
- **A/B/n compare** in the playground: up to three prompt versions side by side, per-version aggregate cards with the best highlighted, genuinely different outputs to diff, and a "Promote winner" button straight into the promotion flow.
- **Wayfinding:** a persistent Build · Evaluate · Observe strip in the top bar — where am I in the loop, one click to any stage.
- **Adaptive Home:** once the guided tour is walked or skipped, Overview flips to triage-first.
- **Dead-button sweep:** every control in the product now mutates something, navigates somewhere, or honestly says "Preview". Plus a hidden ⌘K "Reset demo data" for clean rehearsals.

## Deliberately cut (and why)
- **Full RBAC/audit/billing enforcement** — zero demo-path value; an honest roadmap panel reads better than half-built enterprise UI.
- **Natural-language log filters** — point-click + saved views cover the need; natural language is the copilot's job.
- **Operate / Learn planes** — staged in the nav, specced not skipped.

*Eleven commits, typecheck and production build clean, every change verified live in the browser (desktop + mobile).*
