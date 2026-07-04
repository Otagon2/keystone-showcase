# Day 2 — Design decisions & why (post-audit iteration)

Every finding from the [5-lens audit](docs.html#day2-ux-audit) was implemented the same day. This is what we decided and *why* — the reasoning an engineer or designer would want, not just a changelog. (Each is also readable in-app: the **"Why this design?"** panel now tells the story of the screen you’re on.)

---

## Structure & navigation

**Identity now propagates.** Screens load the real object by id — every prompt is itself, every experiment shows its own scorecard (#5 is Ticket Triage at 91%, not GST #7). *Why:* a list that lies about what it links to is worse than no list; the single most corrosive thing we could leave in.

**The loop ribbon became a contextual trail.** Its links now carry the object you’re working on (`GST FAQ → GST QnA v13 → Q&A correctness → Exp #7 → gst-003`) instead of hard-wiring to the sample. *Why:* a breadcrumb-shaped chain that teleports you to a different object trains users to distrust it. (Unanimous audit finding.)

**The core loop is keyboard-operable.** Table rows are real links with visible focus rings; we added a skip-to-content link, an `aria-live` region, `prefers-reduced-motion` handling, and bumped the focus ring 1px → 2px. *Why:* the row is the primary control — it must work without a mouse (WCAG 2.1.1, Level A).

**A real ⌘K.** It’s now a fuzzy index over every prompt, experiment, trace, dataset and evaluator (was a hardcoded GST-only stub). *Why:* a power user’s muscle memory is "⌘K → type → Enter"; that path has to actually work.

---

## Making evaluation real

**Evaluators and Datasets are real libraries** (list → detail), with a template gallery for new evaluators. *Why:* "reusable evaluator" is our strongest bet, but it was first-class in the rail and second-class in the model — you couldn’t see a second one.

**Experiments hold multiple scorers → a per-metric scorecard.** An experiment can carry an LLM judge *and* a code assertion; results show a column per metric. *Why:* real suites lean on cheap deterministic checks (JSON-valid, exact-match) alongside a judge.

**Results you can act on.** The scorecard now has a **per-topic breakdown** (the metadata was already imported — we just weren’t using it), a score histogram, per-metric tiles, and a small-N confidence note. *Why:* after "83%", the first question is *"which slice is failing?"* — an aggregate hides it, and "+4pp on 12 rows" is inside the noise.

**Compare is one unified N-way view** (this run vs the baseline + others), with per-row deltas and a click straight from a regressed row to its trace. *Why:* "which rows regressed between v11, v12, v13?" was impossible across two half-built compare UIs.

**Production observability (Logs).** A new pillar: sampled live traffic scored online, with monitors on pass-rate / latency / error-rate. *Why:* the audit’s biggest gap — "Observe" only meant offline experiment traces. Curated datasets tell you if a change is *good*; production tells you if it *stays* good.

**Human review on traces.** You can overrule a judge with a thumbs + a note, tracked as judge-vs-human agreement. *Why:* an audience that distrusts LLM judges needs a calibration story, not blind faith.

**Eval-gated promotion.** "Promote to production" now shows the version’s pass-rate and only unlocks when it clears the bar. *Why:* the whole reason an eval tool exists is to stop a regression reaching production.

---

## The 7 original decisions, revisited

- **Token-spend, never $** → kept as the default (it’s model-agnostic and the unit the Compress feature is measured in), but we split **task vs judge tokens** everywhere so cost is attributed honestly, and we’d expose an optional $ view where a budget owner needs it.
- **Single-brace `{variable}`** → kept (matches the shipped parser); needs a documented escape for literal JSON braces.
- **Evaluator = first-class object** → doubled down, now with a library + version pinning so the reuse is real.
- **Summary-first** → kept and deepened (slices, histogram, worst rows).
- **Dark-leads** → kept; fixed the self-inflicted damage — the opacity-dimmed greys and the under-tuned light theme now pass AA.
- **Status = icon + colour** → kept, and extended to the distribution/pass-rate bars that were previously colour-only.
- **Loop ribbon** → reworked from a redundant, misleading chain into a contextual trail (see above).

---

## Copy & onboarding

- **Onboarding** is now a guided *tour of the live sample* ("See the dataset → Open the prompt → …"), not a fake "create your first…" that contradicted a populated app. "What is an evaluation loop?" opens a real explainer; key jargon has hover definitions.
- **Product copy** no longer names competitors or internal roadmap terms ("Braintrust/Arize", "LLMLingua", "fast-follow") — that positioning lives only in the internal design-notes panel.
- **One canonical dataset size** (was shown as 6 / 8 / 12 on different screens).

---

*Not everything is production-grade — several roadmap primitives (production logs, CI gating, human-review queues, SDK/export) are built at prototype fidelity to prove the shape, and are flagged for a scope conversation before real wiring.*
