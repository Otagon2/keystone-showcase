# Prompt Management — The Domain Guide

*Written 2 Jul 2026. Purpose: make you (Amaan) a domain expert — able to explain what prompt management is, how the market does it today, what Keystone has and wants, and which assumptions about the future should shape our design. Written in plain language; every term is defined the first time it appears. Facts about Langfuse and Phoenix were verified against their live docs today (marked ✓); wider-market observations come from research knowledge and get re-verified in Phase 1 (marked ○).*

**How this doc is organized — exactly the three things you asked for:**
- **Part A — Full understanding:** what prompt management *is*, from zero.
- **Part B — Current state:** the market (Langfuse, Phoenix, the wider field) and Keystone (repo reality vs PRD target).
- **Part C — Future assumptions → design decisions:** the bets about where this field is going, and the specific design choice each bet drives.

---

# PART A — FULL UNDERSTANDING

## A1. What a prompt actually is

A **prompt** is the written instruction you give an AI model. That's it. "Summarize this email in two sentences" is a prompt. So is a 3,000-word set of rules telling a support agent how to behave, what tone to use, and what it must never say.

The crucial mental shift — the one this whole field is built on: **a prompt is not throwaway text; it is source code.** It determines what an AI product does, exactly the way code determines what an app does. Change one sentence in a prompt and your product can behave completely differently — better, worse, or embarrassingly. Yet unlike code, prompts are written in English, so *everyone* feels qualified to edit them, and most teams store them nowhere in particular: pasted in a playground tab, buried in the codebase as a string, copied into a Google Doc titled "FINAL prompt v3 (new)".

**Prompt management** is the discipline (and the product surface) that treats prompts the way software teams treat code: stored in one place, versioned, testable, reviewable, deployable, and observable.

## A2. Why it exists — five failure stories

Every feature in a prompt-management tool exists because some team got burned. These five stories *are* the requirements:

1. **"Who changed the prompt?"** — The support bot started being rude on Tuesday. Nobody knows what changed, because the prompt lives in the codebase and three people edited it. → *Need: versions, history, diffs, audit.*
2. **"It worked on my example."** — A builder tests a prompt on one lucky input, ships it, and it fails on 30% of real traffic. → *Need: batch testing on real data, evaluation, side-by-side comparison.*
3. **"Deploying a comma costs a release."** — Fixing a typo in a prompt requires a code deploy, a review, and a 40-minute pipeline, so nobody fixes small things. → *Need: decouple prompt changes from code deploys — fetch the prompt at runtime by name, promote new versions with a click.*
4. **"The playground and production are different animals."** — What the builder tried in a playground isn't what production runs (different model, temperature, message order). → *Need: one artifact that carries its content AND its configuration, runnable the same way everywhere.*
5. **"The bill doubled and nobody noticed."** — A prompt grew to 6,000 tokens through months of additions; every single call pays for that. → *Need: cost visibility while writing, and tools to shrink prompts safely (this is Keystone's compression bet).*

## A3. Anatomy of a prompt (the object, piece by piece)

- **Kind — text vs chat.** A **text prompt** is one block of instructions ("Rewrite this in a friendly tone: …") — used for one-shot transformations. A **chat prompt** is a structured conversation: a **system message** (the standing rules — who the assistant is), then alternating **user** and **assistant** messages. Assistants and agents need chat; utilities need text. Every serious tool models both, and the kind is fixed at creation (✓ Langfuse does exactly this).
- **Variables** — fill-in-the-blank slots, e.g. `{{customer_message}}`. The prompt is a *template*; at run time real values are injected. The magic UX detail: typing a variable should **automatically** declare it as a required input — the prompt defines its own form.
- **Placeholders** (chat only) — a slot that injects a *whole conversation* (e.g. `chat_history`), not one value. This is how an assistant remembers the dialogue so far.
- **References** — one prompt pulling in another by name (e.g. every prompt includes the shared `safety-guidelines`). Kills copy-paste drift: update the guidelines once, every prompt that references them updates. (✓ Langfuse calls this *composability*.)
- **Config / binding** — the settings around the words: which model, temperature (how "creative" the model is), max tokens (output length cap). Tools store this *with* the prompt version so "what we tested" = "what runs."
- **Output declaration** — what the prompt is supposed to produce: plain text, **JSON** (machine-readable structured data — needed when another program consumes the output), or (increasingly) images/files.
- **Version** — every saved change becomes a numbered, immutable snapshot (v1, v2, …). Immutable means v7 never changes after the fact — that's what makes history trustworthy.
- **Deployment pointer** (the concept with three names) — a named pointer that says *which version is live where*: `production` points at v12, `staging` at v14. Renaming the pointer target = deploying. Langfuse calls these **labels** ✓, Phoenix calls them **tags** ✓, our PRD calls them **version-tags**. Same idea everywhere: **promotion is moving a pointer, not copying text.**

## A4. The lifecycle — two loops at two speeds

**The minute loop (iteration):** write → try on an example → read the output → tweak → try again. A builder does this *dozens of times an hour*. Every extra click here is felt as pain. This is what playgrounds optimize.

**The week loop (governance):** candidate ready → test on many real inputs → compare against the current champion → promote to staging → watch it → promote to production → monitor cost/quality → eventually optimize or retire. This is what ops tools optimize.

**The core tension of the whole domain: those two loops fight.** Governance adds steps; speed hates steps. The market's two camps each serve one loop and neglect the other. **Keystone's thesis (the PRD's central bet): if every save is *automatically* a version, and promotion is just moving a pointer, then governance stops being extra work — it becomes a byproduct of iterating fast.** The builder never "does versioning"; it happens. That single idea should be visible in almost every screen we design.

## A5. Who touches prompts

- **The builder** (our persona): technical, lives in the minute loop, ships through the week loop. Design center.
- **The domain expert / PM** (secondary, growing): the person who knows what "good tone" is but doesn't code. Langfuse's marquee pitch ✓ is that prompts managed in a UI let non-engineers edit without touching code. Implication for us: reading and *safely* editing must not require engineer instincts — guardrails (protected environments, type-to-confirm deletes) carry the safety.
- **The operator/admin**: cares about who can promote to production, audit trails, and spend. Mostly lives in Manage.

---

# PART B — CURRENT STATE

## B1. Langfuse (Zaki's benchmark — study this one deepest)

Langfuse is an open-source **LLM engineering platform**: tracing/observability at its core, with prompt management, evaluation, and datasets around it. Relevant to us twice over: it's the reference UI *and* Keystone already routes its run traces into Langfuse internally.

**Their prompt object model (✓ verified today):**
- A prompt has a **name**, a **type** (`text` or `chat` — fixed at creation), the content, and a **config** object (arbitrary JSON — typically model + parameters — stored per version).
- Chat prompts are arrays of `{role, content}` messages; both types use **mustache-style `{{variable}}`** syntax. Chat prompts also support **message placeholders** (inject an array of messages at run time).
- **Versions are automatic**: saving a prompt with an existing name creates the next version. There is a **version diff view** showing changes over time.
- **Labels are deployment pointers**: `production` is special (it's what the SDK serves by default), `latest` always points to the newest version, and teams create **custom labels** for "environments (staging, production), tenants (tenant-1, tenant-2), or experiments (prod-a, prod-b)". **Rollback = move the `production` label back to an older version in the UI.**
- **Protected labels** (paid): admins can lock `production` so only owners/admins can move it — governance via role, not process.
- **Composability**: a text prompt can embed another with `@@@langfusePrompt:name=PromptName|version=1@@@` (pinned) or `|label=production@@@` (follows the pointer). The UI has an "Add prompt reference" button. **Text prompts only.**
- **Consumption**: apps fetch by name (+ optional label/version) via SDK — `get_prompt("movie-critic", label="staging")` — with aggressive **client-side caching** so the network is not in the hot path. This "fetch by name, cache hard" pattern is *the* industry answer to "prompts without redeploys."
- **Loop closure**: prompts link to **traces** (records of real production calls), so each prompt version shows its real-world metrics — generations, latency, cost. This cause-to-effect link (this version → these outcomes) is Langfuse's strongest governance feature.

**UI flow (to verify with screenshots in Phase 1 ○):** a prompts table (name, versions, labels, last updated) → prompt detail with a **version list rail** on the left (each version showing its labels), content view, config, diff between versions, "test in playground" handoff, and metrics tabs. The playground is a separate surface: model picker + parameters on the right, prompt + variables in the middle, output below.

**What Langfuse does NOT have (our openings, per the huddle):** no prompt compression / token-economics coaching; local models only via generic OpenAI-compatible endpoints (not a first-class local story); the playground and prompt library are still two rooms rather than one continuous surface — the builder walks between them.

## B2. Arize Phoenix (the second benchmark)

Phoenix is Arize's open-source **observability + evaluation** platform (built around **OpenInference**, their tracing standard — relevant because Keystone's docs also speak OpenInference).

**Their prompt model (✓ verified today):**
- Prompts are versioned objects; the **Prompt Playground** is the center of gravity — multi-model (side-by-side model comparison built into the playground), parameter experimentation, dataset-driven runs, tool/schema support.
- **Tags are deployment pointers**: built-in `production`, `staging`, `development`, plus **custom tags** (e.g. `v0-release`), each tag pointing at exactly **one** version per prompt; retrieval by tag via SDK (`prompts.get(prompt_identifier=…, tag="production")`). Tag names are constrained identifiers (lowercase, hyphens/underscores).
- **Span replay** (their signature move): take a *real production call* captured in tracing and replay it in the playground with your edited prompt — "would my fix have helped on this actual failure?" This is debugging-from-evidence, the strongest version of PRD JTBD-7.
- Experiments: run prompt variants over datasets with evaluators, tracked over time.

**What Phoenix does NOT have:** compression (nothing); its prompt *library/governance* surface is thinner than Langfuse's (the playground is the hero, the library is secondary); no composability/references.

## B3. The wider field in one paragraph (○ knowledge-based, spot-check in Phase 1)

**LangSmith** (LangChain): prompts with commit-style history + playground, tightly coupled to the LangChain ecosystem. **Braintrust**: evaluation-first — its hero screen is the side-by-side experiment diff; prompts serve evals. **Humanloop** (sunset in 2025 after the Anthropic acqui-hire — a cautionary tale that this market consolidates): pioneered directories + environment deployments + non-engineer editing. **PromptLayer**: registry + visual editor, pitched at letting PMs own prompts. **Vellum**: workflow builder with prompt sandbox + comparisons. **Agenta**: open-source playground + evals with side-by-side. **OpenAI Playground / Anthropic Console**: vendor workbenches — excellent minute-loop, zero governance, single-vendor by design. The convergent pattern set everyone shares: *named versioned prompts · text+chat kinds · {{variables}} · deployment pointers · playground · fetch-by-name SDK · link-to-traces · dataset runs*. Nobody differentiates on these anymore — they're table stakes. Differentiation now lives in: eval depth (Braintrust), replay-from-production (Phoenix), ecosystem (LangSmith), openness (Langfuse), and — Keystone's chosen ground — **cost engineering + model freedom**.

## B4. Keystone today — what the repo actually ships (audited 2 Jul)

The honest baseline (full audit in `00-EXECUTION-PLAN.md` §2):

- A **Prompt** is `{name, description, template, version}` — one flat text template, project-scoped. No kinds, no labels, no deployment pointers, no config, no references.
- **Versioning is real and automatic**: every template edit snapshots the old version and bumps the integer. **Restore is non-destructive** (restoring v2 creates a new v5 whose content = v2) — genuinely good governance, keep it.
- **Variables are f-string style `{variable}`** (single braces) — *not* mustache. `{{secret:KEY}}` injects a secret (a stored credential/value) at run time, encrypted, never shown, scrubbed from traces.
- **Prompts don't run.** They are pinned into **flows** (the canvas graphs); the flow runs. The flow's Prompt node records `prompt_id + pinned version`, offers "Upgrade to vN" when the library moves ahead, and the library shows "used in N flows."
- **Model config lives on the model node**, not the prompt: provider (OpenAI, Anthropic, Mistral, HuggingFace, **Ollama**, any OpenAI-compatible endpoint — all real today), model, api_key (via secrets), base_url, temperature, max_tokens, top_p, stream.
- Metrics: **token counts and wall time only — no dollar costs yet** (no price table in the codebase).
- Current UI: a modest list + side-sheet editor (CodeMirror with `{var}` highlighting and autocomplete), version history with restore. Functional, not yet a "surface."

**Read this as: the foundation (versioning, secrets, provider breadth, flow pinning) is real; everything that makes prompt management a *product* (kinds, pointers, references, runs, compare, compression, cost) is still to be designed — by us.**

## B5. Keystone target — the PRD, decoded against the market

Now you can see the PRD for what it is: **a deliberate remix of Langfuse and Phoenix, plus two bets neither has.** The genome:

| PRD concept | Comes from | The remix |
|---|---|---|
| Chat/text kinds, fixed at creation | Langfuse ✓ | adopted as-is |
| `{{variables}}`, placeholders | Langfuse ✓ | adopted (⚠ repo ships `{var}` — GitHub question #1) |
| **Version-tags** (`production/staging/dev/draft`) | Phoenix's *tags* (fixed enum + custom) ✓ + Langfuse's *labels* (pointer semantics, production-by-default) ✓ | merged, renamed to avoid… |
| **Labels** = free-form `key:value` metadata | …the collision: Langfuse "labels" *deploy*; PRD "labels" *organize*. The PRD split one overloaded word into two clean concepts — genuinely better than both benchmarks | new, with 3-scope governance (system/org/project, curated/ad-hoc) |
| References as chips, pinned by version *or* tag | Langfuse composability ✓ (`@@@langfusePrompt:name=…\|version=…@@@` → PRD's `@@@prompt:name=…@@@`) | adopted + extended to chat prompts, shown as chips not raw tags |
| Run/batch/compare/inspect in-surface | Phoenix playground energy ✓ | folded *into* the prompt surface instead of a separate room — the "never leave" bet |
| Inspect with request/response + timeline | Phoenix span replay ✓ / Langfuse trace link ✓ | adopted; replay-from-production is the natural future extension |
| **Compression (LLMLingua) + before/after economics** | **neither** — differentiator #1 | Microsoft's LLMLingua family: algorithms that shrink a prompt (drop low-information words) while protecting meaning — variables and references are never touched. Ship with visible token/cost savings |
| **Builder-owned credentials + local models first-class** | **neither** (both treat BYO-key as config, not identity) — differentiator #2 | Keystone holds no keys; secrets referenced by name; Ollama/self-hosted as a first-class path — already real in the repo |
| One surface, three modes (Edit·Run·Manage) | the anti-pattern both benchmarks share (library vs playground = two rooms) | the structural innovation to prove |

---

# PART C — FUTURE ASSUMPTIONS → DESIGN DECISIONS

*This is the ledger that turns foresight into pixels. Each assumption: what we believe, how confident we are, and the design decision it drives — with where it lands in the prototype. When Zaki challenges a screen, the answer traces back to a row here (or the row dies and the screen changes).*

**A-1 · Token cost stays a first-order concern.** Even as per-token prices fall, usage grows faster (longer contexts, agent loops, retries) — the *bill* keeps rising. **Confidence: high** (huddle: compression is THE differentiator; founder's "cost coach" thesis; repo already counts tokens).
→ **Decision:** cost is ambient, not a report you visit. Live amber token/cost readout *while typing* (Token budget card), per-run cost on every result row, before/after economics as the hero of the Compress view, and coach-style hints ("switch to gpt-4.1-mini → save ~85%"). Amber = cost, everywhere, only.

**A-2 · Models churn faster than prompts.** New model generations land every few months; teams swap models without rewriting products. **Confidence: high.**
→ **Decision:** a prompt is **model-agnostic**; the model is a *binding* chosen at run time (with a saved per-version *default*), never hardcoded in the artifact. Compare mode lets columns differ by **model**, not just version — "same prompt, three models, same inputs" is a first-class comparison. Model names in pickers come from a registry (repo reality), so new models are data, not redesign.

**A-3 · Local & self-hosted models become normal**, driven by privacy, cost, and open-weight quality (Ollama/vLLM are already real in the repo). **Confidence: medium-high** (huddle: "local models planned"; repo: already implemented).
→ **Decision:** the runner picker treats **Hosted** and **Local/self-hosted** as sibling groups, not an "advanced" afterthought: pick local → endpoint URL field appears, credential optional. Empty states never assume a cloud key is the only path.

**A-4 · Trust = the builder owns keys and data.** Post-2024, teams are allergic to platforms holding their provider credentials. **Confidence: high** (PRD principle #5; repo's write-once encrypted secrets).
→ **Decision:** raw keys are typed exactly once (in Secrets); everywhere else credentials appear only as **names** to select. Run buttons stay disabled until a credential/endpoint is bound, with honest copy ("Choose a model and add an API key to run"). No "demo key" fiction anywhere in the mock.

**A-5 · Multimodal output becomes ordinary.** Prompts increasingly produce images, files, audio — not just text (huddle: "image/HTML outputs, file inputs require future consideration"). **Confidence: medium** (timing uncertain, direction certain).
→ **Decision:** output is a **declared port with a type** (text/JSON/image/file, multiple allowed), not an assumption — so new types are one more chip, not a redesign. Results render by type (text inline, JSON formatted, image thumbnail, file chip); binaries are handled as *references* (a link to stored bytes), never pasted into the record. Attachments row in quick-run, enabled only when the model supports it.

**A-6 · Structured output wins for machine-consumed prompts.** When another program reads the output, JSON-with-schema beats prose; providers now enforce schemas natively. **Confidence: high.**
→ **Decision:** JSON is a first-class output type from day one (declared in the Output card, pretty-rendered in results). A future `schema` attachment on the JSON output type is anticipated in the data model — designed-for, not built.

**A-7 · Prompts become composable modules.** Shared tone/safety/compliance blocks get factored out and referenced, like functions (Langfuse validates demand ✓). **Confidence: medium-high.**
→ **Decision:** references are **chips** (visual objects showing resolution state — green found / red missing), not raw text tags; insertable via ⌘K; pinnable by version (stable) or tag (moves with promotion — powerful and dangerous, so the chip shows which). References are always protected during compression.

**A-8 · Prompt writing gets automated; curation doesn't.** DSPy-style optimizers and meta-prompting will *generate* candidate prompts; humans shift to defining success and choosing winners. **Confidence: medium** (huddle: DSPy explicitly named; PRD defers its screens to "Learn").
→ **Decision:** the surface is architected as a **candidate-judgment machine** — versions are cheap, Compare is central, "winner" is a first-class verb — so machine-generated candidates slot in as just more versions. Today: one "Optimize with DSPy" entry point button. No more.

**A-9 · Evaluation becomes a promotion gate.** Mature teams won't move `production` on vibes; version promotion will demand eval evidence, like CI checks before a merge. **Confidence: medium-high** (Braintrust/Phoenix trajectory; Keystone has eval harness + Langfuse scores already).
→ **Decision:** Compare hands off to "Run as Experiment" (the Evaluate feature's door); the Promote dialog leaves room to grow a "checks" summary (eval status per candidate) without relayout. Promotion is explicit, audited, and role-guardable (Langfuse's protected labels ✓ point the way — maps to our Owner/Editor roles).

**A-10 · Provider-side prompt caching rewards stable prefixes.** Anthropic/OpenAI charge much less for the unchanged leading part of a prompt across calls — so structure (static rules first, volatile variables last) now has a *price*. **Confidence: high** (shipped provider behavior).
→ **Decision:** the editor's segment structure makes prefix/suffix visually legible; the Token budget card can later annotate "cached vs fresh" tokens (the PRD's B.7 metrics already carry `tokens cached`). A coach hint ("move volatile content later to exploit caching") is a natural future line — designed-for.

**A-11 · The "prompt" object grows toward the "agent skill".** Task prompts with tool definitions, files, and sub-steps (huddle: "task prompts… emerging patterns"; repo's D121 sketches Skills-modeled-on-Prompt). **Confidence: medium-low** (direction likely, shape unclear).
→ **Decision:** don't design for it — design so it doesn't break us: `kind` is an open enum (adding `task` later = one more badge + one more editor shape), the editor is already a *stack of typed blocks* (a tool block is just a new block type), and the list/version/promote machinery is kind-agnostic. This is why the PRD's "patterns designed to generalize" line matters.

**A-12 · Non-engineers will edit prompts in the UI.** The PM/domain-expert editing story is a proven adoption driver (Langfuse's core pitch ✓). **Confidence: high.**
→ **Decision:** plain-language everywhere (no regex-speak in copy), destructive friction scaled to risk (type-name-to-delete; simple confirm to archive), protected environments via roles, and the *why* of every state visible (a disabled Run explains itself). The terminal skin may *look* engineer-y — the words must not be.

**A-13 · There is no standard prompt template format** — mustache `{{var}}` vs f-string `{var}` vs Jinja differ across tools; migration between them is real life. **Confidence: high** (our own PRD-vs-repo mismatch proves it).
→ **Decision:** the data model carries `template_format` explicitly (PRD B.1 already does); the editor *shows* which syntax is active rather than assuming the user knows; import/export keeps the format visible. Prototype teaches ONE syntax consistently (pending GitHub question #1) — mixed-syntax screens would be craft failure.

**A-14 · Prompt management consolidates into platforms.** Standalone prompt tools get absorbed (Humanloop's exit is the omen ○); prompt management survives as *a surface inside* observability/workflow platforms — exactly what Keystone is. **Confidence: medium-high.**
→ **Decision:** prompts are **one entity type among peers** (flows, datasets, secrets) sharing platform machinery — same versioning idiom as datasets (the repo already mirrors them), same shell, same ⌘K, same list/table discipline. We design patterns, not a one-off app — the Manage mode layout should feel reusable for any Keystone entity.

---

## C2. The talk track — ten questions you should be able to answer cold

1. **"What is prompt management in one sentence?"** — Treating prompts like source code: one home, automatic versions, safe testing, one-click deployment, and a record of what changed, why, and what it did in production.
2. **"Why not just keep prompts in the codebase?"** — Then every wording fix needs a code deploy, non-engineers can't contribute, and you can't compare or roll back independently of releases. Fetch-by-name + pointer promotion decouples the two.
3. **"What's the difference between a version and a version-tag?"** — A version is a frozen snapshot (v12, forever). A version-tag is a movable pointer naming which snapshot is live in an environment (`production → v12`). Saving makes versions; promoting moves pointers.
4. **"Why does Keystone split 'labels' from 'version-tags' when Langfuse uses one thing?"** — Langfuse overloads one word for two jobs: deployment (`production`) and organization (`tenant-1`). Overloaded concepts confuse governance ("did I just *deploy* by adding a label?"). We separated them: version-tags deploy, labels organize. Cleaner mental model — one of our quiet improvements over the benchmark.
5. **"Why is temperature not part of the prompt text?"** — It's part of the *binding* — how a model runs the prompt. The version stores a *default* binding so tests are reproducible, but the same prompt can run on different models/settings without becoming a different prompt (assumption A-2).
6. **"What's compression, and why trust it?"** — LLMLingua-family algorithms strip low-information words so the prompt costs less and runs faster with (near-)equal behavior. Trust is designed, not claimed: variables and references are visibly protected, removed words are shown struck-through before you commit, applying creates a *new version* (instantly revertible), and the before/after economics are shown honestly.
7. **"Chat prompt vs text prompt — when is which?"** — Chat = anything conversational or role-driven (assistants, agents): system rules + turns + history placeholder. Text = one-shot transformations (rewrite, classify, extract). Different editors, same lifecycle.
8. **"How do I know a prompt change actually helped?"** — Never from one example. Batch it over real inputs, compare candidate vs champion on the same data, inspect the failures (exact request/response), and only then promote. The surface makes that path the path of least resistance.
9. **"Where do API keys live?"** — In Secrets: typed once, encrypted, shown never. Everywhere else you pick a *name*. Keystone holds no platform keys — your providers, your keys, your data. Local models skip keys entirely (endpoint instead).
10. **"What do Langfuse and Phoenix do better than us today — honestly?"** — Langfuse: maturity of the trace-link (per-version production metrics) and team features (protected labels). Phoenix: span replay (re-run a real production failure against your edited prompt). Neither has compression, cost coaching, or our one-surface loop — that's the wedge. (Saying the honest part first is what makes the wedge credible.)

## C3. The jargon dictionary — every term you need, in plain language

*Organized by theme. Skim the "core objects" table first (that's the daily vocabulary), then use the rest as a lookup when a word flies past in a call. Format: what it means — and, where useful, why you'll hear it.*

### C3.1 The core objects (daily vocabulary — know these cold)

| Term | Plain meaning |
|---|---|
| Prompt | The written instructions given to an AI model |
| Text / chat prompt | One instruction block / a structured conversation with roles |
| System message | The standing rules of a chat prompt ("you are a support agent…") |
| Variable `{{x}}` | A fill-in-the-blank the prompt requires at run time |
| Placeholder | A slot injecting a whole conversation history (chat only) |
| Reference | One prompt embedding another by name (shown as a chip) |
| Template / template format | A prompt with blanks in it / which blank syntax is used (mustache `{{x}}` vs f-string `{x}`) |
| Version | A frozen numbered snapshot created on every save |
| Version-tag / label (Langfuse) / tag (Phoenix) | The movable pointer naming which version is live in an environment |
| Promote | Move that pointer up an environment (dev → staging → production) |
| Label (Keystone PRD) | Free-form `key:value` organizing metadata — never deploys anything |
| Binding | Provider + model + parameters + credential a run uses |
| Secret | A stored encrypted credential/value, referenced by name, never displayed |
| Run / trace | One execution of a prompt / its full recorded internals |
| Batch | Same prompt over many inputs at once |
| Compare | Candidates side-by-side on identical inputs |
| Inspect | Opening one result to see exactly what was sent and returned |
| Compression (LLMLingua) | Algorithmic shrinking of a prompt with protected variables/references |
| Evaluator / LLM-judge | A grader (rule or model) scoring outputs at scale |
| Dataset | A saved set of test inputs (± expected outputs) |

### C3.2 Model & inference basics (how the AI side works)

| Term | What it means — and why you'll hear it |
|---|---|
| LLM | Large language model — the AI that reads and writes text (Claude, GPT, Llama…) |
| Provider | The company/service hosting the model (Anthropic, OpenAI, Mistral…). One prompt, many possible providers |
| Inference | Actually running the model on an input. "Inference cost" = the cost of running, not building |
| Token | The unit models read and **bill** in — roughly ¾ of a word. Everything in this product is priced and measured in tokens; it's why the amber numbers exist |
| Tokenizer | The rule set that chops text into tokens. Different models count slightly differently — why token counts are "estimates" |
| Context window | The model's maximum reading length per call (prompt + history + output). The Token-budget bar shows % of this limit |
| Temperature | The "creativity dial" (0 = same answer every time, higher = more varied). Low for classification, higher for writing |
| top_p | A second randomness dial (capping how wide the word choice spreads). Usually leave default; shown under Advanced |
| max tokens | A cap on how long the *output* may be — a cost/safety brake |
| Sampling | The general word for how the model picks its next word (temperature and top_p tune it) |
| Completion | The model's output/answer (API-speak; "generation" means the same) |
| Roles (system/user/assistant) | Who is "speaking" in each chat message: the rules, the human, the AI |
| Streaming | Output arriving word-by-word as it's generated instead of all at once — why answers "type themselves" |
| Latency (p50/p95/p99) | Response time. p50 = typical; p95/p99 = the slow tail (1-in-20 / 1-in-100 worst). Ops people care about the tail |
| Hallucination | The model confidently making something up — the failure evals exist to catch |
| Grounding / RAG | Feeding the model real reference material (retrieved documents) so it answers from facts, not memory. RAG = retrieval-augmented generation |
| Multimodal | Handling more than text: images, audio, files, in or out |
| Open-weight / closed | Models you can download and run yourself (Llama via Ollama) vs API-only (Claude, GPT) |
| Fine-tuning | Retraining a model on your data — the heavyweight alternative to prompt iteration (prompting is cheaper, faster, reversible; usually try prompts first) |
| Embedding | Turning text into numbers so similarity can be computed — powers search/RAG; you'll hear it around datasets |

### C3.3 Prompt-craft terms (how prompts are written)

| Term | What it means |
|---|---|
| Prompt engineering | The craft of writing/structuring prompts to get reliable behavior |
| Zero-shot / few-shot | Asking with no examples / including worked examples in the prompt (the repo's `FewShotPrompt` component = prefix + examples + suffix) |
| Chain-of-thought | Asking the model to reason step-by-step before answering — better accuracy, more tokens (a cost trade-off) |
| Persona | The "you are X" identity given in the system message |
| Delimiters | Markers (```, ###, XML tags) fencing off parts of a prompt so instructions and data don't blur |
| Guardrails | Constraints on output (format, refusals, banned content) — the repo's `ValidatorPrompt` adds format guardrails |
| Prompt injection | An attack: malicious text in the *input* trying to override your instructions ("ignore previous instructions and…"). Why system rules and user content are kept structurally separate |
| Jailbreak | Tricking a model past its safety rules — cousin of injection |
| Meta-prompting | Using an LLM to write/improve prompts for another LLM |
| DSPy | A framework that *automatically* optimizes prompts against a success metric — the "Optimize" button's engine (assumption A-8) |
| LLMLingua / LongLLMLingua / LLMLingua-2 | Microsoft's compression family: shrink a prompt by dropping low-information words. Long- variant is question-aware for long documents; -2 is the faster, task-agnostic revision. Our three method cards |
| Keep rate / target tokens | Compression dials: "keep ~60% of words" vs "get it under N tokens" |
| Prompt caching (provider-side) | Providers charge much less for the unchanged *leading* part of a prompt reused across calls — why stable content should sit first (assumption A-10) |

### C3.4 Versioning & governance (the management layer)

| Term | What it means |
|---|---|
| Immutable | Never edited after creation — a version is immutable, which is what makes history trustworthy |
| Diff | The highlighted difference between two versions (green added / red removed) |
| Rollback / restore | Going back to an older version. Keystone restores *non-destructively*: restoring v2 creates a fresh v5 with v2's content |
| Environment | A stage of realness: dev (building) → staging (team-testing) → production (live for users) |
| Deployment pointer | Our umbrella phrase for version-tag/label/tag — the movable "this version is live here" marker |
| Draft / active / archived | Lifecycle states: not yet real → in use → retired-but-kept. Delete is the only permanent one |
| Lineage | The recorded chain of what produced what ("this run used prompt v12 + model X") — the bill-of-materials of a run |
| Audit trail | The immutable log of who did what when — what makes "who changed the prompt?" answerable |
| RBAC | Role-based access control — permissions by role, not per person |
| Owner / Editor / Viewer | The typical role ladder: govern / change / look |
| Protected labels (Langfuse) | Locking `production` so only admins can move it — role-gated promotion |
| Promotion gate | A required check (e.g. passing evals) before a version may be promoted (assumption A-9) |

### C3.5 Running, tracing & observability

| Term | What it means |
|---|---|
| Observability | Being able to see what your system actually did — logs, traces, metrics — instead of guessing |
| Span | One timed step inside a trace (one model call, one tool call). A trace is a tree of spans |
| Generation | Observability-speak for one LLM call span (Langfuse's term) |
| Session | A group of related runs — e.g. one user's whole conversation |
| OpenTelemetry (OTel) | The industry-standard plumbing for emitting traces — vendor-neutral |
| OpenInference | Phoenix/Arize's open standard for describing *LLM* traces specifically (what was the prompt, model, tokens) |
| Instrumentation | The hooks in code that emit those traces |
| Waterfall / timeline | The Gantt-style view of a trace: which step ran when, for how long — our Inspect view's spine |
| Request / response (raw) | The exact JSON sent to and received from the provider — the ground truth Inspect shows |
| Token breakdown | input / output / cached token counts per run — the amber arithmetic |
| Span replay (Phoenix) | Re-running a *real captured production call* against your edited prompt — "would my fix have helped?" |
| SSE | Server-sent events — the web mechanism that streams run progress live into the UI (how Keystone streams runs) |
| Cost per run | Tokens × provider price. The repo counts tokens but has no price table yet — we mock the $ figure |

### C3.6 Testing & evaluation

| Term | What it means |
|---|---|
| Ground truth / expected output | The known-correct answer a test input should produce |
| Experiment | A recorded, repeatable comparison of variants over a dataset (Langfuse/Phoenix both use this word) |
| A/B test | Splitting real traffic between two versions and measuring which wins live |
| Champion / challenger | The current live version vs the candidate trying to beat it — Compare mode's real story |
| Metric (exact match / contains / regex / json_valid) | Rule-based graders that exist in Keystone's eval harness today |
| Score | The grade an evaluator assigns (number, label, or pass/fail) |
| Regression | The new version being *worse* on something the old one handled — the thing eval gates catch |
| Golden set | A small, curated dataset of must-pass examples |
| Human-in-the-loop / annotation | People grading outputs by hand — the craft the research notes call "annotation as craft" |

### C3.7 Credentials, deployment & integration

| Term | What it means |
|---|---|
| API key | The password-like string a provider gives you; whoever holds it spends your money — hence all the ceremony |
| BYO keys | Bring-your-own: the platform uses *your* provider accounts, holds nothing itself (differentiator #2) |
| Credential reference | Pointing at a secret *by name* instead of pasting the key — the only pattern our UI allows |
| Write-once / masked | You can set a secret but never read it back; shown as `sk-…••••` |
| Encryption at rest | Stored scrambled, so a stolen database doesn't leak keys |
| Endpoint / base URL | The web address a model listens at — the whole config a local model needs (`http://localhost:11434`) |
| Self-hosted / on-prem | Running models on your own machines — privacy + cost control (assumption A-3) |
| Ollama / vLLM | The two popular self-hosting engines: Ollama = run models on your laptop easily; vLLM = serve them fast on servers |
| OpenAI-compatible | Speaking OpenAI's API shape, the de-facto lingua franca — one adapter covers many providers |
| SDK | The code library apps use to talk to a platform (`get_prompt("triage", label="production")`) |
| Fetch-by-name + client-side caching | The consumption pattern: app asks for a prompt by name at run time, keeps a local copy so it's fast — how prompts update without redeploys |
| Webhook | "Call this URL when something happens" — e.g. notify Slack when production is promoted |
| MCP | Model Context Protocol — the standard for exposing tools/data to AI agents (Zaki uses GitHub MCP; Keystone flows can be exposed via MCP) |
| Structured output / JSON schema | Forcing output into a machine-readable shape with a declared contract (assumption A-6) |
| Function / tool calling | The model responding "call this function with these arguments" instead of prose — where prompts meet agents (assumption A-11) |

### C3.8 Keystone-specific vocabulary

| Term | What it means here |
|---|---|
| Flow | A visual pipeline on the canvas (nodes wired together) — the thing that actually runs today |
| Node / component | One step in a flow (a Prompt node, a Model node…) |
| Port / handle | A node's typed connection point — a prompt's `{variables}` become input ports automatically |
| Pinning | A flow's Prompt node locking onto a library prompt at a specific version (`pinned v3`), with "Upgrade to vN" offered when the library moves on |
| Save to library | Promoting an inline, one-off prompt into the shared library |
| Org → Project | The scope ladder: secrets can be org-wide; prompts/flows live in a project |
| Builder / Consumer planes | The two apps: the maker's console (us) vs the auto-generated end-user app |
| Lens | The Terminal editor's mode tabs (Build/Results/Test/Debug/Share) — our Edit/Run/Manage follows the same pattern |
| Cost coach | The design stance: don't just meter spend, *advise* ("switch model → save 85%") — principle candidate #6 |
| `{{secret:KEY}}` | Keystone's in-template token that injects a secret at run time (real, shipped) |
| keystone.tty | The Terminal prototype's brand mark — `.tty` is old Unix-speak for a terminal, the aesthetic's namesake |

### C3.9 Design & handoff terms (what you'll hear from Zaki/engineers about the build)

| Term | What it means |
|---|---|
| shadcn/ui | The React component library Keystone's real frontend uses — copy-in components you own and restyle |
| New York style | shadcn's compact preset (smaller radii, tighter spacing) — our structural skeleton |
| Design tokens | The named values (colors, sizes) a UI is built from — our `--bg`, `--ink`, `--accent` variables |
| Primitive | A base component (Table, Dialog, Tabs) that screens compose — every element in our prototype names its primitive for handoff |
| Tailwind | The CSS utility framework the real app uses (`class="p-4 border"`); our prototype uses plain CSS but maps to it |
| CodeMirror | The embeddable code editor the real app uses for the template field ({var} highlighting, autocomplete) |
| React Flow | The library behind the flow canvas |
| Hash routing | Faking multiple pages in one HTML file via `#/prompts/triage` URLs — how our prototype navigates |
| Mock / fidelity | Fake-but-honest data and interactions / how "real" a prototype looks and behaves. Ours: high fidelity, zero backend |
| Empty / loading / error / denied states | The four non-happy conditions every screen must design, per the Keystone spec discipline |
| Progressive disclosure | Showing the simple thing first, revealing power on demand (the runner picker: model → then credential → then advanced) |
| Cognitive load | How much the user must hold in their head; the currency Zaki's "earn every component" rule spends |

## C4. Source ledger

- ✓ **Verified today (2 Jul 2026):** Langfuse docs — prompt management overview, get-started (object model, types, `{{var}}`), version control (labels, `latest`, custom labels, protected labels, diff view, rollback-by-label-move), composability (`@@@langfusePrompt:…@@@`, text-only, pin by version|label). Phoenix docs — prompt engineering overview (playground, span replay, prompts-in-code), tag-a-prompt (production/staging/development + custom, one version per tag, SDK get-by-tag).
- ✓ **Audited directly:** `keystone-main` repo (full digest in `00-EXECUTION-PLAN.md` §2); Terminal/Admin/BW prototypes (§3).
- ○ **Knowledge-based, re-verify in Phase 1:** Langfuse UI layout specifics (screenshot pass via Playwright), wider-market summaries (LangSmith, Braintrust, PromptLayer, Vellum, Agenta, Humanloop timeline).

*Next: Phase 1 research pass turns the ○ items into ✓, adds annotated screenshots, and feeds the final wording of the 7 principles (`02-DESIGN-PRINCIPLES.md`).*
