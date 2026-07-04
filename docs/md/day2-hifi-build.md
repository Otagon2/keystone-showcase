# Keystone — Hi-fi Build Log

Journal of the shadcn New York prototype build (`C:\Users\mdamk\keystone-hifi`), documented as built.
Run: `cd keystone-hifi && npm run dev` → http://localhost:5188. Plan: `HIFI-PLAN.md`. Lo-fi source: `REVIEW.md`.

Build mode: **standalone clickable prototype** (fresh Vite scaffold, mock GST data) — the recommended default from
HIFI-PLAN §1, pending Zaki's confirmation on repo-vs-standalone + deploy.

---

## Foundation ✅
- **Stack:** React 18 + TypeScript + **Vite 8**, react-router-dom, Radix UI, class-variance-authority + clsx +
  tailwind-merge, lucide-react, Tailwind 3.4 + tailwindcss-animate, cmdk. Mirrors `keystone-repo/apps/keystone-ui`.
- **Theme (dark-leads):** `src/index.css` — shadcn slate tokens, `:root` light / `.dark` dark, `<html class="dark">`
  default. **Keystone semantic tokens** added: `--pass`, `--fail`, `--env-{prod,staging,dev,draft}` (both themes).
  Toggle in the header persists to localStorage. **Verified both themes render** (screenshots).
- **`components.json`** New York + slate + cssVariables; `cn()` in `src/lib/utils.ts`; `@/*` path alias.
- **UI components (20, hand-written):** button · card · badge · table · tabs · dialog · alert-dialog ·
  dropdown-menu · select · command · tooltip · input · textarea · label · separator · checkbox · progress ·
  skeleton · accordion · sheet. *Why hand-written:* the shadcn CLI throws `EPERM … Application Data` (Windows
  junction) walking up from the home dir. Hand-authoring gave exact New York geometry (`h-9`/`rounded-md`/
  `[&_svg]:size-4`) matching the repo's `button.tsx`, and full control over the strict tsconfig
  (`verbatimModuleSyntax`, `erasableSyntaxOnly`).

## Shell ✅ (`src/components/shell/`)
- **AppShell** — loop-staged rail (Overview · Pinned · **Build**/Datasets,Prompts · **Evaluate**/Evaluators,
  Experiments · **Observe**/Traces · **Soon**/Operate,Learn[dimmed]) + workspace switchers + account footer
  (Amaan · inbox 3). Header = route breadcrumb + ⌘K search button + theme toggle. `Outlet` for pages.
- **CommandPalette** — cmdk `CommandDialog`; groups Go to · Create · **On GST QnA** (full 10 verbs) · Insert
  reference · Recent. Context-aware (named for the open prompt); every item routes. Opens on ⌘K / Ctrl-K + the
  header search button.
- **Shared chrome** (`components/keystone/`): `bits.tsx` (EnvBadge, StatusCell, ScoreCell, StatusDotIcon) +
  `chrome.tsx` (SectionLabel, MetricTile, **LoopRibbon**, EmptyState, DetailBreadcrumb) — reused across screens
  for visual consistency.

## Data ✅ (`src/lib/data.ts`)
Mock GST worked example, real repo entities only: 6 prompts (GST QnA v13 → Invoice Extractor), 6 experiment rows
(gst-001…006 with input/status/score/tokens/latency/output/expected/reasoning), Experiment #7 summary
(83% · 10/12 · avg 1,197 tokens · 2.0s · 14.9k total), dataset/prompt names. **Token-spend only, zero `$`.**

## Screens
Built via a **12-agent parallel workflow** (`build-hifi-screens`, wf_d0f3c746-038) — one agent per page file
against a detailed lo-fi spec + the reference files, using only existing components + tokens. 12/12 succeeded, 0
errors. Then wired all routes in `App.tsx`, **`tsc --noEmit` exits 0** for the whole app, and screenshot-verified
every screen (dark; light spot-checked on the color-heaviest screens). Proofs: `decisions/assets/hifi-*.png`.

**All 13 screens ✅ (typecheck clean, both themes):**
- **Prompt List** (`/prompts`) — data table, env dots, icon-shape status, badges, Run + ⋯ menu, split New-prompt.
- **Prompt Edit** (`/prompts/:id`) — two-pane: compose (Chat/Text, System/User/Assistant messages, `{question}`
  chip, Model select) + inspector (Output/Inputs/References/Token-budget) + live-test (Tokens+Latency tiles, **no
  cost**, Capture, "Run as an experiment →"). Mode tabs Edit/Run/Manage navigate.
- **Prompt Run** (`/prompts/:id/run`) — dataset select, "temporary until promoted" note, token/latency tiles,
  results table (Pass/Fail), "Run as an experiment →" seam.
- **Prompt Compress** (`/prompts/:id/compress`) — Before/After (1,240→890 tokens, 28%), greyed-removed + legend,
  LLMLingua ⚑ flag.
- **Prompt Manage** (`/prompts/:id/manage`) — versions table (v13 Current), environment cards, **Promote disabled
  on current-prod** (tooltip), Owner static + member Remove + role selects, labels + Delete AlertDialog.
- **Overview** (`/`) — 5-step getting-started checklist (Build/Evaluate/Observe), active-step highlight, "API key
  stays hidden", "how many tokens it takes", ⌘K hint, recent activity.
- **Datasets** (`/datasets`) — LoopRibbon, GST FAQ table (id/Question/Expected/Topic), column-mapping note.
- **Evaluators** (`/evaluators`) — LLM/Code toggle, template/model, variable chips, scoring rows, Advanced
  accordion, live-test (Evaluator tokens/Latency/Tokens, no cost).
- **Experiment** (`/experiments/:id`) — LoopRibbon, 83% scorecard, token/latency tiles, score-distribution bar +
  legend, results table (ScoreCell) → row navigates to trace.
- **Trace** (`/traces/:id`) — LoopRibbon, header (Fail·0.40 + `#` token chip + latency + Edit buttons), span tree
  (3.6k/1.1k/480 tokens), Reasoning/Messages/Raw JSON tabs, reasoning + Output/Expected.
- **Experiments list** (`/experiments`) + **Traces list** (`/traces`) — LoopRibbon tables → detail.
- **States showcase** (`/states`) — empty×3 + loading skeleton + running-progress + error (`n/a` + reason).

**Fidelity holds:** token-spend everywhere (zero `$`), single-brace `{question}`, GST QnA / AskJolly naming,
icon-shape status (WCAG), context-aware ⌘K (10 verbs), loop-staged rail, summary-first, Capture + seam wording,
promote-disabled / owner-static / member-remove (the carried minors) all realized. Dark leads, light co-equal.

## Responsive + breadcrumb ✅
- **Mobile nav** — rail extracted to a reusable `RailContent`; below `md` the header shows a **hamburger → Sheet**
  (full rail slides in over a scrim); search/breadcrumb go compact. Verified at 390px (`hifi-mobile-nav.png`).
- **Two-pane screens stack** below `lg` (Edit, Evaluators, Trace, Overview, Compress: `flex-col lg:flex-row`,
  right column `w-full lg:w-[…]`). Tables scroll horizontally on mobile (the `Table` overflow-auto wrapper).
  Verified (`hifi-mobile-list.png`, `hifi-mobile-edit.png`).
- **Breadcrumb fixed** — the shell now renders a single **full-path** breadcrumb (`AskJolly / Prompts / GST QnA`,
  route-aware incl. prompt/experiment/trace names); the redundant in-content `DetailBreadcrumb` was removed from
  all 4 Prompt detail pages (`hifi-edit-fixed.png`).
- `tsc --noEmit` clean after all edits. (Also fixed a `baseUrl` deprecation error → `paths` resolves without it.)

## Adversarial audit + fixes ✅
Ran a **4-auditor workflow** (audit-hifi, wf_3760e58e-137) over source + rendered proofs + decision docs, then a
**5-agent partitioned fix workflow** (fix-hifi-audit, wf_47dffa0f-771, grouped by file-owner → no write
conflicts). **Decision compliance + theming came back clean** (zero `$`/cost, single-brace, icon-shape status,
tokens-only colours, both themes). Verdict: 1 blocker + 6 majors + 8 minors — all **interaction/handler bugs from
the parallel build** (screens hardcoding the GST example instead of reading route params), not design drift. All
fixed; **full `tsc --noEmit` exits 0**; key fixes re-screenshotted.

**Blocker:** `Trace` hardcoded gst-003 → every row opened the same Fail trace. → now `useParams`-driven; each row
shows its own verdict/input/output/reasoning, and **token totals reconcile** (root = prompt + evaluator = row
tokens). Verified: gst-001 shows **Pass · 0.90**, tokens **1,240 = 760 + 480** (`hifi-trace-pass.png`).

**Majors:** (1) all 4 Prompt-detail pages hardcoded "GST QnA" → now param-driven header + id-based nav (Compliance
Checker shows title/breadcrumb/**v8** consistent — `hifi-edit-compliance.png`; message body stays the GST example,
a documented prototype simplification). (2) List ⋯ **Delete** was inert → wired to a gated type-to-confirm
`AlertDialog`. (3) Experiment subtitle "Ask Jolly v13" → **"GST QnA v13"** (naming rule; `hifi-experiment-fixed.png`).
(4) Trace token non-reconciliation → fixed (above). (5) mobile List → **card view** below `md`
(`hifi-mobile-list-cards.png`). (6) mobile-nav Sheet + ⌘K dialog had no accessible **Title** → added sr-only titles.

**Minors:** removed the non-functional Sort icon (lo-fi regression); Manage delete now gated; "Showing 6 of 12
rows" captions on Experiment/Datasets; Evaluators pager label → "Row 1 of 6" (matches shown data); States errored
row shows **`rate limit · 429`** inline; Edit header/test Run wired + pager made functional; scrim → `bg-foreground/20`
token (dialog/alert-dialog/sheet); Sheet close focus ring; 5 ⌘K no-ops now route; dead `DetailBreadcrumb` removed.

**Status: demo-ready** for the desktop golden path + mobile, dark + light, audited + fixed. Next is the customer
step (#8): before/after pack + Adel review + esberi/keystone issue #1 — pending Zaki's build-mode/deploy answers.

---

## Iteration 2 — client feedback round (Jul 5, 2026)

Zaki & Safi walked all 13 screens of the clickable prototype; feedback was captured per-screen and implemented the same day, then rebuilt and redeployed.

**Structure & navigation**
- Loop-stage ribbon (Dataset → Prompt → Evaluator → Experiment → Trace) added to the Prompt screens (it already appeared on Dataset/Evaluator/Experiment/Trace).
- Edit / Run / Manage tab bar made identical in position across all three prompt tabs — it previously jumped between tabs.
- "States & edge cases" page added to the side rail (Reference group) so it's reachable, not URL-only.

**Onboarding (Overview)**
- Getting-started now starts fresh at step 1 and shows the whole loop — create dataset → write first prompt → add evaluator → run experiment → open trace — instead of dropping you mid-flow.
- Recent-activity rows are now clickable links to their dataset / prompt / experiment / trace.

**Every dead CTA wired up**
- Datasets: a real CSV import wizard — choose/drag a file (or the sample) → preview → **map each column to a role** (input / expected / metadata / ignore), the pattern we liked in Arize *and* Braintrust — plus a working Add-row and rename / edit-mapping / delete lifecycle.
- Evaluators: "Add to experiment" wired.
- Experiment: **Compare** (pass-rate delta + per-row score diff) and the result-row → **trace** drill-down.
- Experiments & Traces lists: per-row ⋯ actions, filter / sort / search, compare, and row-click navigation.

**Prompts**
- Chat vs text is now explained (one-line menu descriptions + an in-editor tooltip) and the two "New prompt" options open genuinely different editors (chat = system/user/assistant turns; text = a single free-form template).

**Fix**
- Trace deep-links now resolve to the requested row (the earlier gst-003 → gst-001 behaviour was a stale build; the source already honoured the id).

Delivered via four parallel build agents with strict file ownership, typecheck-clean (`tsc --noEmit` exit 0), then `npm run build` → redeploy to `docs/keystone/`.
