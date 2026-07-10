# Live Walkthrough — Arize AX · Braintrust · Keystone (flows, IA, UI, usability)

*Live Playwright tour, 2026-07-10. Full-depth capture: each screen gets a
screenshot, IA placement, UI/component notes, and usability observations (10
heuristics + bad-UI flags). Data strategy: empty/first-run states first, then a
demo/sample project. Screens saved under project root as `arize-NN-*.png` etc.*

---

# CSV UPLOAD FLOW — step-by-step, all three platforms

*Same 10-row GST CSV (`test-dataset-gst-qa.csv`: input, expected_output,
category) uploaded to each platform. Steps counted from the Datasets list page.*

## Arize AX — 6 steps, ~40s, 1 bug hit
1. Datasets & Experiments → **+ New Dataset** (or the "Upload a dataset" empty-
   state card) → full-screen modal, mode tabs **Upload CSV | Create Via Skill ✨
   | Create Via Code**.
2. Type **Dataset Name** (required).
3. **Upload File** (or drag-drop). Helper: **Download Sample.csv**.
4. Instant parse → **full 10-row preview table** + **Dataset Schema** step: map
   a column to `reference` (ground truth), prefilled `attributes.output.value`,
   jargon explainer about `{dataset.reference}` in eval templates.
5. **Create Dataset** (footer, far bottom-right). ⚠️ Bug: floating Alyx button
   intercepted the click at our window size — needed a programmatic click.
6. Redirects **straight into dataset detail** (Examples tab, "Data uploaded" ✓),
   with Version stamp + Add Evaluator + New Experiment CTAs ready.
- **Friction points:** schema-mapping jargon at first contact; long mouse travel
  to footer CTA; Alyx overlay bug; uploaded row order not preserved in view.
- **Delights:** sample CSV download; instant full preview; post-create redirect
  into the data; next loop stage one click away.

## Braintrust — 4 steps, ~30s, smartest mapper
1. Datasets → **Upload CSV/JSON** (or Overview onboarding link) → small dialog:
   dropzone + "insert data programmatically" docs link. **No name field.**
2. Drop/select file → full-screen **Import** mapper:
   - Left: raw **Source file** text + **Import preview** table (live).
   - Right: **semantic buckets** — drag column chips into **Input / Expected /
     Metadata / Tags / ID / Do not import**; toggles "Flatten single-column
     values", "Auto-parse objects in strings", "Move all to input".
   - **Auto-inference**: `expected_output` → Expected automatically; `input` →
     Input. (`category` defaulted into Input — Metadata would be smarter.)
3. **Import** (top-right, adjacent to the work).
4. Lands in dataset detail: name **auto-derived from filename** ("Test Dataset
   Gst Qa"), VERSION id, columns Created/Input/Expected/Tags/Metadata/Comments/
   Origin, right Details panel (description, YAML metadata, "Recently used in"),
   header CTAs **Tag · Field schemas · Review · Snapshots · Evaluate in ▾** and a
   floating **Loop agent** button.
- **Friction:** no chance to name during flow (rename later); Input cells render
  as raw JSON (`{"category":…`) because extra columns fold into input; row order
  again not CSV order.
- **Delights:** semantic drag-mapping with auto-inference (best-in-class);
  source + preview side-by-side; "Evaluate in ▾" = next loop stage one click
  from the data; versioning + snapshots built in.

**Verdict so far:** Braintrust's mapper UX > Arize's (semantic buckets vs.
jargon dropdown), Arize's naming + sample-CSV + explicit ground-truth framing >
Braintrust's. Keystone should combine: name field + sample CSV + semantic
buckets in plain language ("Which column is the correct answer?").

## Keystone (keystone-hifi) — 4 steps, best mapper language, misses the redirect
1. Datasets → **+ New dataset ▾** → menu covers ALL five provenance paths:
   **Empty · Upload CSV · Create via skill ✨ · Create via code · From traces**
   (the union of Arize's 3 + Braintrust's 2).
2. Compact modal: **Dataset name** (pre-filled + pre-selected for overtype,
   **live slug preview** `/gst-faq-golden-set-parity-test`), optional
   Description ("What this dataset tests, and where it came from"), mode tabs,
   dropzone that **pre-announces the next step** ("Columns map to Input ·
   Expected · Metadata · Tags in the next step") + two escape hatches
   ("use the sample GST FAQ data", "try a messy export").
3. Drop file → **in-modal mapping step**: file chip ("test-dataset-gst-qa.csv ·
   10 rows" + Clear), semantic buckets **with plain-language definitions**
   (INPUT "The case to recreate — required" · EXPECTED "Ground truth /
   reference output" · METADATA · TAGS · ID (UPSERT KEY) · NOT IMPORTED),
   drag **or keyboard** ("works with a keyboard too"), auto-mapping
   (input→Input, expected_output→Expected, category→Input), **live preview
   with mapping shown in headers** (`expected_output → Expected`).
4. **"Import 10 rows"** (count in the CTA) → success toast **"10 inserted ·
   0 updated in 40.0s"** (upsert semantics surfaced), KPIs update, row appears
   at v1.
- **Friction:** returns to the LIST, not into the new dataset (Arize/BT redirect
  in — better); `category` defaults to Input (same flaw as BT — should suggest
  Metadata).
- **Delights:** name+slug, bucket definitions, keyboard mapping, outcome-stating
  CTA, upsert-aware toast, messy-export test path (unique).

### Three-way CSV verdict
| | Arize | Braintrust | Keystone |
|---|---|---|---|
| Steps | 6 | 4 | 4 |
| Name during flow | ✅ required | ❌ from filename | ✅ pre-filled + slug |
| Mapping model | 1 "reference" dropdown, span-attr jargon | semantic buckets, drag, auto-infer | semantic buckets **+ plain definitions + keyboard** |
| Preview | full table | source + parsed table | live preview w/ mapping headers |
| Post-create | → detail ✅ | → detail ✅ | → list ⚠️ |
| Unique | sample CSV download | auto-inference toggles | messy-export path, upsert toast |
- **Fix for Keystone:** redirect into the dataset (or add "View dataset" to the
  toast) + auto-suggest Metadata for unmapped extra columns.

---

# PART A — ARIZE AX

Account: fresh **Free plan**, empty (0/25k spans, 0 MB/1 GB) → true first-run.

## A.0 Login (`/auth/login`)
- **UI:** split-screen. Left = brand "Arize AX" + hero *"The Agent Improvement
  Loop / The AI engineering platform for continual learning. Observe. Evaluate.
  Improve."* + customer logos (Reddit, DoorDash, Instacart, Motorway, Pepsi,
  Priceline). Right = form.
- **Form:** **Data Region** selector (US) *on the login screen*, Email, Password
  (show/hide), Forgot Password, Log In, then Google / GitHub / **SAML SSO**,
  Sign Up.
- **Usability:** clean, conventional, trustworthy (5/5 polish). Data-region on
  login = enterprise/residency signal up front; mild extra load for a solo
  founder. Hero names the loop — sets the mental model before you're even in.

## A.1 Home / first-run (`/organizations/{org}/spaces/{space}`)
- **IA — left rail, grouped BY THE LOOP:**
  - Top: Arize AX logo + collapse; **Space switcher** ("mdamkhan Space"); Search (Ctrl+K)
  - **Home**
  - **Observe**: Tracing Projects ▸ · Monitors · Dashboards
  - **Evaluate**: Evaluators · Labeling Queues
  - **Improve**: Datasets & Experiments · Prompt Playground (beta) · Prompts
  - **More ▸**
  - Bottom: **Free Plan Usage** (Spans 0/25k, Storage 0/1GB) · **Upgrade Account**
    (purple) · Invite User · Settings · Help ▸ · account (Amaan Khan)
- **Content (empty state):** greeting "Good Afternoon, Amaan". **Alyx** is the
  hero: *"Meet Alyx, your observability agent"* — chat box ("Ask a question, type
  @ for context"), model selector (Claude-sonnet-5), Recent chats, starter chips
  ("How do I send in traces?", "How do I use Arize?"). Below: **GET STARTED**
  card — *"1 Observe your agent — send your first trace…"* + **Start Tracing** +
  a sample trace waterfall preview (Agent/Retrieval/Tool call/LLM). Fallback link
  *"Not ready to set up traces? Run a prompt experiment instead."*
- **Usability:**
  - ✅ Never a blank wall — Alyx + Get Started + sample-trace viz fill the empty
    state; two onboarding paths (trace OR prompt experiment).
  - ✅ AI assistant as onboarding concierge (answers "how do I…" in-context).
  - ✅ Transparent usage meter always visible (also a persistent upsell).
  - ✅ Nav taxonomy = the loop (Observe→Evaluate→Improve) — strong mental model.
  - ⚠️ *Bad-UI flag (low):* "Datasets & Experiments" sits under **Improve** while
    "Evaluators" sits under **Evaluate** — experiments *are* evaluation; the split
    muddies the model. *Rec for Keystone:* keep Datasets + Evaluators +
    Experiments together under one **Evaluate** group (we already do this ✅).
  - ⚠️ *Bad-UI flag (low):* the "**1** Observe your agent" numbering implies a
    sequence (2, 3…) that isn't shown. *Rec:* either show the full numbered
    checklist or drop the number.
  - Note: Arize leads the nav with **Observe** (observability-first); Keystone
    leads with **Build** (build-first). Different emphasis — worth a deliberate
    choice.

## A.2 Tracing Projects — empty (`/projects`)
- **UI:** "Tracing Projects hold your agent's telemetry data…". Live badge
  **"Listening for traces… 5s"** (auto-advances when a trace lands). Header
  chips: **Space ID**, **Create API Key**. **TERMINAL** onboarding block:
  `npx skills add Arize-ai/arize-skills` then *"Ask your AI coding agent:
  'Instrument my agent to send traces to Arize AX'"* — works with Cursor, Claude
  Code, Codex. Links: "See docs", **"Instrument Manually"**, and *"Not ready to
  log traces? See the demo project instead."*
- **Alyx** auto-opened as a **right-docked panel** (persists across all screens):
  "Make the most out of Alyx", context chip *"Help me send traces from my
  agent"*, model selector, `@`-for-context input, and a persistent bottom nudge
  *"Tell Alyx about yourself… [Set up]"*.
- **Usability:**
  - ✅ Very modern, low-friction dev onboarding (AI-coding-agent instrumentation).
  - ✅ Live listener = instant success feedback; demo-project escape hatch.
  - ✅ Alyx docked right, context-aware — direct parallel to our Ask Keystone.
  - ⚠️ *Bad-UI flag (med):* clicking the "Tracing Projects" nav item opened a
    **"Recently viewed" hover-flyout that overlaps and obscures the page H1 +
    description**. *Rec:* nav item should navigate cleanly; don't overlay a
    flyout on the destination content.
  - ⚠️ *Bad-UI flag (med):* onboarding is still **developer-first** (terminal,
    npx, API keys) — a non-technical builder is blocked at the door even though
    Alyx is present. *Rec for Keystone:* offer a no-code path (paste a key /
    connect, or "explore with sample data") as the PRIMARY CTA, terminal as
    secondary.
  - ⚠️ *Redundancy (low):* two Alyx surfaces now (Home hero + right dock) — unclear
    where "the" assistant lives. *Rec:* one canonical assistant location.

## A.3 Monitors — empty (`/monitors?selectedSubtab=llmMonitors`)
- **UI:** "Monitors / Automatically detect issues". **+ New Monitor**. Search +
  filters (All Statuses · Muted & Unmuted). Subtab param hints at monitor
  *types* (LLM monitors; ML-heritage drift/data-quality likely elsewhere).
  Empty state: **"Create Your First Monitor"** + View Docs + two quick-create
  template cards: **Latency Monitor**, **Token Count Monitor**.
- **Usability:** ✅ consistent empty-state pattern; quick-start templates lower
  the barrier. ⚠️ persistent "Tell Alyx about yourself [Set up]" banner is mildly
  nagging on every screen.

## A.4 Dashboards — empty (`/dashboards`)
- **UI:** "Dashboards / Check and share key metrics". **+ New Dashboard**.
  Empty state: **"Create Your First Dashboard"** + View Docs + three templates:
  **Tracing Project Overview** (trace volume, latency, model perf, token usage,
  **cost tracking**, eval scores), **Token Tracking and Latency**, **Custom
  Dashboard** (build-your-own).
- **Usability:** ✅ real **custom dashboard builder** with templates — a
  capability our Keystone Overview lacks (**gap to close**). ⚠️ *Bad-UI flag
  (low):* copy says "analyze your **models**" — ML-legacy wording that reads
  off for agent builders. *Rec for Keystone:* speak "agents/apps", never "models".

## A.5 Evaluators — empty (`/evaluators?selectedTab=evaluators`)
- **UI:** header "Evaluators / Manage your evaluators and running tasks" + two
  actions: **+ New Task**, **+ New Evaluator**. Tabs: **Evaluator Hub |
  Running Eval Tasks** (definitions vs. executions — nice separation). Empty
  state headline: **"Turn judgment into a repeatable signal"** (excellent copy —
  explains WHY evals in seven words). Template cards: **Start with Alyx** ("a
  usefulness judge that compares input to output" — AI builds your first eval),
  **Hallucination**, **User Frustration**. Alyx panel contextualizes: "What is an
  evaluator, and why should I create one?" / "Recommend and build an eval for my
  use case".
- **Usability:** ✅ best empty state so far — value-prop headline + AI-assisted
  first eval + concrete templates. ⚠️ *(low)* "New Task" vs "New Evaluator"
  distinction is jargon at first contact (a "task" = a scheduled eval run —
  unexplained). *Rec:* Keystone should name these plainly ("Evaluator" +
  "Schedule run") or explain inline.

## A.6 Labeling Queues — empty (`/queues`)
- **UI:** "Labeling Queues / Add human feedback to LLM application data points."
  **+ New Labeling Queue**. Empty state: "Assign data to subject matter
  experts/3rd parties for custom annotation. Use labeled data to build golden
  datasets for fine-tuning." Single card: New Labeling Queue ("add records
  later"). Alyx suggestions adapt per screen (make/edit/custom view for queues).
- **Usability:** ✅ consistent pattern; SME/3rd-party assignment framing is
  enterprise-friendly. ⚠️ *(low)* "LLM application data points" — more insider
  jargon vs. Braintrust's plain "review".

## A.7 Datasets & Experiments — empty + creation flow (`/datasets`)
- **Empty state:** "Create a dataset to test and improve your app". Three
  creation paths: **Start with Alyx** (generated starter customer-support set),
  **Create regression dataset from traces** ("turn low-scoring user interactions
  into a regression set" — the Braintrust-style capture loop), **Upload a
  dataset**. **New Experiment is disabled until a dataset exists** (good
  dependency communication). Alyx suggestion: "Generate a synthetic dataset".
- **New Dataset modal:** left tabs **Upload CSV | Create Via Skill ✨ | Create
  Via Code**. Name field, drag-drop zone, **Download Sample.csv** helper.
- **Upload test (our 10-row GST CSV):** parsed instantly; full 10-row preview
  table; **Dataset Schema mapping** — map a column to `reference` (ground
  truth), pre-filled `attributes.output.value`, with explainer text about using
  `{dataset.reference}` in eval templates.
- **Usability:**
  - ✅ Excellent creation triad (AI-generate / from-traces / upload) — covers all
    three provenance types we planned for Keystone.
  - ✅ Sample CSV download + instant preview = low-friction, confidence-building.
  - ⚠️ *Bad-UI flag (med):* the schema `reference` mapping is powerful but
    **jargon-heavy at the worst moment** (first upload): "attributes.output.value"
    and `{dataset.reference}` assume you already know Arize's span-attribute
    model. *Rec for Keystone:* plain-language mapping ("Which column is the
    correct answer?") with the technical detail collapsed.
  - ⚠️ *(med)* Modal is full-screen-ish and the Create button sits far bottom-right,
    ~900px from the preview content — long mouse travel; footer actions feel
    detached on large screens.

## A.8 Home — POPULATED (second space "AskJolly", 28.7k traces)
- Home reshapes with data: Alyx hero becomes **"Let's dig into your traces"**
  with data-aware quick actions: *Find critical issues in my traces · Help me
  build an eval · Summarize my experiment results*. **Recent 5** projects strip
  (sparkline cards, metric switcher Traces/…). **What needs your attention**
  panel — here promoting **Signal** (auto-issue detection) with a *waitlist* CTA.
  **Recently viewed** panel.
- **Usability:** ✅ Home = adaptive mission control (empty→onboarding,
  populated→triage). This is the pattern for Keystone's Overview. ⚠️ *(low)*
  "What needs your attention" showing a feature ad (Signal waitlist) instead of
  actual attention items dilutes the panel's promise.

## A.9 Datasets — POPULATED list + CREATED our dataset ✅
- **List view:** table (Name, # Examples, # Experiments, Created By, Last
  Updated, Tags) + per-row quick actions: **Edit tags**, **Run dataset in
  playground** (loop-hop from the list!), options menu. Real client data present:
  "GST FAQ Sample (12)" and "Official GST FAQs v1" (1,593 examples, 10 experiments).
- **We created "GST FAQ Golden Set (Esberi test)"** via Upload CSV → succeeded.
  Redirects straight to **dataset detail**:
  - Header: back · dataset name · Info · tags · **Version selector (timestamped
    "2026-07-10 10:11:00")** · **Add Evaluator** · **+ New Experiment** · ⋯
  - Tabs: **Examples (10) | Experiments** · "Data uploaded" confirmation ✓
  - **Example Query** filter bar ("find all examples that match conditions")
  - Table: Example ID (hash chip), Input, expected_output, **Evaluations,
    Annotations, Trace Annotations**, category (+ Columns picker, export,
    row-height, + Examples).
- **Usability:**
  - ✅ Post-create redirect straight into the data = instant gratification.
  - ✅ Dataset **versioning** + next-stage CTAs (Add Evaluator / New Experiment)
    on the detail header — the loop is walkable from here.
  - ⚠️ *Bad-UI flag (low):* **uploaded row order not preserved** in the examples
    table (CSV row 7 appears first) — confusing when verifying an upload.
  - ⚠️ *Bad-UI flag (med):* the floating **Alyx button overlaps the modal footer**
    at smaller window sizes — it intercepted our click on "Create Dataset"
    (button visually reachable but unclickable). Classic z-index/overlay bug.
    *Rec for Keystone:* keep the copilot trigger out of dialog hit-areas.

## A.10 Tracing Projects — POPULATED list (`/projects`)
- Table: Name, Triggered Monitors, **Last 7 Days Volume (sparkline)**, Tags,
  Created At, ⋯. Two **Demo**-badged projects (generative-llm-tracing 5,390;
  llm-travel-agent 4,091) + Playground Traces 28,786.
- ⚠️ *Bad-UI flag (HIGH — confirmed twice):* the nav's "Recently viewed" flyout
  (a) visually covers the page title/first rows AND (b) leaves an invisible
  **underlay that blocks ALL clicks** on the list until dismissed with Escape.
  We could not click a project row. *Rec:* never trap pointer events under a
  passive flyout.

## A.11 Project view — Traces tab (demo travel agent)
- **Tabs:** Traces | Spans | Sessions | **Agent Graph** | **Agent Path** |
  Evals & Metrics (4) | Signal. Header: time range + **Live** toggle + **+ Add
  Online Evaluator** + ⋯. Filter row: saved views ("Arize Default") + **"Filter
  spans or ask Alyx"** NL filter + Add Filter.
- **KPI strip:** Traces 355 · Spans 8,451 | P50 1.24s · P99 13.2s | Tokens 1.271M
  | **Cost — "No data. Set up cost configs"**.
- **Trace table:** Status ✓, Start Time, Kind chip (AGENT / RETRIEVER /
  UNKNOWN), Name, Input, Output, Latency, Span Evaluations, **Trace Evaluations
  (inline chips: "Agent Trajectory Eval: correct/incorrect")**, Session Evals.
- **Usability:**
  - ✅ Inline eval chips on every trace row = quality visible at the list level.
  - ✅ Saved filter views + NL filtering; Live mode; export; column picker.
  - ⚠️ *Bad-UI flag (HIGH):* default time range **Last 15 Min** made a project
    with 355 traces show **"No Data"** — while the KPI strip simultaneously said
    "1 trace" (contradiction). A new user would conclude instrumentation is
    broken. *Rec for Keystone:* default to a window guaranteed to show the most
    recent data ("Last 7 days" or auto-fit-to-data), and never let two regions
    disagree.
  - ⚠️ *(med)* **Cost is empty until you configure "cost configs"** — token
    counts exist but $ requires setup. *Rec:* Keystone ships cost defaults per
    model, zero-config (our differentiator).
  - ⚠️ *(low)* Longer retention ranges (Last Month+) are disabled on this plan —
    grayed with no tooltip explaining why. ⚠️ *(low)* Input column renders raw
    JSON; Kind=UNKNOWN chips unexplained.

## A.12 Trace detail slideover — Trace Tree + Agent Graph ⭐
- **Layout:** 3 panels — trace list stays left; middle **Trace Tree (24) |
  Timeline | Agent Graph** tabs; right span detail (**Input/Output | Attributes**
  + "Customize Tabs", Pretty/raw toggles).
- **Tree:** root TripAgentGraph (AGENT, "1 eval", 8.8s) → research_agent /
  local_agent / budget_agent / itinerary_agent, each with nested ChatOpenAI +
  tool/retriever spans, **per-span duration AND token count inline** (e.g.
  "1.05s, 376 tk"). Span search. Session chip; trace-eval chip in header.
- **Right panel:** structured key-value Input (destination Tokyo, budget $900…),
  pretty Output (the itinerary), span-kind chip.
- **Agent Graph tab:** node graph **Start → [Budget, Local, Research] →
  Itinerary_agent → End**, color-coded (start green / agents purple / end red),
  zoom controls. One glance = topology. **The most compelling agent-native view
  in the product — Keystone gap.**
- **Usability:** ✅✅ this whole surface is the industry bar. ⚠️ *(low)* trace list
  column shows name truncated ("TripA…") by default; ⚠️ *(low)* "Cost --" dead
  metric repeated in trace header.

## A.13 Prompt Playground (`/prompt-hub/playground`)
- **Playgrounds list:** saved, named configurations (ours "GST QnA UX Review" +
  Zaki's "Official GST FAQs v1") — collaborative objects with author + timestamps,
  not throwaway scratch.
- **Working surface:** prompt editor (role-tagged messages, `{question}` variable
  highlighted, **Load a prompt** from Hub, model selector claude-opus-4-8, params,
  + Message, + Functions); right-edge **+** adds a comparison column (A/B);
  **Run History**; **Run**.
- **Bottom half = the loop:** attached **dataset** ("GST FAQ Sample (12)" → "Run
  on all 12 rows") + attached **Evaluators (1)** + Columns. Output header
  aggregates: **Total 12 · Avg 11.83s · Avg 804 tk · Avg $0.017862 · "Avg
  Accurate Q&A Judge: 0.8"**. Each row: input variable + rendered markdown
  output with per-row latency/tokens.
- **Usability:** ✅ playground = prompt × dataset × evaluators with aggregate
  quality **and $ cost** — the strongest expression of the loop in one screen;
  the model for Keystone's PromptRun. ⚠️ *(med, consistency)* cost shows here
  zero-config while Tracing demands "cost configs" — same metric, two rules.

## A.14 Arize wrap — not walked (time-boxed)
Prompts hub detail, More menu (ML/CV legacy views live there per research),
Settings/API keys, Monitors creation flow, Experiments compare view. Enough
captured for the comparison; revisit if needed.

---

# PART B — BRAINTRUST

Account: org **esberi**, project "My Project", Starter plan (fresh). Rail shows
**Starter plan usage: Credits $0/$10 · Logs 0/1 GB · Scores 0/10,000** — the
per-score metering is visible in the nav itself.

## B.0 Overall IA & first impression
- **Light theme**, spacious, plain typography. Feels like Linear/Notion, not a
  console. Non-engineer-friendly at first glance.
- **Flat project nav (14 items, no stage grouping):** Overview · Logs · Monitor ·
  Topics · Review · Playgrounds · Experiments · Datasets · Prompts · Scorers ·
  Parameters · Tools · SQL sandbox · Settings. Org switcher above ("esberi" →
  "View all" projects).
- **Loop-first Overview:** hero "What can I help you with?" + "Add an AI
  provider to get started with Loop" (the AI agent IS the front door), then 3
  onboarding columns: **Observability** (Set up tracing / Install bt CLI /
  Define human review scores), **Evaluation** (Upload a dataset / Add a prompt /
  Create an experiment), **Suggestions** (Invite team / AI providers / Alerts /
  Slack).
- ⚠️ *Bad-UI flag (med):* a **marketing popup (GLM-5.2 promo)** sits over the
  bottom-left of EVERY screen during first-run, covering part of the nav until
  dismissed. Promo > user on day one.
- ⚠️ *(med)* 14 flat nav items = the "surface sprawl" from research, confirmed —
  no wayfinding for which stage you're in; Parameters/Tools/SQL sandbox
  intimidate non-engineers. Contrast: Arize groups by loop stage; Keystone
  groups by Build/Evaluate/Observe (ours is the clearest).

## B.1 Datasets — empty state + detail
- Empty: explainer + cross-link "Don't have any data yet? **Set up tracing**…" +
  **Upload CSV/JSON | Empty dataset**. (No AI-generate path, unlike Arize.)
- Detail (post-import): VERSION id, **saved views**, Add filter, columns
  Created/Input/Expected/Tags/Metadata/Comments/**Origin**, right **Details
  panel** (description, YAML metadata, "Recently used in"), header **Tag ·
  Field schemas · Review · Snapshots · Evaluate in ▾** + floating **Loop agent**.
- ✅ "Evaluate in ▾" = the loop-hop CTA; Comments column = collaboration
  built into the data grid.

## B.2 Logs — empty state ⭐
- **"Waiting for logs"** live listener. **One-line curl wizard**
  (`curl -fsSL …/wizard/setup.sh | sh`) that "automatically instruments your
  repo and handles API key configuration" + "Set up manually instead".
- **"Unlocks with your first trace"** strip: Monitor / Review / Scorers /
  Datasets cards, each explaining what it does once data arrives — the loop's
  dependency graph made visible. **Best onboarding pattern of the whole tour;
  adopt for Keystone.**
- Header: Review · Enable topics · Automations.

## B.3 Playgrounds — list + surface ⭐
- Empty state: copy + **Create empty playground** + **three seeded starter
  examples** (Compare prompts / Compare models / Custom scorers). ⚠️ *(low)*
  clicking a starter silently creates a list item instead of opening it — extra
  click, momentary confusion.
- **Surface (the flagship):** up to N **side-by-side task columns** (Base +
  Comparison tasks), each with model selector, Params, message editor with
  `{{input}}` Mustache highlighting, Text-output/Tool-MCP attach, **Save
  prompt** (Draft badge). Header: **Diff toggle · + Experiments · Run**.
- Bottom: attached **dataset grid** (27 rows) + **+ Row · + Task (3) · + Scorer
  (1)** chips (ExactMatch attached, removable ×), saved views, Display; per-
  comparison **statistical verdicts** ("Tie. No significant difference");
  "This cell has not been run yet" placeholders; floating **Loop** button.
- ✅ The A/B/n prompt-compare workbench in one screen — stronger multi-variant
  UX than Arize's + column approach. The bar for Keystone's PromptRun.

## B.4 Scorers — empty state
- "Create from scratch": **LLM judge scorer | Code scorer** (the two families,
  plainly named) + 6 AutoEvals templates: Factuality, Closed Q&A, Security,
  Possible, Summary, ExactMatch — one-line plain-English descriptions each.
- ✅ Mirrors Keystone's Evaluators template gallery; cleanest eval creation
  entry of the three platforms.

## B.5 Review — empty state (weakest screen)
- ⚠️ *Bad-UI flag (med):* text-only dead end: "Visit project configuration to
  create human review scores…" — a link into Settings, **no CTA button, no
  template, no explanation of what review looks like**. Breaks their own
  empty-state pattern. *Rec for Keystone:* Review empty state should offer
  "Create your first review score" inline + a preview of the review UI.

## B.6 Not walked (empty account, time-boxed)
Monitor (empty — needs logs), Topics, Prompts detail, Experiments detail
(created via playground only), org Settings/RBAC, SQL sandbox. Covered by web
research; revisit with data if needed.

---

# PART C — KEYSTONE (keystone-hifi, localhost)

Context: production-fidelity React/TS prototype of the Esberi product (the
public `esberi/esberi.github.com` repo is only the static marketing site — no
product code there; Keystone is the deepest expression of the product).
Sample org ESBERI · project AskJolly, populated GST QnA data. Light theme.

## C.0 IA & shell
- Rail grouped **by loop stage** (Build / Evaluate / Observe + Soon: Operate ·
  Learn · Manage) + Overview + **Pinned** + global **+ Create**. Top bar:
  Search ⌘K · **Ask** (copilot) · account w/ actionable badge (3). Collapse
  with `[` kbd hint. g-chord nav, breadcrumb + sibling-switcher on detail pages.
- **Clearest wayfinding of the three**: Arize groups by loop but muddles
  Datasets/Experiments under "Improve"; Braintrust is a 14-item flat list;
  Keystone's Build→Evaluate→Observe reads as a story.
- ⚠️ Prototype note: hash routing — path URLs (`/datasets`) silently render
  Overview; real build needs proper URL routing.

## C.1 Overview — the guided loop ⭐
- **"Tour the GST QnA sample"** — 5 steps grouped BY STAGE (Build: see dataset /
  open prompt · Evaluate: see evaluator / open scorecard · Observe: read a
  trace), progress "0 of 5 seen", per-step CTA + Mark seen, Skip tour.
- Right: **"Model connection ready — your API key stays hidden"** + Manage
  connection; **"Learn the loop"** card ("measures whether your AI is good
  enough — **and how many tokens it takes**" — cost thesis in the first
  paragraph) + "What is an evaluation loop?" + "See it end-to-end →".
- ⌘K hint bar; **Recent activity** with outcome chips (Pass rate 83% · Scored
  0.40 · Saved as v13).
- ✅ The most explicit loop-teaching of the three platforms; populated sample
  data from minute one (vs Arize's dev-terminal, BT's provider-key gate).

## C.2 Datasets list + CSV flow
(see CSV flow section above) — KPI strip (14 datasets · 5 curated · 11,969
records · 12 experiments this week), first-run banner with promise copy,
version chips, **tags with promotion state** (`golden → v3`,
`regression-suite → v1`), team labels, curated ✓ badges, Draft badge,
archived-hidden note, pagination.

## C.3 Traces list — the cost thesis at row level ⭐
- Subtitle: "Every run's spans, scores, and the evaluator's reasoning —
  **token-spend split task vs judge**."
- Columns: Row id · Input · **Score chip (Pass 0.90 / Fail 0.40 / Partial
  0.67)** · Source (#7 → experiment) · **Tokens split "1,240 · 760+480"** ·
  Latency · When. Search spans input/output/**reasoning**; status/source
  filters.
- ✅ Unique vs both competitors: per-row task-vs-judge token split. (Arize has
  inline eval chips; neither splits spend.)

## C.4 Trace detail — verdict → fix, one screen ⭐
- Header: ← Experiment #7 · **Row 3 of 6 pager · "Next failing row"** · Export.
- **Verdict bar:** Fail · Score 0.40 · threshold (Pass ≥ 0.70) · id · **1,180
  tok (task 700 + judge 480)** · 2.3s · **Edit the evaluator | Edit the prompt
  →** (fix actions live on the failure).
- Left: Tree | Timeline, spans with per-span tokens; token-spend equation.
- Right: evaluator span — Reasoning | Messages | Raw JSON; criterion; judge's
  plain-English REASONING; **Output vs Expected side-by-side**.
- **HUMAN REVIEW block inline:** "Override the judge on this row and record
  why — used to calibrate the evaluator" (Agree / Judge is wrong + note),
  "Awaiting your review" chip → judge-calibration built into the trace.
- **Ask Keystone** docked right with contextual actions (Summarize experiment
  #7 · Generate 10 synthetic GST rows · Optimize the GST QnA prompt · What's
  failing in production? · Build an eval for JSON validity) — each answer
  deep-links. Model: Claude Opus 4.8.
- vs Arize's trace view: Arize wins on span-tree depth, Timeline, Agent Graph,
  session linkage; Keystone wins on verdict→fix loop, token split, inline
  calibration, next-failing-row triage. **Adopt Arize's agent graph; keep our
  loop mechanics.**

## C.5 Prompt Compress (differentiator, confirmed live)
- Prompt shell: GST QnA **v13 · production** badge, tabs Edit | Run | Manage |
  **Compress**, pin, Run. Method cards ×3 (LLMLingua / LongLLMLingua /
  LLMLingua-2 with one-line tradeoffs), **Keep rate ⇄ Target tokens** toggle +
  slider ("Keep ~60% of the prompt"), Preserve digits, Compress.
- Tiles: **Before 34 tokens · After — · Saved — · Cost saved/run —**; preview
  with `{{question}}` protected chip + legend; **Apply disabled until computed**
  with explainer; applies as a new version, original kept.
- ✅ Nothing remotely like this in Arize or Braintrust.

## C.6 Not audited this pass
PromptRun/Edit/Manage detail, Evaluators gallery, Experiment detail, Monitor,
Logs capture flow, Review queue, StatesShowcase. Known from code; include in
the full artifact if needed.

---

# PART D — SYNTHESIS

## D.1 Ease-of-use scorecard (1–5 per heuristic, from the live tour)

| Heuristic | Arize AX | Braintrust | Keystone |
|---|---|---|---|
| 1. Time-to-first-value | 3 — dev-terminal onboarding; demo project saves it | 3 — needs AI provider key / tracing first | **5** — populated sample + guided tour instantly |
| 2. Wayfinding (where am I in the loop) | 4 — nav grouped by loop (minor Improve/Evaluate muddle) | 2 — 14 flat items, no stages | **5** — Build→Evaluate→Observe + tour by stage |
| 3. First-run / empty states | 4 — consistent "Create your first X" + templates + Alyx | 3 — great Logs ("Unlocks with…"), weak Review; promo popup | **5** — sample-data-first, no empty walls |
| 4. Cognitive load / jargon | 2 — span-attr paths, "eval tasks", ML legacy terms | 4 — plain words, but Parameters/Tools/SQL lurk | **4–5** — plain definitions everywhere |
| 5. Discoverability of the loop | 3 — powerful but scattered; Alyx compensates | 3 — strong object links ("Evaluate in ▾"), weak overview | **5** — tour + copilot that routes |
| 6. Consistency | 3 — cost works in Playground but needs config in Tracing; two Alyx surfaces | 4 — consistent light UI; Review breaks pattern | 4 — consistent shadcn system |
| 7. Feedback & recoverability | 3 — "No Data" vs "1 trace" contradiction; live listeners good | 4 — live "Waiting for logs", verdict labels | **5** — upsert toast, disabled-until-computed with explainer |
| 8. Non-engineer accessibility | 2 — engineer's console | 4 — cross-functional by design | **5** — designed for it |
| 9. Flow friction (core loop hops) | 3 — good CTAs (dataset→experiment) but 6-step CSV, modal reach | 4 — one-click hops everywhere | 4 — great hops; misses post-import redirect |
| 10. Polish & a11y | 4 — slick dark UI; overlay bugs | 4 — clean; persistent promo | **5** — skip-link, aria-live, kbd mapping |
| **Total /50** | **31** | **35** | **47** |

*Caveats: Keystone is a prototype scored on design intent (no real data/perf);
Arize was scored across two accounts (empty + populated); Braintrust scored
mostly on empty states. Directionally solid, not a lab study.*

## D.2 Bad UI decisions — ranked, with recommendations (for Amaan to accept/reject)

**ARIZE**
1. 🔴 **Default 15-min time window ⇒ "No Data" on a full project** (+ KPI strip
   contradicting the table). *Rec:* default to last-7-days or auto-fit to most
   recent data; never let two regions disagree. → Keystone: adopt auto-fit.
2. 🔴 **"Recently viewed" nav flyout blocks the page** (visual overlap + an
   invisible underlay that swallows ALL clicks until Escape). *Rec:* flyouts
   must never trap pointer events; hover-intent + auto-dismiss on page click.
3. 🟠 **Floating Alyx button intercepts modal footer clicks** at smaller
   windows. *Rec:* copilot trigger must yield to dialogs (z-index + inert).
4. 🟠 **Cost requires "cost configs" in Tracing while Playground shows $ free**.
   *Rec:* ship model-price defaults; cost is zero-config (Keystone already
   plans this — keep it).
5. 🟠 **Schema-mapping jargon at first upload** (`attributes.output.value`,
   `{dataset.reference}`). *Rec:* plain question + collapsible advanced.
6. 🟡 "New Task vs New Evaluator" unexplained; "models" wording; disabled
   retention ranges without tooltip; "1" numbered checklist with no 2/3;
   Datasets&Experiments under "Improve" while Evaluators under "Evaluate".

**BRAINTRUST**
7. 🔴 **Review empty state is a dead end** (text link into Settings, no CTA, no
   preview). *Rec:* inline "create first review score" + queue preview.
   → Keystone: make Review's empty state a first-class guided state.
8. 🟠 **Marketing popup (GLM-5.2) over the nav on first-run, on every page**.
   *Rec:* never let promos occupy first-run; use the changelog surface.
9. 🟠 **14-item flat nav** — no stages, Parameters/Tools/SQL sandbox intimidate
   non-engineers. *Rec:* group by loop stage (Keystone already does).
10. 🟡 No dataset name during import (filename-derived); starter example
    silently creates a list item instead of opening; `category` auto-mapped to
    Input; score-based metering surfaced only as a usage meter.

**KEYSTONE (ours — fixes to make)**
11. 🟠 **Post-import lands on the list, not the new dataset**. *Rec:* redirect
    into detail (or "View dataset" in the toast). — cheap fix, high payoff.
12. 🟠 **No agent-topology view** (Arize's Agent Graph is the reference).
    *Rec:* add an Agent Graph / trajectory tab on Trace detail + convergence
    metric on Experiments.
13. 🟡 `category` column defaults to Input (mirror BT flaw). *Rec:* suggest
    Metadata for unrecognized extra columns.
14. 🟡 Hash routing breaks deep links (prototype artifact). *Rec:* real routes
    in dev handoff.
15. 🟡 No custom dashboard builder (fixed Overview). *Rec:* Phase-2 tile
    composer (both competitors have one).

## D.3 What to steal (best-of-each for Keystone)
- **From Arize:** Agent Graph tab ⭐; adaptive Home (empty→onboarding,
  populated→triage); saved filter views + NL filter box on traces; "Run dataset
  in playground" row action; playground aggregate header (avg $ + judge score);
  timestamped dataset versions; Live-mode toggle.
- **From Braintrust:** "Unlocks with your first trace" dependency onboarding ⭐;
  one-line curl/wizard instrumentation; semantic drag-mapper toggles
  (auto-parse, flatten); "Evaluate in ▾" on every dataset; statistical verdicts
  ("Tie — no significant difference") on comparisons; Comments column on data
  grids; seeded starter playgrounds.
- **Keep ours (unique):** token split task-vs-judge everywhere; Compress;
  verdict-bar fix actions; inline judge calibration; next-failing-row triage;
  guided stage tour; copilot-that-routes.
