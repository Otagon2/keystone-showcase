# The Zaki walkthrough — the whole prototype, end to end

*The presenter's script for the full product review. **This covers the entire prototype — every screen and every route, not only the ones touched in Pass 2.** Two of them get the deep, end-to-end treatment because that's Sunday's actual ask: **Dataset Management (Part 1)** and **Prompt Management (Part 2)** — every entry path, every state, every flow, the reasoning behind each decision, and how each loops back into the product. **Part 3 covers every remaining screen** at documentation depth — enough to walk it and defend it, there for completeness rather than as Sunday's focus.*

> **Sunday scope, stated plainly:** Zaki's review is about **Datasets and Prompts, end to end.** Spend your time in Parts 1 and 2 — they are written to be walked click-by-click with nothing skipped. Part 3 is the rest of the product, documented so it's on the record and answerable, not so it eats the session.

**Before you start:** open the prototype, press `⌘K` → **Reset demo data** (clean slate), and keep this doc on a second screen. Everything is mock data — every claim below is clickable.

---

## 0 · The 90-second framing (say this before touching anything)

**The one-liner:** *"Every observability tool shows you what your AI did. Keystone is built around the loop that makes it better — and it treats cost as a design material, not a billing report."*

**The IA in one breath:** the left rail *is* the evaluation loop — **Build** (Prompts) → **Evaluate** (Datasets · Evaluators · Experiments) → **Observe** (Monitor · Traces · Logs · Review), with Overview and Connect above it and Manage below. The top bar repeats the loop as a **Build · Evaluate · Observe** compass, so on any screen you know where you are and can jump a stage in one click.

**Why stages, not a flat list (the IA decision):** we tested both competitors cold. Braintrust's flat 14-item nav scored 2/5 on wayfinding in our audit — users can't tell where they are in the loop. Arize organizes by data type, which suits engineers only. Stages make the mental model legible to the non-engineer half of the team — the underserved market.

**Where Datasets and Prompts sit — and why they're the two we go deep on:** they are the two ends of the loop that a human actually authors. **Datasets** are the ground truth everything is measured against; **Prompts** are the deployable everything is measured *for*. Get those two right — governed, versioned, evaluated, revertible — and the rest of the loop is plumbing between them. That's exactly why Sunday is about these two.

**One rule to state up front:** cost in Keystone is always **token-spend, split task vs judge — never a fabricated dollar figure**. You'll see it on every screen; it's a principle, not a widget.

---

# Part 1 · Dataset Management — end to end

*Walk this as one continuous story: how a dataset is born, grows, gets governed, feeds the loop, and gets fed back. Four entry paths in, a full lifecycle in the middle, two loop-backs out.*

## 1.0 The three ways a dataset begins (show the menu first)

Open `#/datasets` and click **New dataset ▾** before anything else — the menu itself is an argument:
- **Upload CSV** — the common path (1.2).
- **From traces** — build a test set out of real production spans you've already seen fail (jumps to Traces to pick).
- **Empty dataset** — start blank and add rows by hand.

**Why show the menu:** a dataset isn't only "a file you uploaded." Two of the three origins are *inside the loop* — you make test data out of things that actually happened. State this: **the best datasets are grown from production, not authored in a vacuum.** We'll use Upload for the live demo, but the other two are the ones that matter long-term.

## 1.1 The Datasets list — `#/datasets`

**What it is:** the library of test data — every dataset with version chips, movable tags (`golden → v3`), curated badges, labels, and a KPI strip.

**Show, in order:**
- Scan the columns; open the **Display** menu (column choices persist across reloads).
- Select two rows → the **bulk bar** appears: **Tag** applies a real label · **Export** copies real JSON · **Archive** works.
- Try the **type-the-name-to-confirm delete** and cancel it — destructive actions cost a deliberate keystroke.
- Point at the **empty-state copy** (if you reset): "Start from a file, from traces, or empty" with buttons for each — the list teaches its own three on-ramps.

**Why it's designed this way:**
- *A dataset is governed data, not a CSV in a bucket.* Three concepts deliberately kept separate: **labels** (free-form classifiers on the dataset), **tags** (movable pointers from a name to a version — promoting `golden` is a deliberate act), **row tags** (per-record slice strings). Competitors blur these into one.
- *Every action is reversible or confirmed* — archive hides, delete needs the name typed.

## 1.2 Creating a dataset — the import flow, in full

**Flow:** New dataset ▾ → **Upload CSV**.

**Show, step by step:**
1. The **name field** with the **live slug** underneath (what the URL and API id will be — no surprises later).
2. Drop the sample CSV — or click **"…or use the sample GST FAQ data"** so it works with nothing on hand.
3. **The mapper** — one plain row per column: the column name, a real **first-row sample value**, and an **"Import as" dropdown** with plain-English definitions (Input · Expected answer · Metadata · ID · Ignore), **pre-filled by auto-detection**.
4. The **✓ readback line**: "Input: question · Expected: official_answer · Metadata: topic · ID: id" — read it out; it's the plain-English contract of what's about to happen.
5. The **upsert-key note** (which column decides insert-vs-update), the **3-row preview**, then **Import 8 rows**.

**Why (this screen was redesigned twice, from live evidence — flag that):**
- *Plain words beat schema paths.* Arize asks you to map `attributes.output.value` on first contact. We show your own data and ask "which column is the correct answer?"
- *Dropdowns beat drag.* The first version used drag-to-bucket; live review showed it was hard to parse and dragging could dismiss the dialog. It now reads top-to-bottom, works with a keyboard, and **ignores stray outside clicks once a file is loaded** — your mapping can't be thrown away by accident.
- *`category`/descriptor columns auto-sort into Metadata* — both competitors dump these into Input and pollute the test case.
- *Warnings before commit, never after* — blank inputs, duplicate ids, missing ground truth are flagged **calmly, pre-import**; the competitors mis-import silently and you find out later.

**The messy-import state (worth showing if you have time):** re-open the dialog with a file that has a blank cell or a dupe id — the warnings appear inline, non-blocking, in plain language. **Calm handling of bad data is a feature**; the first contact with a new tool is usually a messy CSV.

## 1.3 Landing *inside* the new dataset

**Flow:** Import → you arrive **inside** the dataset (not back on the list).

**Show:** the success banner with **upsert math** ("8 inserted · 0 updated"), rows in CSV order, **v1** in history, and **Evaluate in ▾** already on the header.

**Why:** our teardown found we originally dumped users back on the list (as most tools do). Import → *inside the data* → the next loop stage one click away. And re-importing rows with a matching ID **updates in place** — the upsert contract means **no silent duplicates, ever.**

## 1.4 Dataset detail — `#/datasets/gst-faq` — the four tabs, in order

This is the heart of Part 1. Walk all four tabs; each is a deliberate answer to "what does it mean to govern test data?"

**Items** — the data itself
- 10,000 real rows: instant search, structured filters, **Sample 100**, column toggles, **select-all-across-pages** with an explicit escalation step (you can't nuke 10k rows by reflex).
- **Add row** by hand (the empty/grow path) → appears as a working draft.
- **Edit a row** → the **working-draft banner** appears → **Save as v4** or **Discard**. Editing never silently mutates the live version.
- Point at the **provenance mix strip** at the bottom: imported / human / captured / synthetic %.

**Versions** — history that can't lie
- Immutable history; every change is **vN+1** with author + note.
- **Restore never rewrites** — it opens an old version as a *draft you re-commit*, so history is append-only.
- The **tag dialog** moves `golden` forward deliberately; the **diff dialog** shows what changed between two versions.

**Experiments** — the accountability trail
- Every run that used this data, each showing the **pinned version** it ran against — so any past result is reproducible forever.

**Manage** — lifecycle & governance
- Lifecycle (draft → active → archived), labels, danger zone.

**Why (the four-tab argument):**
- *Versioning that can't lie* — vN+1 + author + note + pinned experiments means any historical result can be reproduced. Braintrust has snapshots, Arize has timestamps; **neither has movable tags + pinned experiments + per-row provenance + an upsert contract** — the four things together are what make a golden set auditable.
- *Provenance per row* — a captured row keeps a link to the trace it came from; synthetic rows read as synthetic until a human promotes them. Coverage you can *trust*, not just count.
- *10,000 rows on purpose* — the scale proves the grid design (paging, sampling, select-all escalation) instead of hand-waving it.

## 1.5 The loop OUT — Evaluate in ▾ (dataset → experiment)

**Flow:** on the dataset header, **Evaluate in ▾** → run this dataset against a prompt version + evaluators → lands you in a new **Experiment** with the dataset version **pinned**.
Also: from the list, a row's ⋯ → **"Run in playground"** (`/prompts/gst-qna/run?dataset=…`) or **"New experiment"** (`?view=experiments&new=1`).

**Why:** the data's whole reason to exist is to *measure something*. The next loop stage is never more than one click from the data, and the experiment records exactly which version it judged — Part 1 hands off cleanly to the Experiments story in Part 3.

## 1.6 The loop BACK — how rows return to a dataset

**Flow (two capture paths, both real):**
- **From a trace:** a failing production span → ⋯ → **Add to dataset** (with an origin backlink to the trace).
- **From Review:** resolve a flagged span with **"Resolve & capture to dataset."**

**Why:** this closes Part 1's loop — production failures become regression tests, tagged with where they came from. **A dataset that only grows from CSVs goes stale; one that grows from real failures gets sharper.** That's the entire thesis of governed test data, and it's why "From traces" sits in the create menu at 1.0.

---

# Part 2 · Prompt Management — end to end

*Same treatment: how a prompt is authored, evaluated, made cheaper, and promoted to production — and how it, too, loops back. A prompt in Keystone is a **deployable with a version history and an environment**, not a text file.*

## 2.1 The prompt library — `#/prompts`

**What it is:** prompts as **deployables** — version chips, **environment badges** (production / staging / dev / draft), labels, last-run status, run counts.

**Show:**
- The shared toolbar (search · structured filters · saved views · sort).
- A row's ⋯ menu: **Pin to sidebar** (updates the rail live) · Duplicate · Export · **type-to-confirm Delete**.
- Read one row out loud as a sentence: **`GST QnA v13 · production`** is a *deployment statement*, not a filename.

**Why:** "What's actually live right now?" is answered from the shelf, at a glance. Braintrust versions prompts but has **no environments**; Arize has a hub but **promotion isn't a verb** there. Here it is.

## 2.2 Edit — `#/prompts/gst-qna` — authoring the contract

**Show:**
- Chat segments; type `{{something}}` and watch **Inputs derive live** (variables aren't declared in a side panel — they emerge from the text).
- `@[references]` pulls in shared blocks (a reusable system preamble lives in one place).
- Segment ⋯ menu; full **undo/redo**.
- The **token budget** card: "~1,547 / 200,000 tokens per run — Compress can trim the filler."

**Why:** variables are part of the **contract** — declared by use, shown live, and protected everywhere downstream (the playground fills them from datasets; Compress provably never strips them). And the budget speaks **tokens, not invented dollars** — the cost principle shows up the moment you start authoring.

## 2.3 Run — the playground — `#/prompts/gst-qna/run` · Single | Batch | Compare

This is the deepest screen in Part 2. Walk all three modes.

**Single** — one input, one output, fast iteration while you're still shaping the prompt.

**Batch** — evaluate against real data
- The **input-set builder**: Load from dataset · Paste rows · Add row (note the dataset link — this is where Part 1's data flows in).
- Attach **inline scorers** — *the same evaluators as Experiments*, not a lesser playground-only check.
- Run → per-cell **score pills** + the **aggregate strip** (rows · avg tokens · avg latency · **total tokens** · avg scores).
- Click a row → the **Inspect** dialog: assembled prompt, request/response, timeline, comment thread.

**Compare (A/B/n)** — the decision
- v12 vs v13 side by side → **Add variant** → pull in v11 → per-variant **aggregate cards with the best highlighted**.
- Genuinely different outputs, **diff-highlighted** (we fixed the earlier "identical outputs, different scores" credibility bug).
- **Promote winner** → jumps straight into Manage.

**Why:**
- *Scorers live in the playground* — you shouldn't need to open a formal experiment to know if an output is good. Same judges, same thresholds, everywhere.
- *Compare ends in a promotion* — comparing was never the goal; **shipping the better version** was. Braintrust compares in a playground and stops there.
- *Everything is capture-able* — any good row can be sent to a dataset with an origin backlink (Part 2 feeding Part 1).

## 2.4 Compress — `#/prompts/gst-qna/compress` ⭐ the differentiator

**Show:**
- Three **LLMLingua** methods, each with a one-line tradeoff.
- **Keep-rate ⇄ target-tokens** control (drive it from either end).
- Run it → **struck-through filler**, and **green-protected `{{variables}}`** (proof the compression can't break the contract).
- The tiles: **Before / After / Saved % / Saved per 1k runs (in tokens)**.
- **Apply as v14** — v13 stays intact.

**Why:** this is the **"cost coach" thesis as a feature** — neither competitor has anything in this category. And it respects the lifecycle: a compression is **just another reviewed, comparable, revertible version**, not a magic in-place rewrite. "38% fewer tokens" is a stronger sentence than "$0.003 saved" — and it's honest.

## 2.5 Manage — `#/prompts/gst-qna/manage` — governance & the promotion gate

**Show:**
- **Version history** with per-version comment threads.
- **Environments with a promotion gate**: the Promote button explains itself — *"needs ≥80% on GST FAQ — v13 scored 83%."* Try promoting a version that hasn't cleared the bar; it won't let you and it tells you the number.
- **Associated datasets** (which test sets this prompt is measured against).
- **Members** — roles, invite, remove — all working.
- **Labels** — add/remove inline. **Set as baseline** flips the real store.
- **DSPy optimize** — honestly labeled **Preview** (not faked).
- **Danger zone**.

**Why:** ***promotion is gated by evaluation*** — you cannot promote an unevaluated prompt, and the gate **names the number it needs.** That's governance neither competitor models. And where something isn't built (DSPy), the UI says **"Preview"** instead of pretending — honesty as a design rule.

## 2.6 The full Prompt lifecycle in one sentence (say this to close Part 2)

*Author it in Edit → prove it in Run → make it cheaper in Compress → promote it through environments in Manage, gated by a real score → and capture its best outputs back into a dataset.* **Draft to production to feedback, all versioned, all revertible, all measured.** That's the loop Sunday is about, seen from the Prompt end.

---

# Part 3 · The rest of the product, screen by screen (documentation depth)

*Everything else, in loop order. Deep enough to walk and defend if Zaki asks — but this is the "good to have on the record" half, not Sunday's focus. If time is short, summarize and offer the detail on request.*

## 3.1 Connect — `#/connect` (and `?state=new`)

**What:** the instrumentation on-ramp — how traces get into Keystone in the first place.
**Show:** the connected state (live pulse, SDK snippets, write-only key) → then `?state=new`: the 3-step wizard → click **Simulate first trace** → the four "Unlocks with your first trace" cards light up.
**IA:** directly under Overview — it's the zeroth question of the whole product.
**Why:** the teardown rated Braintrust's unlock-onboarding the single best pattern we saw; ours states what each surface *will do* before data exists, so an empty product still teaches. One line of code, key never rendered.

## 3.2 Overview — `#/`

**What:** Home — a guided tour first, a triage board forever after.
**Show:** the 5-step tour grouped Build/Evaluate/Observe (each step opens a real screen) → skip it → **Home flips triage-first**: "Needs your attention" (failing row · uncaptured prod failure · pending reviews · a Compress cost nudge) leads, the tour collapses to a Replay card.
**Why:** never an empty wall (Arize greets you with a terminal). Home *adapts*: teaching mode for week one, triage mode for every day after — each attention item is a real, linkable problem, and one is always the Cost Coach doing its job.

## 3.3 Evaluators — `#/evaluators` and `#/evaluators/qa-correctness`

**What:** reusable, versioned judges — an LLM criterion or a code assertion — attached to experiments and to live traffic.
**Show:** the list (kind · version · threshold · used-in) → open Q&A correctness: the criterion in plain words, judge model, scoring rubric, pass threshold → the **Live test** panel (walk real rows; code judges cost 0 judge tokens) → **Save changes** bumps the version for real → scroll to **Alignment**: agreement %, the judge-vs-human confusion matrix (the "judge passed / human says incorrect" cell is the dangerous one), linked disagreements.
**Why:** *one judge, every surface* — the same evaluator scores experiments, the playground, and live traffic; edit it once, everything picks it up. And *judge-the-judge* — human verdicts recorded anywhere in the product grade the evaluator here; recurring disagreement means the rubric needs sharpening. Neither competitor closes this loop.

## 3.4 Experiments — `#/experiments` and `#/experiments/7`

**What:** the scorecards — a prompt version × a pinned dataset version × evaluators. *(This is where Part 1's "Evaluate in" and Part 2's "run as experiment" both land.)*
**Show:** the list (pass-rate trend chart, **Convergence column with delta vs baseline**, shared filters/saved views, N-way compare dialog) → open **#7**: headline pass rate with the **plain-words verdict** ("Beats baseline #6 by 16 pts · n=6, directional"), token tiles (task vs judge), **Convergence 0.89**, score distribution, per-metric tiles, **pass rate by topic → "Weakest topic: Rates — start debugging here"**, histogram with the lowest rows → Rows view → click the failing row.
**Why:** *verdicts in words, honest about sample size*; *convergence is the agent-native metric* (did the agent get there directly or wander — steps + retries); *every headline derives from the rows*, so the scorecard can't disagree with its own table.

## 3.5 Traces — `#/traces` and `#/traces/gst-003?exp=7`

**What:** the microscope — one row's full story. *(One of the two capture-back sources for Part 1.)*
**Show (list):** preset chips (All · Failing · Latest experiment · Token hogs) + shared filter toolbar + tokens split per row + Load more.
**Show (detail — the crown jewel):** the verdict bar (Fail · 0.40 · task 700 + judge 480 tokens) with the two fix actions **right there** (Edit the evaluator / Edit the prompt) → **Tree | Timeline | Graph** — Graph shows input → task → two judges in parallel (the JSON check reads **"code · free"**) → verdict, with the **trajectory line** ("wandered — 5 steps, optimal 3, 2 retries · convergence 0.60") → the judge's plain-English reasoning, output vs expected → **Human review inline**: click Agree — it *persists* and feeds the evaluator's Alignment view → the `[` `]` pager and "next failing row" → and ⋯ → **Add to dataset**.
**Why:** *a failing trace should end in a fix, not a screenshot*; *cost is visible in the structure*; *calibration happens where the disagreement happens.*

## 3.6 Logs — `#/logs`

**What:** live production traffic; the sampled slice gets online scores.
**Show:** the monitors strip (one breached) → the "online, not offline" explainer (links to where judges are configured) → filter to fails → the 0.35 row → ⋯ → **Add to dataset**.
**Why:** *a production failure becomes a regression test in two clicks.* Every request is logged; only the sample is scored (cost discipline again).

## 3.7 Monitor — `#/monitor` · Dashboard | Online evals

**Show (Dashboard):** the KPI row (Requests · Pass rate · Latency · **Token spend**) → switch the time range — *the data actually changes* → **Add tile ▾** (Tokens per answer · Judge share — one-click, removable, persisted) → alerts with Healthy/Approaching states → **New alert** opens a real rule builder.
**Show (Online evals):** the judge table (scope · sample 5% · on/off · pass-rate trend) → **+ Add online evaluator** — pick a judge, a scope, a sample rate, done.
**Why:** *the dashboard presets are the cost story* (one click, token-led, vs Arize's query-language widgets); *online evals live in Monitor, in plain words* — "the same judge you trust in testing, watching production."

## 3.8 Review — `#/review`

**What:** the human-in-the-loop queue — flagged production spans, one at a time. *(The second capture-back source for Part 1.)*
**Show:** the span card (input · output · auto score) → verdict buttons + label + the **0–1 score slider** + **assignee** → note → **Resolve** — the queue count *drops and stays dropped* → **"Resolve & capture to dataset"** for the failure → the "Queue clear" state.
**Why:** three score shapes (categorical · slider · free-text) because different questions need different fidelity — and every verdict feeds the Alignment view. Braintrust's Review was a documented dead end in our teardown; ours resolves *into* the loop.

## 3.9 Inbox — `#/inbox` (account menu)

**What:** the personal cross-project stream. Two kinds of "done": actionable items clear only by acting; informational items clear by reading.
**Show:** act on the approval request — the item resolves and **the account-menu badge count drops** (derived, never hardcoded).
**Why:** approvals, invites, expiring credentials — the things that block other people — get a surface where reading isn't mistaken for acting.

## 3.10 Settings / Manage — `#/settings`

**What:** deliberately "lite": profile · org members with working role selects · **usage by project in token-spend (task vs judge)** · an honest roadmap panel (Billing · Audit log · SSO/SAML + SCIM).
**Why (say this straight):** *a half-built enterprise plane reads worse than a clear roadmap.* What the demo needs is real; what isn't built says "Roadmap." Usage speaking tokens closes the loop on the cost principle. Operate and Learn stay "Soon" in the nav: staged scope, specced not skipped.

## 3.11 The shell (point these out as you pass them, not as a separate stop)

- **⌘K palette** — real fuzzy search over every entity, context verbs on the open prompt, create actions, theme, and the hidden **Reset demo data**.
- **Ask Keystone** — the copilot that *routes*: every answer ends in a deep link to the right stage.
- **Breadcrumb + sibling switcher** on detail pages; **g-chords** (`g d`, `g p`, `g c`…); collapsible rail; Light/Dark/System; skip-link and live-region a11y; fully responsive.
- **The Build · Evaluate · Observe compass** in the top bar — the IA argument made permanent.
- *(One dev-only route exists, `#/states`, a component-states gallery — not a product screen; mention only if asked.)*

---

## 4 · The decisions register (if Zaki wants the "why" list in one place)

1. **Stage-based IA over flat nav** — wayfinding scored 2/5 on Braintrust's flat list; the loop is the mental model.
2. **Tokens, never fabricated $** — task-vs-judge split is the *actionable* division: judge cost is the eval tax you tune; task cost is what Compress attacks. Arize shows "Cost —" until you configure pricing tables.
3. **Ship populated, teach by touring** — a wired sample project beats empty states + a manual; the tour retires itself into triage.
4. **Connect's unlock pattern** — the product explains its surfaces before data exists (adopted from the best thing our teardown found, made clearer).
5. **Datasets are governed entities** — labels ≠ tags ≠ row-tags; immutable versions; pinned experiments; upsert; provenance; grown from production, not just CSVs. Auditable golden sets.
6. **Prompts are deployables** — environments, gated promotion (evaluation as the gate), attributable results, revertible optimization.
7. **One judge everywhere** — experiments, playground, live traffic; edit once.
8. **Humans grade the judge** — verdicts persist → Alignment matrix → sharper rubrics. The calibration loop neither competitor closes.
9. **Fix actions live on the failure** — trace verdict bar → edit prompt / edit evaluator; Review → capture to dataset.
10. **Honesty as a design rule** — "n=6, directional", "Preview" labels, roadmap cards, "wandered — 2 retries". Trust is the product.

## 5 · Suggested session shape (90 min with Zaki — weighted to the Sunday ask)

1. **Framing + IA** (5 min) — section 0.
2. **Dataset Management, end to end** (30 min) — sections 1.0–1.6. Upload live; show the loop-out and loop-back.
3. **Prompt Management, end to end** (30 min) — sections 2.1–2.6; end on Compress and the gated promotion.
4. **The rest of the product** (15 min) — Part 3, brisk; go deep only where he asks.
5. **Decisions register + what we deliberately didn't build** (7 min) — section 4 + the cuts.
6. **His calls** (3 min) — the decisions to ratify from the Day 5 page.

> The two 30-minute blocks are the point. Everything else flexes around them.
