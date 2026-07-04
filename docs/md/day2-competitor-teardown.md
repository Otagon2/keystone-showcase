# Braintrust vs Arize AX — Synthesis for Keystone

**What this is.** A hands-on, side-by-side teardown of the two reference eval tools, run **live** by driving each real web app with Playwright and executing Zaki's exact loop — **Dataset → Prompt → Evaluator → Experiment → Traces** — on the **same GST FAQ dataset** (12-row sample from Adel's benchmark), using **Claude models only**. Every claim below is backed by a screenshot in `braintrust/` or `arize-phoenix/` and an entry in the per-tool interaction maps.

**How to read it.** Each section gives the **observation** (what actually happened on screen) and then **what I make of it** (the interpretation that should drive a Keystone decision). Amaan asked specifically for the second layer, so it's called out as **→ Read:** throughout.

**What we actually built in each tool (proof the loop ran):**
- **Braintrust** (org Esberi / project AskJolly): dataset `GST FAQ Sample (12)` + prompt `GST QnA (UX Review)` (Claude Opus 4.8) + scorer `Factuality` (Claude Haiku 4.5 judge) → experiment **`GST QnA (UX Review)` = Factuality 55%**, traces drilled to the judge's `select_choice` call.
- **Arize AX** (space AskJolly): dataset `GST FAQ Sample (12)` + playground prompt (Claude Opus 4.8) + evaluator `Accurate Q&A Judge` (Claude Sonnet 5 judge) → experiment **Avg Accurate Q&A Judge = 0.8**, traces via per-row "View trace".

---

## 1. Side-by-side GST flow — how each step differs

| Loop step | Braintrust | Arize AX | → Read (what it means for Keystone) |
|---|---|---|---|
| **Dataset import** | `+Dataset` (name) → empty state → upload CSV → **labeled buckets** (Input/Expected/Metadata/Tags/ID/skip) with **live preview** + **drag** to reassign; smart toggles (flatten/auto-parse). Auto-detect nailed only `id`. `06`,`07` | `New Dataset` modal (Upload CSV / Skill / Code) → upload → **one mapping: `reference`**, which **defaults to `attributes.output.value`** (OTel jargon); no input/metadata bucketing; all columns stored, referenced by name. `03`–`05` | Braintrust's **explicit template + preview + drag** is why it feels "2-min"; Arize's single cryptic-defaulted field feels "confusing/10-min." **Keystone: ship the labeled template, live preview, and default mappings to the *user's* column names.** |
| **Prompt** | Full-page editor; **rich-text** messages; `{{mustache}}`; model (Claude Opus 4.8) + Params; Text/JSON output; live "Chat with your prompt" tester; versioned. `11` | Playground: **ACE code editor**; `{single-brace}`; model (Claude Opus 4.8) + Params; "Load a prompt" from Hub; bottom dataset picker + Input Variables table. `09`–`11` | Both cover Zaki's prompt needs. Braintrust reads more **designer/PM-friendly** (rich text + inline test); Arize reads more **engineer-first** (code editor). **Keystone: rich-text blocks with an inline tester; mustache; text/JSON output.** |
| **Evaluator** | Top-level **Scorers**; templates (Factuality, Q&A, Summary, ExactMatch, Security, Possible); LLM-judge/TS/Python in one editor; **standard vars `{{input}}/{{expected}}/{{output}}`** (no remapping); choice→score table + CoT + threshold. `12`,`13` | Top-level **Evaluators** + inline gallery; **far more templates** (Hallucination, Q&A, Toxicity, Summarization, SQL, Tool-Calling, RAG…); **per-variable mapping required** (input/question/output → columns each time). `13`–`15` | Both **decouple evaluators from prompts** — validates Zaki's core IA ask. Arize has **more defaults**; Braintrust has **less setup friction** (standard vars). **Keystone: decoupled Evaluators view + Arize-sized default library + Braintrust-style standard variables (no remapping).** |
| **Experiment** | Playground run (⌘↵), per-cell latency/tokens/cost/👍👎; ephemeral until **"+Experiment"** snapshot; or dataset **"Evaluate in → Experiment"**. `14`–`21` | Playground **Run auto-creates an experiment**; Output header shows avg latency/tokens/cost + avg score. `12`,`16`,`17` | Arize's **run≡experiment** is lower-friction; Braintrust's **ephemeral-then-snapshot** is a footgun (you can lose a run). **Keystone: persist runs by default; make "promote/compare" explicit, not "save or lose it."** |
| **Results** | **Experiments list = scorecard** (score % · tokens · duration · cost + trend chart); experiment detail has a **score-distribution histogram**. Summary-first. `26`,`27` | **Compare-Experiments grid**, raw-output-first (question/reference/output + eval badge); aggregates in the **column header**; Charting tab. `18`–`21` | Braintrust **lands you on the summary** (Zaki's #1 principle); Arize lands you in the **raw grid**. **Keystone: summary-first scorecard by default, raw rows one click down.** |
| **Traces** | Row-click → **trace drawer**; span tree `eval → task → model` **and** `eval → scorer → judge`; judge span shows the `select_choice` reasoning. Score is fully traceable. `28`–`30` | Per-cell **"View trace"** → Trace Tree/Timeline/**Agent Graph**; LLM span with status/latency/tokens/cost; **but the experiment eval score is NOT on the trace** (Evaluations tab empty). `22`,`23` | Braintrust's **eval↔trace join** is the standout Zaki praised — every score traces to the exact call + reasoning. Arize links to the *task* trace but **not the score's reasoning**. **Keystone: nest the evaluator span inside the run trace, Braintrust-style; borrow Arize's Timeline/Agent-Graph for multi-step agents.** |

---

## 2. Consolidated interaction inventory — copy / adapt / avoid

| Pattern | Seen in | Verdict for Keystone |
|---|---|---|
| **Labeled dataset buckets + live preview + drag** | Braintrust import | **Copy.** Best-in-class; the reason import feels instant. |
| **Standard evaluator variables** (`input/expected/output`) | Braintrust scorers | **Copy.** Eliminates per-eval remapping (Zaki #4). |
| **Nested eval span in the run trace** (score→reasoning) | Braintrust traces | **Copy.** The trace-linking Zaki wants. |
| **Summary-first experiments list + score histogram** | Braintrust results | **Copy.** Directly serves Zaki's summary-first principle. |
| **Staged nav (Observe/Evaluate/Improve)** | Arize rail | **Adapt.** Matches Zaki's stages; keep it *persistent* (not Braintrust's hidden overlay) but lighter than Arize. |
| **Large prebuilt evaluator library** | Arize gallery | **Copy the breadth** (Q&A, Summary, Tone, Hallucination, Toxicity, Tool-Calling…). |
| **Run ≡ create experiment (auto-persist)** | Arize playground | **Adapt.** Persist by default; avoid Braintrust's "ephemeral until snapshot." |
| **Per-row output + latency + tokens + eval badge** | Arize compare grid | **Copy** as the drill-down layer (under the summary). |
| **Timeline / Agent Graph trace views** | Arize traces | **Copy later** for multi-step agents/Flows. |
| **NOT_PARSABLE surfaced distinctly** | Arize eval badges | **Copy.** Honest about judge brittleness. |
| **Command palette (⌘K)** | Both | **Copy.** Both lean on it; pairs well with Zaki's context-aware ⌘K. |
| **Per-variable evaluator remapping** | Arize | **Avoid.** Taxing; use standard vars. |
| **Cryptic default mappings (OTel jargon)** | Arize import | **Avoid.** Never show `attributes.output.value` to a user by default. |
| **Ephemeral playground (lose run on re-run)** | Braintrust | **Avoid.** Persist or warn. |
| **Hidden overlay nav (re-open every time)** | Braintrust | **Avoid.** Contributes to "back-and-forth." |
| **Scorer version pinning without a nudge** | Braintrust | **Avoid / soften.** I lost an edit to this; surface "this run used vN." |
| **Silent evaluator failure ("–", no error)** | Braintrust | **Avoid.** Surface provider/judge failures loudly. |
| **Escape closes the whole modal** | Arize | **Avoid.** Scope Escape to the innermost layer. |
| **Floating AI button overlapping primary CTAs** | Arize | **Avoid.** Keep the assistant clear of action buttons. |
| **Sort-on-header AND toolbar filter/display** | Both | **Decide one.** Zaki's redundancy question — pick a single mental model. |

---

## 3. Strengths / weaknesses through Zaki's three lenses (frustrating / confusing / taxing)

**Braintrust**
- *Frustrating:* ephemeral playground runs; the **hidden overlay nav** (deliberate re-open each time); scorer **version pinning** cost me an edit.
- *Confusing:* **silent scorer failure** ("–"/"No score data") — I only found the real cause (wrong judge provider) by opening the trace; scorer choice-score→CoT→threshold is conceptually heavy first-run.
- *Taxing:* few clicks overall; the main tax is the setup *concepts*, not the click count. **→ Read: Braintrust's friction is conceptual, not navigational — clean surface, sharp edges underneath.**

**Arize AX**
- *Frustrating:* two genuine **bugs** (Escape nukes the dialog; Alyx button overlaps CTAs); trace's Evaluations tab empty (score not joined).
- *Confusing:* **cryptic default mappings** (OTel attribute names); examples table shows Example-ID+id, hiding question/answer; "Compare Experiments" for a single experiment.
- *Taxing:* **per-variable evaluator remapping**; **wide grids** needing horizontal scroll; more chrome/steps. **→ Read: Arize's friction is navigational/density-driven — powerful and complete, but it makes you work; this is exactly Zaki's "too dense / too much at once."**

**One-line verdict:** *Braintrust = clean and summary-first with the best trace-linking, but sharp conceptual edges and an over-hidden nav. Arize = complete, staged, and enterprise-grade with more defaults, but denser, buggier at the edges, and raw-data-first.* **Keystone should marry Braintrust's clarity + trace-linking with Arize's staged IA + evaluator breadth, and avoid both tools' edges.**

---

## 4. Accessibility notes (both)

- **Braintrust (light):** strong semantic roles (button/textbox/menu/dialog/tab/switch/slider/combobox); draggable import chips expose `role=button` + `aria-roledescription="draggable"` (keyboard-operable); modals trap focus. Watch: some low-contrast grey secondary text; a few icon-only buttons (verify `aria-label`).
- **Arize (dark):** good role coverage (tab/combobox/option/dialog/menuitem/checkbox/searchbox; column headers expose "Column options"). Dark-theme contrast generally adequate. **Real operability risks:** Escape-closes-dialog (keyboard hazard) and the floating-button overlap on primary CTAs.
- **→ Read for Keystone:** both clear the semantic-roles bar; the differentiator is **focus/overlay discipline** — scope Escape, never overlap CTAs, keep contrast on secondary text ≥ WCAG AA. These are cheap wins we can beat both on.

---

## 5. Best way to overcome Zaki's ask — the Keystone loop

**Thesis:** Keystone should be the **"summary-first eval loop with first-class trace-linking, on a staged rail, with zero-remapping evaluators."** Concretely, per loop stage:

1. **Dataset** — one **standard template** (id · input · expected/reference · metadata · tags), Braintrust-style **labeled buckets + live preview + drag**, with **smart defaults to the user's own column names** and a light heuristic (`answer/expected/output→Expected`, `id→ID`). Legible populated table by default (show input/expected, not hashes).
2. **Prompt** — rich-text system/user blocks, **`{{mustache}}`** variables, model+params (Anthropic/OpenAI), text/JSON output, inline tester; keep it **flat** (Flows handle multi-step, per Zaki). Design the **references-by-version/tag** flow cleanly (neither tool nailed it).
3. **Evaluator** — a **decoupled Evaluators view** (both tools validate this), a **big default library** (Q&A, Summary, Tone, Hallucination, Toxicity, Tool-Calling…), **standard variables so there's no remapping**, and a **simple choice preset** (correct/partial/incorrect → 1/.5/0) with CoT/threshold behind "advanced." Read the standard dataset vars automatically.
4. **Experiment** — **persist every run** (no ephemeral loss), score inline, and make **compare/diff** explicit. **Surface evaluator/provider failures loudly** (never a bare "–"); flag unparseable judge output (**NOT_PARSABLE**) honestly.
5. **Results → Traces** — **land on a summary scorecard** (pass %, model, params, avg cost, avg latency, tokens + a score-distribution histogram), raw rows one click down, and **every score drills into a trace where the evaluator runs as a nested span with its reasoning** (Braintrust's join) — plus Timeline/Agent-Graph for Flows later.

### Mapping to the decisions Zaki left to you

| Open decision | Evidence from the teardown | Recommendation |
|---|---|---|
| **Where do prompt-mgmt & evaluators sit (Build vs Evaluate)?** | Arize: Evaluators under **Evaluate**, Prompts/Playground under **Improve**, Datasets+Experiments combined. Braintrust: all flat. | **Stage the rail** (Build: Datasets, Prompts · Evaluate: Evaluators, Experiments · Observe: Traces/Logs). Keep **Evaluators decoupled** at the Evaluate stage. |
| **"Project" vs "workspace"?** | Braintrust: Org→**Project**. Arize: Org→**Space**→projects (heavier). | Zaki leans **workspace**; Arize's Space≈workspace supports that. Use **Workspace → Project**. |
| **Sort/filter on toolbar or column headers?** | Both do **both** (headers sort + toolbar filter/Display) — the redundancy Zaki flagged. | **Pick one:** sort+filter **on column headers**, toolbar only for **view/columns + saved views**. |
| **References by version/tag flow?** | Braintrust surfaces prompt *versions* but no clean "insert prompt by tag"; Arize "Load a prompt" from Hub. Neither is great. | **Design fresh:** an inline "insert reference → pick prompt → pick version/tag" picker with a visible resolved-version chip. |
| **Onboarding: BYOK vs internal-admin?** | Both are **BYOK** (Braintrust AI-providers; Arize provider integrations + Data Region/SAML). Provider-key gaps caused a **silent** Braintrust failure. | Copy **BYOK** onboarding, but **validate keys up front** and **surface missing-provider errors loudly** at run time. |

### Three things only *we* can beat both on
- **Join the score to the trace by default** (Braintrust does; Arize doesn't) — and make it the **primary** row interaction, not a secondary icon.
- **Summary-first everywhere** without hiding the raw rows — Arize buries the summary, Braintrust hides the nav; we can do summary-first *and* a persistent light rail.
- **No sharp edges:** persist runs, loud failures, scoped Escape, CTAs never overlapped, user-named column mappings. Each is a specific defect we watched cost time in these tools.

---

## Appendix — artifacts
- `braintrust/` — 30 screenshots (`00`–`30`), `braintrust-ux-report.md`, `braintrust-interaction-map.md`.
- `arize-phoenix/` — 24 screenshots (`00`–`23`), `arize-ux-report.md`, `arize-interaction-map.md`.
- `data/gst-faqs-clean.csv` (1,593 rows) + `data/gst-faqs-sample.csv` (12 rows).
- Live experiments remain in each account (Braintrust: *GST QnA (UX Review)*; Arize: *GST QnA UX Review*) for Zaki/Adel to inspect.
