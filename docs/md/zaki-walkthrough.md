# The Zaki walkthrough — every screen, one by one
*The presenter's script for the full product review. Order: **Dataset Management first, Prompt Management second**, then the rest of the product screen by screen. For each screen: what it is, the flow it serves, where it sits in the IA and why, and the decisions worth defending.*

**Before you start:** open the prototype, press `⌘K` → **Reset demo data** (clean slate), and keep this doc on a second screen. Everything is mock data — every claim below is clickable.

---

## 0 · The 90-second framing (say this before touching anything)

**The one-liner:** *"Every observability tool shows you what your AI did. Keystone is built around the loop that makes it better — and it treats cost as a design material, not a billing report."*

**The IA in one breath:** the left rail *is* the evaluation loop — **Build** (Prompts) → **Evaluate** (Datasets · Evaluators · Experiments) → **Observe** (Monitor · Traces · Logs · Review), with Overview and Connect above it and Manage below. The top bar repeats the loop as a **Build · Evaluate · Observe** compass, so on any screen you know where you are and can jump a stage in one click.

**Why stages, not a flat list (the IA decision):** we tested both competitors cold. Braintrust's flat 14-item nav scored 2/5 on wayfinding in our audit — users can't tell where they are in the loop. Arize organizes by data type, which suits engineers only. Stages make the mental model legible to the non-engineer half of the team — the underserved market.

**One more rule to state up front:** cost in Keystone is always **token-spend, split task vs judge — never a fabricated dollar figure**. You'll see it on every screen; it's a principle, not a widget.

---

# Part 1 · Dataset Management (start here)

## 1.1 The Datasets list — `#/datasets`

**What it is:** the library of test data — every dataset with version chips, movable tags (`golden → v3`), curated badges, labels, and a KPI strip.

**Show:** scan the columns → open the **Display** menu (column choices persist) → select two rows and show the **bulk bar** (Tag applies a real label · Export copies real JSON · Archive works) → try the type-name-to-confirm delete and cancel it.

**Why it's designed this way:**
- *A dataset is governed data, not a CSV in a bucket.* Three concepts deliberately kept separate: **labels** (free-form classifiers on the dataset), **tags** (movable pointers from a name to a version — promoting `golden` is a deliberate act), **row tags** (per-record slice strings). Competitors blur these.
- *Every action is reversible or confirmed* — archive hides, delete needs the name typed.

## 1.2 Creating a dataset — New dataset ▾ → Upload CSV

**Show:** the name field with the **live slug** → drop the sample CSV (or click "…or use the sample GST FAQ data") → **the mapper**: one plain row per column — the column name, a real first-row sample value, and an "Import as" dropdown with plain-English definitions, pre-filled by auto-detection → point at the ✓ readback line ("Input: question · Expected: official_answer · Metadata: topic · ID: id") → the upsert-key note → the 3-row preview → "Import 8 rows".

**Why (this screen was redesigned twice, from evidence):**
- *Plain words beat schema paths.* Arize asks you to map `attributes.output.value` on first contact. We show your own data and ask "which column is the correct answer?"
- *Dropdowns beat drag.* The first version used drag-to-bucket; live review showed it was hard to parse and dragging could dismiss the dialog. Now it reads top-to-bottom, works with a keyboard, and the dialog ignores stray outside clicks once a file is loaded — your mapping can't be thrown away by accident.
- *`category` auto-sorts into Metadata* — both competitors dump descriptor columns into Input and pollute the test case.
- *Warnings before commit, never after* — blank inputs, duplicate ids, missing ground truth are flagged calmly pre-import; the competitors mis-import silently.

## 1.3 Landing inside the new dataset

**Show:** you arrive **inside** the dataset — success banner with upsert math ("8 inserted · 0 updated"), rows in CSV order, v1 in history, and **Evaluate in ▾** on the header.

**Why:** our teardown found we originally dumped users back on the list (as did nobody good). Import → *inside the data* → next loop stage one click away. Also: re-importing rows with a matching ID **updates in place** — the upsert contract means no silent duplicates, ever.

## 1.4 Dataset detail — `#/datasets/gst-faq` (Items · Versions · Experiments · Manage)

**Show, in order:**
- **Items:** 10,000 real rows — instant search, structured filters, Sample 100, column toggles, select-all-across-pages with an explicit escalation step. Edit a row → the **working-draft banner** appears → Save as v4 or Discard. Point at the **provenance mix strip** at the bottom (imported / human / captured / synthetic %).
- **Versions:** immutable history; **restore never rewrites** — it opens an old version as a draft you re-commit; the **tag dialog** moves `golden` forward deliberately; the diff dialog.
- **Experiments:** every run that used this data, with the **pinned version** it ran against.
- **Manage:** lifecycle (draft → active → archived), labels, danger zone.

**Why:**
- *Versioning that can't lie* — every change is vN+1 with author + note; experiments pin the version they ran, so any past result is reproducible forever. Braintrust has snapshots, Arize has timestamps; neither has movable tags + pinned experiments + per-row provenance + an upsert contract — the four things that make a golden set auditable.
- *Provenance per row* — a captured row keeps a link to the trace it came from; synthetic rows are visible as synthetic until a human promotes them. Coverage you can trust.
- *10,000 rows on purpose* — the scale proves the grid design (paging, sampling, select-all escalation) instead of hand-waving it.

---

# Part 2 · Prompt Management

## 2.1 The prompt library — `#/prompts`

**What it is:** prompts as **deployables** — version chips, environment badges (production/staging/dev/draft), labels, last-run status, run counts.

**Show:** the shared toolbar (search · structured filters · saved views · sort) → a row's ⋯ menu (Pin to sidebar updates the rail live · Duplicate · Export · type-to-confirm Delete).

**Why:** `GST QnA v13 · production` is a *deployment statement*, not a filename. "What's live?" is answered from the shelf. Braintrust versions prompts but has no environments; Arize has a hub but promotion isn't a verb.

## 2.2 Edit — `#/prompts/gst-qna`

**Show:** chat segments; type `{{something}}` and watch **Inputs derive live**; `@[references]` for shared blocks; segment ⋯ menu; undo/redo; the **token budget** card ("~1,547 / 200,000 tokens per run — Compress can trim the filler").

**Why:** variables are part of the contract — declared, shown, protected everywhere downstream (the playground fills them from datasets; Compress provably never strips them). And the budget speaks tokens, not invented dollars.

## 2.3 Run (the playground) — `#/prompts/gst-qna/run` · Single | Batch | Compare

**Show:**
- **Batch:** the input-set builder (Load from dataset · Paste rows · Add row) → attach **inline scorers** (the same evaluators as Experiments) → run → per-cell score pills + the aggregate strip (rows · avg tokens · avg latency · **total tokens** · avg scores) → click a row → the **Inspect** dialog (assembled prompt, request/response, timeline, comment thread).
- **Compare (A/B/n):** v12 vs v13 side by side → **Add variant** → pull in v11 → per-variant aggregate cards with the **best highlighted** → genuinely different outputs, diff-highlighted → **Promote winner** jumps into Manage.

**Why:**
- *Scorers live in the playground* — you shouldn't need a formal experiment to know if an output is good. Same judges, same thresholds, everywhere.
- *Compare ends in a promotion* — comparing was never the goal; shipping the better version was. Braintrust compares in a playground and stops there.
- *Everything is capture-able* — any good row can be sent to a dataset with an origin backlink (the loop feeding itself).

## 2.4 Compress — `#/prompts/gst-qna/compress` ⭐ the differentiator

**Show:** three LLMLingua methods with one-line tradeoffs → keep-rate ⇄ target tokens → run it → struck-through filler, **green protected `{{variables}}`** → the tiles: Before / After / Saved % / **Saved per 1k runs in tokens** → Apply as v14 (v13 intact).

**Why:** this is the "cost coach" thesis as a *feature*. Neither competitor has anything in the category. And it respects the lifecycle — compression is just another reviewed, comparable, revertible version.

## 2.5 Manage — `#/prompts/gst-qna/manage`

**Show:** version history with per-version comment threads → **environments with a promotion gate** (the Promote button explains itself: "needs ≥80% on GST FAQ — v13 scored 83%") → associated datasets → members (roles, invite, remove — all working) → labels (add/remove inline) → Set as baseline (flips the real store) → DSPy optimize honestly labeled **Preview** → danger zone.

**Why:** *promotion is gated by evaluation* — you cannot promote an unevaluated prompt; the gate names the number it needs. That's governance neither competitor models. And where something isn't built (DSPy), the UI says "Preview" instead of pretending.

---

# Part 3 · The rest, screen by screen (loop order)

## 3.1 Connect — `#/connect` (and `?state=new`)

**What:** the instrumentation on-ramp — how traces get into Keystone.
**Show:** the connected state (live pulse, SDK snippets, write-only key) → then `?state=new`: the 3-step wizard → click **Simulate first trace** → the four "Unlocks with your first trace" cards light up.
**IA:** directly under Overview — it's the zeroth question of the whole product.
**Why:** the teardown rated Braintrust's unlock-onboarding the single best pattern we saw; ours states what each surface *will do* before data exists, so an empty product still teaches. One line of code, key never rendered.

## 3.2 Overview — `#/`

**What:** Home — a guided tour first, a triage board forever after.
**Show:** the 5-step tour grouped Build/Evaluate/Observe (each step opens a real screen) → skip it → **Home flips triage-first**: "Needs your attention" (failing row · uncaptured prod failure · pending reviews · a Compress cost nudge) leads, the tour collapses to a Replay card.
**Why:** never an empty wall (Arize greets you with a terminal). And Home *adapts*: teaching mode for week one, triage mode for every day after — each attention item is a real, linkable problem, and one of them is always the Cost Coach doing its job.

## 3.3 Evaluators — `#/evaluators` and `#/evaluators/qa-correctness`

**What:** reusable, versioned judges — an LLM criterion or a code assertion — attached to experiments and to live traffic.
**Show:** the list (kind · version · threshold · used-in) → open Q&A correctness: the criterion in plain words, judge model, scoring rubric, pass threshold → the **Live test** panel (walk real rows; code judges cost 0 judge tokens) → **Save changes** bumps the version for real → scroll to **Alignment**: agreement %, the judge-vs-human confusion matrix (the "judge passed / human says incorrect" cell is the dangerous one), linked disagreements.
**Why:**
- *One judge, every surface* — the same evaluator scores experiments, the playground, and live traffic; edit it once, everything picks it up.
- *Judge-the-judge* — human verdicts recorded anywhere in the product grade the evaluator here. Recurring disagreements = the criterion needs a sharper rubric. Neither competitor closes this loop.

## 3.4 Experiments — `#/experiments` and `#/experiments/7`

**What:** the scorecards — a prompt version × a pinned dataset version × evaluators.
**Show:** the list (pass-rate trend chart, **Convergence column with delta vs baseline**, shared filters/saved views, N-way compare dialog) → open **#7**: headline pass rate with the **plain-words verdict** ("Beats baseline #6 by 16 pts · n=6, directional"), token tiles (task vs judge), **Convergence 0.89** (hover for the definition), score distribution, per-metric tiles, **pass rate by topic → "Weakest topic: Rates — start debugging here"**, histogram with the lowest rows → Rows view → click the failing row.
**Why:**
- *Verdicts in words, honest about sample size* — "83%" alone invites over-confidence; "n=6, directional" is the honest read.
- *Convergence is the agent-native metric* — pass rate says whether the answer was right; convergence says whether the agent got there directly or wandered (steps + retries). Arize gestures at this; we made it a number.
- *Every headline derives from the rows* — the scorecard can't disagree with its own table.

## 3.5 Traces — `#/traces` and `#/traces/gst-003?exp=7`

**What:** the microscope — one row's full story.
**Show (list):** preset chips (All · Failing · Latest experiment · Token hogs) + the shared filter toolbar + tokens split per row + Load more.
**Show (detail — the crown jewel):** the verdict bar (Fail · 0.40 · task 700 + judge 480 tokens) with the two fix actions **right there** (Edit the evaluator / Edit the prompt) → **Tree | Timeline | Graph** — Graph shows input → task → two judges in parallel (the JSON check reads **"code · free"**) → verdict, with the **trajectory line**: "wandered — 5 steps (optimal 3), 2 retries · convergence 0.60" → the judge's plain-English reasoning, output vs expected → **Human review inline**: click Agree — it *persists* and feeds the evaluator's Alignment view → the `[` `]` pager and "next failing row".
**Why:**
- *A failing trace should end in a fix, not a screenshot* — both repair paths are on the verdict bar.
- *Cost is visible in the structure* — the graph shows where tokens go per node; code evaluators being "free" is an argument you can see.
- *Calibration happens where the disagreement happens* — not in a separate labeling product.

## 3.6 Logs — `#/logs`

**What:** live production traffic; the sampled slice gets online scores.
**Show:** the monitors strip (one breached) → the explainer ("online, not offline" — linking to where judges are configured) → filter to fails → the 0.35 row → ⋯ → **Add to dataset**.
**Why:** *a production failure becomes a regression test in two clicks* — the loop restarting is the whole point of this screen. Every request is logged; only the sample is scored (cost discipline again).

## 3.7 Monitor — `#/monitor` · Dashboard | Online evals

**Show (Dashboard):** the KPI row (Requests · Pass rate · Latency · **Token spend**) → switch the time range — *the data actually changes* → **Add tile ▾** (Tokens per answer · Judge share — one-click, removable, persisted) → alerts with Healthy/Approaching states → **New alert** opens a real rule builder.
**Show (Online evals):** the judge table (scope · sample 5% · on/off · pass-rate trend) → **+ Add online evaluator** — pick a judge, a scope, a sample rate, done.
**Why:**
- *The dashboard presets are the cost story* — Arize builds widgets from a query language; ours are one click and token-led.
- *Online evals live in Monitor, in plain words* — "the same judge you trust in testing, watching production." Arize has the capability buried in jargon.

## 3.8 Review — `#/review`

**What:** the human-in-the-loop queue — flagged production spans, one at a time.
**Show:** the span card (input · output · auto score) → verdict buttons + label + the **0–1 score slider** + **assignee** → note → **Resolve** — the queue count *drops and stays dropped* → "Resolve & capture to dataset" for the failure → the "Queue clear" state.
**Why:** three score shapes (categorical · slider · free-text) because different questions need different fidelity — and every verdict feeds the Alignment view. Braintrust's Review was a documented dead end in our teardown; ours resolves *into* the loop (dataset capture, judge calibration).

## 3.9 Inbox — `#/inbox` (account menu)

**What:** the personal cross-project stream. Two kinds of "done": actionable items clear only by acting; informational items clear by reading.
**Show:** act on the approval request — the item resolves and **the account-menu badge count drops** (it's derived, never hardcoded).
**Why:** approvals, invites, expiring credentials — the things that block other people — get a surface where reading isn't mistaken for acting.

## 3.10 Settings / Manage — `#/settings`

**What:** deliberately "lite": profile · org members with working role selects · **usage by project in token-spend (task vs judge)** · an honest roadmap panel (Billing · Audit log · SSO/SAML + SCIM).
**Why (say this straight):** *a half-built enterprise plane reads worse than a clear roadmap.* What the demo needs is real; what isn't built says "Roadmap" on a card. Usage speaking tokens closes the loop on the cost principle — even admin reporting uses the same honest unit. Operate and Learn stay "Soon" in the nav: staged scope, specced not skipped.

## 3.11 The shell (point these out as you go, not as a separate stop)

- **⌘K palette** — real fuzzy search over every entity, context verbs on the open prompt, create actions, theme, and the hidden **Reset demo data**.
- **Ask Keystone** — the copilot that *routes*: every answer ends in a deep link to the right stage ("Alyx and Loop chat about your data; ours takes you there").
- **Breadcrumb + sibling switcher** on detail pages; **g-chords** (`g d`, `g p`, `g c`…); collapsible rail; Light/Dark/System; skip-link and live-region a11y; fully responsive.
- **The Build · Evaluate · Observe compass** in the top bar — the IA argument made permanent.

---

## 4 · The decisions register (if Zaki wants the "why" list in one place)

1. **Stage-based IA over flat nav** — wayfinding scored 2/5 on Braintrust's flat list; the loop is the mental model.
2. **Tokens, never fabricated $** — task-vs-judge split is the *actionable* division: judge cost is the eval tax you tune; task cost is what Compress attacks. Arize shows "Cost —" until you configure pricing tables.
3. **Ship populated, teach by touring** — a wired sample project beats empty states + a manual; the tour retires itself into triage.
4. **Connect's unlock pattern** — the product explains its surfaces before data exists (adopted from the best thing our teardown found, made clearer).
5. **Datasets are governed entities** — labels ≠ tags ≠ row-tags; immutable versions; pinned experiments; upsert; provenance. Auditable golden sets.
6. **Prompts are deployables** — environments, gated promotion (evaluation as the gate), attributable results, revertible optimization.
7. **One judge everywhere** — experiments, playground, live traffic; edit once.
8. **Humans grade the judge** — verdicts persist → Alignment matrix → sharper rubrics. The calibration loop neither competitor closes.
9. **Fix actions live on the failure** — trace verdict bar → edit prompt / edit evaluator; Review → capture to dataset.
10. **Honesty as a design rule** — "n=6, directional", "Preview" labels, roadmap cards, "wandered — 2 retries". Trust is the product.

## 5 · Suggested session shape (90 min with Zaki)

1. **Framing + IA** (5 min) — section 0.
2. **Dataset Management** (20 min) — sections 1.1–1.4, upload live.
3. **Prompt Management** (20 min) — sections 2.1–2.5, end on Compress.
4. **The loop, screen by screen** (30 min) — sections 3.1–3.10 in order.
5. **Decisions register + cuts** (10 min) — section 4, plus what was deliberately not built and why.
6. **His calls** (5 min) — the four decisions to ratify from the Day 5 page.
