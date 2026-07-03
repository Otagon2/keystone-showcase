# Keystone Prompt Management — Execution Plan

*Written 2 Jul 2026, after the huddle with Zaki (8:03–8:45 AM CDT). This is the master plan for the prompt-management engagement: what we build, in what order, against which sources of truth, and why. Every other deliverable in this folder hangs off this file.*

---

## 0. The one-paragraph version

We are designing **prompt management for Keystone** — the surface where a builder writes, tries, compares, versions, and ships prompts without leaving one screen. We build it as a **standalone HTML prototype** (no backend, mock data), visually aligned with the **Terminal UI** (the latest design iteration) on the **shadcn New York** component skeleton, for the **builder persona**. Before pixels: 1–2 days of Langfuse + Arize Phoenix research, then a **7-principles document** (pressure-tested, per Zaki's method), a **domain guide** (so Amaan can talk about prompt management like an expert), and a **design-decisions log** (every choice + why). Differentiators to make visible in the UI: **prompt compression (LLMLingua) with before/after token economics** and **model flexibility on the builder's own credentials** — the two things Zaki says Langfuse and Phoenix don't have.

---

## 1. Inputs on the table — and which one wins when they disagree

| # | Source | What it is | Role |
|---|--------|-----------|------|
| 1 | **Huddle notes 7/2** (Zaki + Amaan) | The client's live intent | **Highest authority on scope & process** |
| 2 | **The PRD** ("Prompt Management — Product & Design Brief", the Claude artifact Zaki shared) | Full product spec: thesis, 12 jobs-to-be-done, data model, screen appendices A–J | **The design contract** — what the prototype must show |
| 3 | **`keystone-main` repo** (latest drop) | The real codebase | **Today's reality** — vocabulary + what exists vs doesn't (see §2) |
| 4 | **`Keystone-FlowEditor-Terminal.html`** | Latest visual iteration | **The visual contract** — the prototype must look like a sibling of this file |
| 5 | **`Keystone-Admin-Prototype.html`** | Foundation prototype (shadcn New York, faithful) | Pattern library to borrow from (tables, hash-router, forms) — restyled into Terminal skin |
| 6 | **`Keystone-DesignSystem-BW.html`** | Design-system doctrine | The bridge rules (one reserved hue, value-not-color, dark+light parity) |
| 7 | **`research-notes/`** (ALIGNMENT.md, RESEARCH-DIGEST.md) | Prior Esberi/Keystone research | Reuse; don't repeat. Competitor walkthroughs already partially done |
| 8 | Presentation.pdf, FlowEditor-Studio/Prototype | Earlier iterations | Reference only |

**Precedence when sources conflict:**
- **Scope / features** → Huddle > PRD > repo. (The PRD describes the target product; the repo hasn't caught up — that's expected and fine for a mock prototype.)
- **Visual language** → Terminal file > Admin prototype > BW system > older docs. (DESIGN.md's amber/Newsreader "core.x" soul is the *marketing* skin — not this app surface. ALIGNMENT.md's "shadcn zinc + Inter" contract predates the Terminal iteration; the Terminal file supersedes it for look-and-feel while keeping the same structural discipline.)
- **Vocabulary** → PRD terms for target concepts; repo terms where they exist (so engineers recognize their own words). Flag mismatches in the decisions log.

**Transcription fixes from the huddle notes** (so nobody googles the wrong thing): "Harris Phoenix" = **Arize Phoenix**. "Clot Code" = **Claude Code**. "LLM Lingua" = **LLMLingua** (Microsoft's prompt-compression library).

---

## 2. Ground truth: what the repo has today vs what the PRD asks for

We audited `keystone-main` end-to-end. This matters for two reasons: (a) the prototype should reuse *real* vocabulary where it exists, and (b) **every gap below is a candidate GitHub question** (Zaki asked for PRD questions on GitHub, not Slack, so he can wire the backend with GitHub MCP).

### What EXISTS in `keystone-main` today (align with it)

| Concept | Reality in code |
|---|---|
| **Prompt entity** | `{ id, name, description, template, version }` — project-scoped, one flat text template |
| **Versioning** | Integer versions (v1, v2…), auto-snapshot on every template edit, **non-destructive restore** (restoring v2 creates v5) |
| **Variables** | **f-string style `{variable}`** — single word in single braces. *Not* mustache `{{var}}` (SPEC.md says "mustache" loosely; code says f-string. The PRD's `{{message}}` examples don't match the shipped regex — GitHub question #1) |
| **Secrets in templates** | `{{secret:KEY}}` tokens, resolved at run time, values encrypted, never displayed, redacted from traces |
| **Secrets model** | Org-level + project-level (project shadows org); free-text `environment` field ("base", "prod", "staging"…) |
| **Prompt ↔ flow binding** | A flow's Prompt node **pins** a library prompt (`prompt_id` + `pinned version`, template copied in); "Upgrade to vN" when the library moves ahead; "Save to library as…" promotes an inline prompt; `/usage` endpoint lists which flows use a prompt |
| **Providers** | OpenAI, Anthropic, Mistral, HuggingFace, **Ollama (local)**, any OpenAI-compatible endpoint — implemented **now**, not "later" |
| **Run params** | Live on the **model node** (provider, model, api_key→secret, base_url, temperature, max_tokens, top_p, stream) — not on the prompt |
| **Metrics** | Token counts (prompt/completion/total) + wall time. **No dollar-cost table yet** |
| **Tracing** | Routed to Langfuse per project (relevant: Keystone already *uses* Langfuse internally for traces) |
| **Prompt UI today** | List (Name + vN badge) → side Sheet with Name/Description/Template (CodeMirror, `{var}` highlighted, autocomplete after `{`), version history + restore, "Used in N flows" |

### What the PRD asks for that does NOT exist yet (we mock it — that's the point)

| PRD concept | Repo status | Note for the prototype |
|---|---|---|
| Chat vs text prompt **kinds** | ❌ No kind field; chat exists only as a canvas component (`ChatPromptTemplate`) whose `system_message` the library can't even store | Mock both kinds per PRD; flag the storage gap on GitHub |
| **Version-tags** (production/staging/dev/draft) + promote | ❌ Versions are bare integers | Mock per PRD Appendix B.4 |
| **Labels** (`key:value`, 3 scopes, curated/ad-hoc) | ❌ Prompts have no labels (flows have tags; prompts don't) | Mock per PRD B.4b |
| **References** between prompts (chips) | ❌ No prompt→prompt references; library is flat | Mock per PRD B.5 |
| **Compression** (LLMLingua ×3, token savings) | ❌ Zero occurrences in the codebase | Mock per PRD Appendix F — this is a **headline differentiator**, design it well |
| **Run a prompt directly** (single/batch/compare/inspect) | ❌ No run-a-prompt endpoint; only flows run | Mock per PRD Appendix E |
| **Connections** | ❌ Design-doc only (D070/D076); PRD also defers it — runner references Secrets directly | Follow the PRD: no Connections screen |
| Output types (JSON/image/file) | ❌ Only via a Validator component's format dropdown | Mock per PRD B.3b |
| Cost in dollars | ❌ Tokens only | Mock it (amber, Terminal cost idiom) |

**Read on this:** the PRD is a *target-state* spec — the huddle confirms Zaki wants exactly this direction (compression, model flexibility, mock-only prototype, "no backend wiring"). So the gaps are not contradictions; they're the roadmap. But they generate precise GitHub questions (§8).

---

## 3. The visual alignment contract (Terminal skin on a shadcn New York skeleton)

The user-facing rule: **the new prototype must look like a sibling of `Keystone-FlowEditor-Terminal.html`.** The structural rule: **every component maps 1:1 to a shadcn/ui New York primitive** (Table, Tabs, Dialog, Command, Badge, DropdownMenu, Sheet→dockpane…), so a developer can later rebuild it in real shadcn without redesign. The Terminal skin is a *rendering* of shadcn New York — same anatomy, terminal clothes.

### Tokens — copy verbatim from the Terminal file (dark, default)

```css
--bg:#0B0B0D; --bg-1:#101013; --bg-2:#15151A; --bg-3:#1D1E24;      /* grounds */
--line:#25262C; --line-2:#32343B; --line-3:#44464F;                 /* hairlines */
--ink:#E7EAEF; --ink-hi:#F6F8FB; --ink-2:#AEB3BC; --ink-3:#7C818B; --ink-4:#53565E;
--accent:#39C6CF;   /* CYAN — structure, focus, selection ONLY */
--cost:#F2C879;     /* AMBER — cost/tokens ONLY (the sacred rule) */
--ok:#3FCF6A;       /* GREEN — success + prompt glyphs */
--err:#F26D6D;      /* RED — errors */
--c-prompts:#8078B0;/* violet — the Prompts category marker dot */
--mono:ui-monospace,"JetBrains Mono","SF Mono",Menlo,Consolas,monospace;
```
Light theme = the full `body.light` "paper terminal" set from the same file (both themes are first-class; toggle via `◐`).

### The ten Terminal rules (non-negotiable)

1. **Monospace everywhere** (13px base, ligatures off). No Inter, no Newsreader on this surface.
2. **Radius 0.** Square corners on everything — buttons, chips, dialogs, inputs.
3. **Hairlines, not shadows.** 1px `--line/-2/-3` borders do all separation; the one allowed shadow is on floating dockpanes.
4. **Box-drawing frames** on pane headers: `┌ PROMPTS ┐`, `└─ PROPERTIES · name ─┘`, frame chars in cyan.
5. **Glyph icons, not SVG:** `{}` prompts · `✦` model · `▶ ■ ✓` run states · `⌕` search · `◈` brand · `❯ ➜` prompt markers.
6. **Uppercase micro-labels**, 10.5–11px, letter-spacing .08–.14em, for section headers and table headers.
7. **Hierarchy by value & weight, not hue.** Greyscale ramp does the work; the four signal colors are single-meaning (cyan/amber/green/red). Category violet only as a 7px dot.
8. **Amber = cost, and only cost.** Every token/cost readout is amber; nothing else is ever amber. (This is the "cost coach" differentiator made visible.)
9. **Status as glyphs/square chips**, not colored pills: `.tag` (1px border, square) for `v12`; square 6px dots for env badges.
10. **App shell:** 34px topbar (brand ◈ keystone.tty · breadcrumb · tool glyphs · amber costbox · inverted RUN button · lens tabs) + 200px left rail (`┌ EXPLORER ┐`, BUILD/OBSERVE/SHIP groups — **"Prompts" nav item already exists there with the `{}` glyph**) + 30px bottom cmdline (mode chip, selection, `➜ project ❯` prompt with blinking caret, stats).

### PRD component → Terminal idiom mapping (the crosswalk)

| PRD (Part 2) says | We render it as |
|---|---|
| shadcn `Table` (prompts list) | Admin `.table` pattern flattened: mono 10px uppercase headers, 1px row hairlines, hover `--bg-2` |
| `Badge` Env (green/amber/blue/grey) | Square `.tag` chip + 6px status square; env colors mapped to Terminal signals (production=green, staging=`--ink-2`, dev=cyan, draft=`--ink-4`) — amber stays reserved for cost, so staging is NOT amber (decision log entry) |
| Labels `key:value` pills | Square bordered chips, `--ink-3` |
| Mode `Tabs` (Edit·Run·Manage) | The `.lens` numbered underline strip (`01 edit` `02 run` `03 manage`) |
| Centered `Dialog` (all overlays) | The `.cmdk` frame recipe: scrim + 1px `--line-3` bordered box, no radius |
| `Command` ⌘K palette | Terminal `.cmdk` verbatim (groups, `❯` search row, `↑↓ ↵ esc` footer) |
| Model picker + credential select | `.ctrl` select rows (`✦ claude-sonnet-4-6 ▾`), secret picker listing Secret names only |
| Prompt body editor | `.ta` textarea idiom; `{variable}` highlighted, `{{secret:KEY}}` distinct; reference chips inline (green resolved / red missing) |
| Temperature slider | ASCII slider `━━━━█─────` |
| Token budget / cost | Amber `.costtab` idiom: big amber figure, `◇ tokens` `◷ latency` rows, sparkline |
| Quick-run / Inspect panels | `.dockpane` (`.lenspane` 352px right dock) |
| Compare columns | Stage split into 2–3 bordered columns, `✓ winner` inverted chip |
| Compression preview | `.code` block with struck-through removed words, protected `{vars}`/chips highlighted; amber before/after stats strip |
| Toasts | One-line cmdline messages (`✓ restored v11 as v13`) — the terminal answer to Sonner |

---

## 4. Deliverables & folder layout

All in `Esberi/Keystone-PromptManagement/`:

| File | What | Phase |
|---|---|---|
| `00-EXECUTION-PLAN.md` | This file | ✅ now |
| `01-RESEARCH-LANGFUSE-PHOENIX.md` | Competitor deep-dive digest + vocabulary crosswalk + screenshots | Phase 1 |
| `02-DESIGN-PRINCIPLES.md` | **The 7 principles**, pressure-tested, with why + how-applied + what-they-rule-out | Phase 2 |
| `03-DOMAIN-GUIDE.md` | "Become a domain expert" doc: prompt management 360° | Phase 2–3 |
| `04-DESIGN-DECISIONS.md` | Running log: every decision, options considered, choice, why, which principle it serves | starts Phase 2, lives forever |
| `05-GITHUB-QUESTIONS.md` | Drafted questions → posted to GitHub issues | Phase 1–2 |
| `Keystone-PromptManagement-Prototype.html` | The standalone prototype (single file, no backend) | Phase 3–4 |
| *(claude.ai Artifact)* | Same prototype published as an Artifact for Zaki to click through | Phase 4 |

---

## 5. Phase plan

### Phase 0 — Plan sign-off *(today, 2 Jul)*
This document. Confirm with Zaki at the next sync: scope = prompt management only, mock-only, Terminal-aligned, branch `prompt-management-v1`.

### Phase 1 — Research: Langfuse + Arize Phoenix *(2–3 Jul; share findings 4 Jul — the "day after tomorrow" commitment)*

Zaki's instruction: *understand existing systems' vocabulary and flows* — heavy emphasis on **Langfuse UI and how the whole thing works** (note: Keystone already pipes traces into Langfuse, so its vocabulary matters doubly).

**Langfuse — study list (the deep one):**
- Prompt object model: name, prompt content (text vs chat), config JSON, **labels** (`production`, `latest`, custom — Langfuse's labels are what our PRD calls *version-tags*; their naming collision with our "labels" is exactly the kind of vocabulary trap to document), versions, protected labels.
- The prompt detail UI: version list rail, diff view, playground handoff, "used in traces" linkage, metrics per version (generations, latency, cost).
- Compile/consumption model: `{{mustache}}` variables, SDK `get_prompt(name, label)`, caching, rollback story.
- Playground: model + params panel, variable filling, tool/schema support.
- Experiments/datasets tie-in and the eval loop.
- What's *missing* (our differentiators): no compression, weaker local-model story.
- Method: docs + live app (Playwright MCP captures of demo.langfuse.com, board section 08 workflow), plus what RESEARCH-DIGEST.md already holds.

**Arize Phoenix — study list (the lighter one):**
- Prompt management (prompts as versioned objects, playground with datasets, prompt diffs), OpenInference tracing vocabulary, evals.
- Same lens: object model, screens, vocabulary, gaps.

**Output (`01-…md`):** side-by-side vocabulary crosswalk (Langfuse ↔ Phoenix ↔ Keystone PRD ↔ keystone-main code), annotated screenshots, 5–10 "patterns worth stealing", 5 "traps to avoid", and the differentiation statement (compression + token economics + credential ownership + local models).

### Phase 2 — The 7 design principles + domain guide *(3–4 Jul)*

**Honest tension to resolve with Zaki:** the huddle says **3–5 principles, not 15**; the ask here is **7**. Strategy: draft **7**, structured as **5 core + 2 prompt-management-specific**, pressure-test each per Zaki's method (a principle must be able to *lose* an argument — each one lists a real decision it forced and a redundancy it removed, e.g. the sort-button-vs-sortable-columns example). If any principle can't cite a decision it changed, it merges or dies — and we present the survivors (7 if all earn their place, 5 if not). That *is* the pressure-test, demonstrated.

**Candidate principles to draft from** (seeded by the PRD's five + huddle + Terminal ethos — final wording after Phase 1 research):
1. **The loop is sacred** — write → try → inspect → fix → ship; common path = one click.
2. **Governed by default, not by chore** — every save is already a version; promotion moves a pointer.
3. **One way to do a thing** — mode tabs / whole-object toolbar / item ⋯ menu / centered dialog. One rule everywhere.
4. **Show, don't hide, what happened** — every run inspectable to the raw request; no guessing.
5. **The builder owns keys and data** — secrets referenced never typed; local models first-class.
6. **Cost is a first-class signal** *(prompt-specific)* — amber token economics live while you type, before/after on compression; coach, not meter.
7. **Earn every component** *(prompt-specific, the craft principle)* — each control must justify itself against cognitive load; kill redundancy; value-and-weight before color.

**Output:** `02-DESIGN-PRINCIPLES.md` — each principle: one-line statement → why (grounded in research/PRD/huddle) → how it applies to prompt management specifically (Zaki: "Keystone context, not generic") → 2–3 concrete decisions it drives in our screens → what it forbids.

**In parallel, `03-DOMAIN-GUIDE.md`** (the "make Amaan a domain expert" doc):
1. What prompt management is and why it exists (prompts = source code of AI products; the speed-vs-governance tension).
2. The lifecycle: author → template variables → try → batch → compare → version → tag/promote → observe → optimize.
3. Concept glossary with plain-language definitions (prompt kinds, variables vs placeholders vs references, versions vs version-tags vs labels, runs/traces, secrets/credentials, compression, evaluators) — each mapped to what Langfuse/Phoenix/keystone-main call it.
4. How the market does it (from Phase 1) and where Keystone diverges + why.
5. Emerging patterns Zaki flagged for the future (task prompts, image/HTML outputs, file inputs, Ollama/multimodal) — designed-for but not built.
6. A "talk track": 10 crisp Q&As Amaan can answer in a review (e.g. "why not put temperature on the prompt?" → "run params bind at run time; the version stores only a *default* binding").

### Phase 3 — Prototype architecture + P0 build *(4–6 Jul)*

**Tech shape (decided):**
- **One self-contained HTML file** (no framework, no build, no external requests — Artifact-safe). Copy the Terminal `:root` + `body.light` token blocks verbatim.
- **Admin prototype's hash-router + `SCREENS` registry** pattern for navigation, rendered in Terminal skin; app shell (topbar/rail/cmdline) shared across screens; rail's existing **Prompts** item active.
- **Mock state as in-memory JS objects** following PRD Appendix B shapes (`Prompt`, `PromptVersion`, `version_tag`, `labels`, `references`, `default_binding`) — so the mock data *is* documentation of the target data model.
- **Seed data = the Maya scenario** (PRD §7), so every screen tells one coherent story: `support-triage` (chat, v12, production, 50-ticket batch, a v11-vs-v12 compare with a marked winner), `tone-rewrite` (text, staging, with a compression run showing ~40% savings), `safety-guidelines` (the referenced prompt), one draft, one archived, one with a **red missing reference** (states must be honest). Secrets: `OPENAI_KEY`, `ANTHROPIC_KEY` + an Ollama endpoint. Providers/models: real names from the repo registry (`claude-sonnet-4-6`, `gpt-4.1-mini`, Ollama `llama3`…).
- **Interactions faked honestly:** runs "execute" with a 1–2s staged animation (cmdline caret blinks, node-style progress, then results + amber cost). Nothing pretends to hit a network.

**P0 screens (the sacred loop — build in this order, per PRD priorities):**
1. **Prompts list** — table (Name · Kind · Labels · Version · Env · Last run · Runs · ⋯), row-hover Run, ⋯ menu (grouped: Open/Run — Duplicate/Rename/Edit labels — Promote/Export — Archive/Delete-red), search, sort, New ▸ chat/text, honest empty state.
2. **Edit mode — chat prompt** — message-segment stack (system/user/assistant), placeholder chip (`chat_history`), reference chips (green/red), `{variable}` highlighting, inspector cards (Output · Inputs · References · **amber Token budget**), right-click segment menu.
3. **Edit mode — text prompt** — single `.ta` canvas + same inspector.
4. **Quick-run dockpane** — runner binding picker (progressive: model → credential-from-Secrets or endpoint-for-local; **Run disabled until runnable**; "Use for this run only / Save as version default"), inputs form, output area, Capture.
5. **Version history + promote** (Manage ▸ Versions + Promote dialog) — the governance half of the loop: version rows (v · tag · when · author), diff, restore ("restores as v13" — the repo's non-destructive semantics, kept), promote dialog moving the env pointer.
6. **⌘K palette** — Go to / Create / Do / Insert reference / Recent.

**Exit criteria for P0:** Maya's chat-prompt story is clickable end-to-end (open list → open prompt → edit → quick-run → save (auto v12) → promote to production), in dark *and* light, keyboard-navigable, every screen with loading/empty/error states designed.

### Phase 4 — P1/P2 depth + critique + ship *(6–9 Jul, iterative — Zaki: "no rush, but not snail pace"; depth over breadth)*

**P1:** Run mode (Single · **Batch** with load-from-dataset + results table · **Compare** 2–3 columns with diff-vs-first + winner · **Inspect** with assembled-prompt/request/response tabs, token breakdown, waterfall) → Manage (Details/labels with scope-grouped autocomplete · Access · Settings with archive/delete cards) → dialogs (Rename/Promote/Export/Archive/Delete-type-name).
**P2:** **Compress view** (3 method cards, keep-rate slider ↔ target tokens, protected-token preview with strikethrough, amber before/after strip, Apply-as-new-version) — P2 in PRD priority but **presentation-critical** (it's the differentiator; it gets full polish), multimodal outputs (JSON/image/file rendering), evaluator card.
**Critique loop (same as flow editor):** 3 structured rounds with the `ui-ux-audit` skill + Playwright screenshot passes (dark/light, cache-buster `?v=N`), each round logged in `04-DESIGN-DECISIONS.md`.
**Ship:** final HTML in this folder + published as a **claude.ai Artifact** + committed to the repo branch **`prompt-management-v1`** (with a README pointing at these docs). Post-ship: LinkedIn learnings post (Zaki's suggestion — versioning/tuning/experimentation).

### Ongoing process (from the huddle)
- **Daily syncs** with Zaki — each sync: what shipped, what's next, one decision to ratify.
- **PRD questions → GitHub issues** (never Slack) so Zaki can wire the backend via GitHub MCP.
- **Don't lean on the prior Keystone repo UI as a design reference** (Zaki: start fresh; simple focus on interaction + user flows). We use the repo only for *vocabulary and truth*, not layout.

---

## 6. Screen inventory (complete checklist, PRD Part 2 → prototype)

| # | Screen / surface | PRD ref | Priority |
|---|---|---|---|
| 1 | App shell (topbar · rail · cmdline) | App. A | P0 |
| 2 | Prompts list + empty/loading/error states | App. C | P0 |
| 3 | Edit — chat kind (segments, placeholder, references) | App. D | P0 |
| 4 | Edit — text kind | App. D | P0 |
| 5 | Inspector cards (Output/Inputs/References/Token budget) | App. D | P0 |
| 6 | Quick-run dockpane + runner binding picker | App. D + E-runner | P0 |
| 7 | Manage ▸ Versions (history, diff, restore, promote) | App. G2 | P0 |
| 8 | ⌘K palette | App. I | P0 |
| 9 | Run ▸ Single | App. E | P1 |
| 10 | Run ▸ Batch (+ load from dataset) | App. E | P1 |
| 11 | Run ▸ Compare (2–3 columns, winner) | App. E | P1 |
| 12 | Inspect (request/response, tokens, waterfall) | App. E | P1 |
| 13 | Manage ▸ Details/Labels/Access/Settings | App. G | P1 |
| 14 | Dialogs: Rename · Promote · Export · Archive · Delete | App. H | P1 |
| 15 | Compress view | App. F | P2* (*full polish — differentiator*) |
| 16 | Multimodal outputs (JSON/image/file chips) | B.3b, E | P2 |
| 17 | Evaluator card (occasional) | App. G3 | P2 |

---

## 7. Design decisions already made (seeding `04-DESIGN-DECISIONS.md`)

1. **Terminal skin over Admin skin** — Zaki's latest iteration wins; Admin remains the structural foundation (router, tables, states discipline).
2. **shadcn New York as the skeleton, terminal as the clothes** — every element names its shadcn primitive in a code comment, keeping the dev handoff honest.
3. **Variable syntax shown as `{variable}`** (repo's f-string reality), not the PRD's `{{variable}}` — pending GitHub confirmation; the highlighting/auto-input behavior is identical either way. `{{secret:KEY}}` stays as-is (it's real).
4. **Env badge colors**: production=green, dev=cyan, staging=neutral-bright, draft=dim — because amber is constitutionally reserved for cost (Terminal rule #8); the PRD's "amber staging" loses to the system rule.
5. **Restore semantics stay non-destructive** ("Restore v9 → creates v13") — matches shipped backend behavior and is honest governance.
6. **No Connections screen** — PRD and repo agree it's deferred; runner binds to Secrets directly.
7. **Real model names from the repo registry** in all pickers (incl. Ollama free-text + base_url for local) — Anthropic + OpenAI prominent per huddle, others present but not featured.
8. **Toasts via cmdline**, not floating Sonner — the terminal-native answer, one pattern.

## 8. Open questions → GitHub (drafted in Phase 1–2, `05-GITHUB-QUESTIONS.md`)

1. Variable syntax: PRD shows `{{message}}` / mustache-vs-f-string field; code ships f-string `{word}` only. Which does the prototype teach?
2. Chat prompts: library entity today can't store `system_message` (flat template). Is the B.2/B.3 `messages[]` body shape the committed direction?
3. Version-tags: fixed enum (`production/staging/dev/draft`) per PRD vs free-text environments as secrets use today — one vocabulary for both?
4. Labels: is the 3-scope (system/org/project) + curated/ad-hoc governance model v1, or do prompts start with flat labels like flows' tags?
5. Compression: LLMLingua runs where (server-side lib? which of the 3 methods ship first)? Does "Apply" always mint a new version?
6. Cost: no price table exists — is a rate card planned (Admin catalog had In/Out $/1K), or do we show tokens-only until then?
7. Direct prompt runs: new endpoint, or compiled through an ephemeral flow? (Affects what Inspect can honestly show.)
8. Does Keystone's internal Langfuse usage extend to syncing prompts to Langfuse, or is prompt storage Keystone-native only?

## 9. Risks & guardrails

- **Scope creep** — the PRD is huge; the huddle's rule wins: *few well-crafted features beat incomplete breadth*. P0 must be excellent before P1 exists.
- **"AI-generated without craft"** — Zaki's explicit caution. Guardrails: the principles doc gates every screen; 3 critique rounds; every component must cite the principle that earned it; the decisions log proves human judgment.
- **Vocabulary drift** — the Langfuse "labels ≠ our labels" trap; the crosswalk table in `01-…md` is the antidote.
- **7-vs-5 principles** — resolved by pressure-test-and-present (see Phase 2); Zaki ratifies at a sync.
- **OneDrive + Playwright caching** — use `?v=N` cache-buster on every screenshot pass (known trap from the flow-editor rounds).

---

*Next action once this plan is approved: Phase 1 research kickoff (Langfuse deep-dive first), findings shared by 4 Jul.*
