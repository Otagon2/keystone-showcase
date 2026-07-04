# Keystone — Hi-fi Plan (shadcn New York → clickable prototype)

**Input:** `REVIEW.md` (lo-fi complete + verified) · **Target:** `prompt-management-v1` for Adel
**Grounded in:** `keystone-repo/apps/keystone-ui` (read-only clone) + `keystone-repo/CLAUDE.md`

---

## 0. Stack is already ours — fidelity will be 1:1

The repo UI is **React 18 + TypeScript + Vite**, **react-router-dom**, **Radix UI** primitives (dialog,
dropdown-menu, select, tabs, tooltip, checkbox, label, separator, slot), **class-variance-authority + clsx +
tailwind-merge** (the shadcn `cn()` stack), **lucide-react**, **Tailwind 3.4** + tailwindcss-animate. That *is*
shadcn **New York** — the exact system our lo-fi was drawn against. Also present: `@assistant-ui/react` (chat),
`@xyflow/react` (canvas), CodeMirror (code editor). So the lo-fi maps to real components with **no translation loss**.

---

## 1. Decisions to confirm BEFORE coding (these change the plan)

| # | Decision | Options | Lean |
|---|---|---|---|
| **A** | **Build mode** | (a) **Standalone UI-only prototype** — fresh Vite+shadcn scaffold, mock data, no backend; fast for Adel to click. (b) **On a writable repo copy** (`prompt-management-v1` branch) wired to the real server API — then repo rules apply: **TDD, frozen contract, Core/Server, `ks` parity, docker-compose**. | (a) for a clickable demo; (b) if it must become production |
| **B** | **Deploy target** | Repo's real deployment is **docker-compose** (one image, FastAPI serves UI). My earlier note said "Heroku" — reconcile. A UI-only prototype can go to any static/Node host; the real app is compose-only. | Confirm with Zaki |
| **C** | **Data** | Mock the GST worked example (fast, deterministic) vs live server API (real runs, needs the stack up). | Mock for the demo |
| **D** | **Scope (from REVIEW.md §8)** | Evaluator-as-object (UI now, backend ⚑); Compression in/out; Org budgets (only legit `$`); canned sample project. | Ask Zaki |

> If **A(b)**, we must work in a **git worktree** on a feature branch, keep `make lint/test/e2e` green, and add a
> matching `ks` command for every UI capability. That's a different, heavier engagement than a clickable demo.

---

## 2. Theming — where dark-leads finally lands

Lo-fi deferred theming (Pencil can't render themed vars). Hi-fi is where it's real:
- **CSS variables, dark-first + light co-equal.** `:root` = dark; `.light` (or `[data-theme=light]`) overrides.
  Slate monochrome: `--primary` near-black-on-light / near-white-on-dark; neutral surfaces; **no indigo**
  (repo CHANGELOG removed it).
- **Semantic tokens** carried from lo-fi: `--pass` green, `--fail` red, `--env-{prod,staging,dev,draft}`,
  `--warning`/tool-amber left **as the repo already uses them** (amber is taken — we never reused it for cost).
- **Token-spend, no `$`** stays enforced (except the one flagged Org-budgets surface if D confirms it).
- Map our lo-fi variables → the repo's `tailwind.config` + `index.css` token names (reuse, don't invent).

---

## 3. Screen → component mapping (shadcn/Radix)

| Screen | Primary components |
|---|---|
| **Shell** | Sidebar (rail, Build/Evaluate/Observe groups) · Breadcrumb · `Command` (⌘K) · loop ribbon |
| **Prompt List** | `Table` + row `DropdownMenu` (⋯) · `Badge` (kind/labels/env) · icon-shape status · `Button` (New prompt `DropdownMenu`) |
| **Edit** | `Tabs` (Edit/Run/Manage) · message cards · `Select` (model, **version**) · `Badge` (env) · inspector cards · live-test `Card` + `Button` (Run / Capture) |
| **Run (batch)** | `Table` (streaming rows) · `Progress` · summary tiles · `Button` (Run as an experiment) |
| **Compress** | diff view (greyed removed words + legend) · token-spend before/after |
| **Manage** | version list · `Select`/static owner · members · promote (`AlertDialog`, disabled on current-prod) |
| **Dialogs** | `Dialog` / `AlertDialog` (centered, 60% scrim) — Delete (type-to-confirm), Rename, Promote, Export, Archive |
| **⌘K** | `Command` (cmdk) — Go to · Create · **On \<prompt\>** (10 verbs) · Insert reference · Recent |
| **Loop** (Overview/Dataset/Evaluator/Experiment/Trace) | `Card` · `Table` · `Progress` · span tree · `Tabs` (Reasoning/Messages/Raw JSON) · CodeMirror for JSON |
| **States** | `Empty` pattern · `Skeleton` · `Progress` · `Alert` (error banner) |
| **Responsive** | `Sheet` (mobile nav + tablet inspector drawer) · `Accordion` (mobile inspector) · Table→card at `<768` |

---

## 4. Build order (Shell first, then golden path, then breadth)

1. **Scaffold + tokens** — Vite+shadcn init (or repo worktree), theme vars (dark-leads), `cn()`, lucide, base
   layout shell (rail + header + ⌘K). Wire routes (react-router).
2. **Prompt List** (entry) → **Edit** → **Run** → **Experiment** → **Trace** — the golden path, end to end, with
   the GST mock data. This is the demo spine Adel walks.
3. **Manage · Compress · Dialogs · ⌘K** — complete the sub-app.
4. **Loop screens** (Overview/Dataset/Evaluator) + **states** + **responsive breakpoints** (real CSS).
5. **Polish pass** — the carried minors (REVIEW.md §7): version `Select`, env badge, promote-disabled, column
   order, two-co-primary tuning — cheap now that they're real components.

---

## 5. Fidelity checklist (carry every lo-fi decision)

- [ ] Summary-first everywhere; flat centered dialogs, no side-drawers (except the mobile/tablet `Sheet`).
- [ ] Token-spend + latency; **zero fabricated `$`** (Org-budgets only if D confirms).
- [ ] Single-brace `{variable}` (matches parser); `{{secret:KEY}}` reserved; **secrets never render**.
- [ ] Icon-shape status (WCAG 1.4.1), not colour alone.
- [ ] Context-aware ⌘K named for the open prompt; full 10-verb set.
- [ ] Real fields only (GST QnA v13, GST FAQ dataset, Q&A correctness) — matches repo entities.
- [ ] Loop-staged rail (Build/Evaluate/Observe); `AskJolly` breadcrumb root.
- [ ] Responsive: nothing desktop-only; usable to 390px.
- [ ] Dark + light both shipped and correct.

---

## 6. Verification (same rigor as lo-fi)

- Screenshot every screen (Playwright) in **both themes** + **3 widths**; compare to the lo-fi proofs.
- If A(b): `make lint` + `make test` + `make e2e` green; a `ks` command per UI capability; commits per green
  milestone on a worktree branch.
- Re-run an adversarial audit on the built UI before Adel sees it.

---

## Next action
Confirm **§1 A–D** (build mode, deploy, data, scope) → then Step 1 scaffold. Send Zaki the REVIEW.md + these four
questions in the same message.
