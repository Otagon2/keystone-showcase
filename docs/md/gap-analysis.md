# Gap Analysis — what Zaki's contracts want vs what I originally planned

*3 Jul 2026. Compares the **pre-contract research** (`Keystone-PromptManagement/00-EXECUTION-PLAN.md` + `03-DOMAIN-GUIDE.md`, also bundled as `Keystone-PromptManagement-Docs.pdf`) against **Zaki's four GitHub issue briefs** (#1 Prompt Management, #2 Org Admin, #3 Inbox, #4 System Admin). Purpose: surface every place my original thinking diverged from what the client actually asked for, so the redesign is grounded in the contract — not in my earlier assumptions.*

**Headline:** the pre-contract plan was built on two assumptions the contracts overturned — (1) the **Terminal-TTY skin**, and (2) the **maximal PRD data model**. Zaki's briefs instead mandate **shadcn/ui default styling, dark developer-console theme, lucide icons** and a **simpler prompt model**. The three prototypes already shipped were built to the contracts (so they're on-spec) — this analysis documents *why* they look different from my plan, and what still needs correcting for true fidelity.

Legend: **⟲ Reversal** (contract overturned my plan) · **＋ Expansion** (contract added scope I hadn't planned) · **✓ Alignment** (my plan already matched) · **↳ Redesign action**.

---

## A. Visual language — the biggest divergence

| # | What I planned (pre-contract) | What Zaki's contract wants | Type |
|---|---|---|---|
| A1 | **Terminal-TTY skin**: the prototype must look like a sibling of `Keystone-FlowEditor-Terminal.html` — monospace *everywhere*, hairlines instead of shadows, box-drawing frames (`┌ PROMPTS ┐`), glyph icons (`{}`, `✦`, `❯`), uppercase tracked micro-labels. | **"shadcn/ui + lucide icons, default styling, dark 'developer console' theme"** — stated verbatim in all four briefs. A conventional, clean shadcn surface. | ⟲ |
| A2 | **Monospace 13px for all text** (body, tables, everything); ligatures off. | **Sans (Inter/system) for UI**; mono reserved for IDs, versions, tokens, cost, code — the normal shadcn split. | ⟲ |
| A3 | **Radius 0** — square corners on everything. | shadcn **radii** — `rounded-md` (6px) controls, `rounded-lg`(8px)/`rounded-xl`(12px) cards & dialogs. | ⟲ |
| A4 | **Hairlines, not shadows** — 1px borders carry all separation; shadows banned except one dockpane. | New York **uses `shadow-sm` on cards** and a real elevation ladder (sm→popover→lg dialog). Depth is part of the style. | ⟲ |
| A5 | **Glyph icons** (Unicode). | **lucide SVG icons.** | ⟲ |
| A6 | **"Amber = cost, and only cost"** — a sacred single-purpose signal; my decision-log even ruled *"staging is NOT amber."* | Zaki's #1 brief (App. B) states the Env badge is **"green production, amber staging, blue dev, grey draft."** Amber is a status colour, not cost-exclusive. | ⟲ |
| A7 | **Toasts as one-line cmdline messages** (terminal-native). | shadcn **Sonner-style toasts** (floating cards). | ⟲ |
| A8 | **Hierarchy by value & weight, not hue**; colour rationed hard. | Normal shadcn colour usage — semantic status colours (success/warning/destructive/info) used freely but accessibly. | ⟲ |

**Why the reversal is total:** the Terminal skin was the *latest internal iteration* when I wrote the plan; Zaki's briefs (written after) explicitly re-set the visual contract to plain shadcn. **The client's written instruction supersedes the internal iteration.** The Terminal skin now belongs only to the flow-editor exploration.

---

## B. Prompt-management model — simpler than I planned

The pre-contract plan followed the *maximal PRD* (12 JTBDs, a full data model in Appendix B). Zaki's issue #1 brief is a **leaner** version of that same PRD. Differences:

| # | What I planned (from the maximal PRD) | Zaki's #1 brief | Type |
|---|---|---|---|
| B1 | **Output types: text · JSON · image · file · message** (multimodal, PRD B.3b) — with image thumbnails / file chips rendered by type. | **Output = a ToggleGroup of Message / Text / JSON** only. Multimodal output dropped. | ⟲ |
| B2 | **Labels = 3-scope governance** (system/org/project), curated vs ad-hoc, autocomplete grouped by scope (PRD B.4b). | **Free-form tags** — grey pills, add/remove, "Manage tags." Flat. No scopes, no curation. | ⟲ |
| B3 | **A full "runner binding" appendix** (PRD App. E-runner): progressive credential picker, Secrets select, "+ Add API key," endpoint-for-local, **run-gated-until-credential**, "use once vs save as default." | Quick-run = **model + temperature + max-length + attachments**. No elaborate Secrets/credential picker, no run-gating on credentials. | ⟲ |
| B4 | Variable syntax **`{variable}`** (single brace — the repo's f-string reality), flagged as a GitHub question. | **`{{message}}`** (double brace) — stated in the brief. | ⟲ (resolved in Zaki's favour; the shipped prototypes use `{{ }}`) |
| B5 | **Compression + token economics = THE headline differentiator**, "cost coach" made visible everywhere in amber. | Compression (App. E, LLMLingua ×3, before/after) is **one mode among equals**; no "cost coach" framing. | ⟲ (still build it well — but not over-emphasised) |
| B6 | A standalone **7-principles document** (5 core + 2 prompt-specific), pressure-tested as a deliverable. | Each brief carries its **own inline principles** (Inbox 5, System Admin 5, etc.); the huddle said 3–5. No standalone 7-principles artifact requested. | ⟲ |

**Net:** the contract's prompt surface is **meaningfully simpler** — fewer output types, flat labels, a lightweight quick-run. The shipped Prompts prototype already reflects this (Message/Text/JSON toggle, flat label chips, simple quick-run). Good.

---

## C. Scope — the engagement tripled

| # | Pre-contract plan | Zaki's contracts | Type |
|---|---|---|---|
| C1 | **Prompt management ONLY** (one surface, one prototype). | **Four issues**: #1 Prompt Mgmt, #2 Org Admin, #3 Inbox, #4 System Admin. | ＋ |
| C2 | — (not planned) | **#3 Inbox** — a cross-project feed + task queue whose load-bearing mechanic is *informational (read-to-clear) vs actionable (act-to-clear)*. Entirely new. | ＋ |
| C3 | — (not planned) | **#4 System Admin** — a control-plane surface with the tenant-boundary model and the audited *Enter-org assume-role*. Entirely new. | ＋ |
| C4 | — (not planned) | **#2 Org Admin** — assigned but **with no brief text**; leans on the existing Pencil/Admin design. Now being built as the 4th prototype. | ＋ |

---

## D. Process & method

| # | Pre-contract plan | Zaki's contracts | Type |
|---|---|---|---|
| D1 | Front-load **Langfuse/Phoenix deep research**, a **domain-expert guide**, and a **GitHub-questions doc** as first deliverables (Phases 1–2 before pixels). | The briefs are **self-contained** ("no prior Keystone knowledge assumed") — the ask is to **build the prototypes from the briefs**. Research/domain-guide are still useful (and done) but are not the contract. | ⟲ (soft) |
| D2 | Build in the **Terminal repo branch** `prompt-management-v1`, ship as a claude.ai Artifact. | Build **standalone HTML** per surface, shadcn default. (Branch/Artifact still fine as delivery.) | ✓/⟲ |

---

## E. Where my plan already matched the contract (keep these)

| # | Both agree |
|---|---|
| E1 | **The "one way" interaction rule** — my principle #3 is *exactly* Zaki's stated rule: a Tabs trigger switches mode · a toolbar icon acts on the whole object · a ⋯ menu acts on the one item · every pop-up is a centered dialog. ✓ |
| E2 | **Versioning by default** — every save is a numbered version; governance as a byproduct. ✓ |
| E3 | **Non-destructive restore** ("restore v9 → creates v13"). ✓ |
| E4 | **Destructive friction scales** — archive = simple confirm; delete = type-the-name. ✓ |
| E5 | **⌘K command palette** everywhere; keyboard-first. ✓ |
| E6 | **Chat vs text prompt kinds**, `{{variable}}` auto-creating inputs, reference chips (green/red), placeholders. ✓ |
| E7 | **The builder persona**; mock-only, no backend wiring. ✓ |

---

## F. What this means for the redesign (the action list)

The shipped prototypes already obey the contract on the big reversals (they're shadcn default dark, lucide, sans-with-mono-accents, Message/Text/JSON, flat labels, amber-staging). So the redesign is **not** a course-correction — it's a **fidelity pass** to make them read as *authentic* shadcn New York, plus the new Org Admin build.

- **↳ F1 — Badge radius.** Current badges are pills (`rounded-full`); shadcn New York badges are **`rounded-md`** rectangles. Switch (keep the coloured status dot). *(Env badges may stay slightly rounded, but align to the NY badge, not a full pill.)*
- **↳ F2 — Differentiated radius scale.** Current flat 8px everywhere → **6px controls / 8–10px cards / 12px dialogs** (the NY ladder), so hierarchy reads through radius too.
- **↳ F3 — New York elevation discipline.** Cards carry a genuine `shadow-sm`; popovers `shadow-md`; dialogs `shadow-lg` — already partly done in the motion pass; make it systematic and token-driven.
- **↳ F4 — 8pt spacing rhythm + type scale.** Snap paddings to 4/8/12/16/24; base UI text 14px (`text-sm`), `font-medium`/`font-semibold` for emphasis, mono only for tokens/versions/cost.
- **↳ F5 — Component anatomy to spec** (see `briefs/SHADCN-NEWYORK-SPEC.md`): Button `h-9`, Input `h-9`, Tabs list on `bg-muted` with active `bg-background shadow-sm`, DropdownMenu `shadow-md rounded-md`, Table muted header + `bg-muted/50` row hover, focus-visible ring with offset.
- **↳ F6 — Keep every audited interaction** (streaming run, drag-reorder, animated queue-resolution, skeletons, undo toasts, count-ups). No regression — the redesign refines the *skin*, not the *behaviour*.
- **↳ F7 — Build the Org Admin prototype** (#2) from the Pencil/Admin design lineage, same NY fidelity, covering the org-scope governance surfaces (Overview · Members/roles · Projects · Secrets · Connections · Billing · API keys · Audit · Approvals).
