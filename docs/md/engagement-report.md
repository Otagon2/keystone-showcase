# Keystone Design Report — what we built, the principles, and the decisions

*2–3 July 2026 · Amaan (design) · covering the prompt-management engagement and Zaki's four GitHub issues. This is the narrative record: every deliverable, every principle we designed by, every decision with its reason, and the method used to get there.*

---

## 1. Executive summary

The engagement went from a kickoff huddle to **four client-ready, clickable prototypes** (one per GitHub issue) plus the research and documentation spine behind them:

| Deliverable | What it is | Where |
|---|---|---|
| Execution plan | Master plan: phases, screen inventory, PRD-vs-repo gap audit, seeded GitHub questions | `Keystone-PromptManagement/00-EXECUTION-PLAN.md` |
| Domain guide + jargon dictionary | "Become a domain expert" doc: prompt management from zero, market state (Langfuse/Phoenix verified against live docs), 14 future assumptions → design decisions, ~120-term glossary | `Keystone-PromptManagement/03-DOMAIN-GUIDE.md` (+ PDF) |
| **Prompts prototype** (Issue #1) | The full write → try → compare → compress → govern loop, Maya scenario seeded. `-v2` = shadcn New York fidelity pass | `Keystone-Prompts-Prototype.html` (+ `-v2`) |
| **Inbox prototype** (Issue #3) | Cross-project feed + task queue with the act-to-clear mechanic enforced in code. `-v2` = NY fidelity | `Keystone-Inbox-Prototype.html` (+ `-v2`) |
| **Platform prototype** (Issue #4) | System Admin control plane with the audited Enter-org assume-role journey. `-v2` = NY fidelity | `Keystone-Platform-Prototype.html` (+ `-v2`) |
| **Org Admin prototype** (Issue #2) | Org-scope governance console — Members/roles · Projects · Secrets · Connections · Billing · API keys · Audit; built at NY fidelity from the Pencil/Admin lineage | `Keystone-OrgAdmin-Prototype.html` |
| Gap analysis + shadcn NY spec | "What Zaki's contract wants vs what I originally planned" + the New York implementation reference | `GAP-ANALYSIS.md` · `briefs/SHADCN-NEWYORK-SPEC.md` |
| Briefs + style contract | Zaki's four briefs (incl. the synthesized #2) + the binding visual/interaction contract + redesign brief | `briefs/` |
| Audit log | Six rounds: 3 formal ui-ux-audit + 3 design-refinement passes (feel · states · NY fidelity), with findings/fixes/non-changes | `AUDIT-NOTES.md` |
| GitHub comment drafts | Ready-to-paste replies for issues #1–#4, including the open backend questions | `GITHUB-COMMENTS.md` |
| Reusable skill | The whole workflow packaged: research → build → audit, one command for future briefs | `~/.claude/skills/brief-to-prototype/` |

Every prototype is a single self-contained HTML file — no framework, no build step, no backend, no network calls — exactly what Zaki asked for ("mock prototype, no real backend wiring"). All four pass syntax validation, run with zero console errors, and were exercised end-to-end in the browser in dark **and** light themes. The `-v2` files are the New York fidelity refinement of the three originals (behaviour identical; badges pill→rounded-md, etc.) — kept alongside the v1 baselines for comparison, not overwriting them.

## 2. The journey

| When | What happened |
|---|---|
| Jul 2, morning | Huddle with Zaki: scope (prompt management first), method (3–5 pressure-tested principles), differentiators (LLMLingua compression + model flexibility), process (GitHub not Slack, daily syncs, mock-only) |
| Jul 2 | Execution plan written; `keystone-main` audited end-to-end — the **PRD-vs-repo gap table** became the backbone for both the mock scope and the GitHub questions |
| Jul 2 | Domain guide written; Langfuse + Arize Phoenix facts **verified against their live docs** (not memory); jargon dictionary added on request |
| Jul 2, evening | Zaki filed four GitHub issues (#1 Prompt Management, #2 Org Admin — no brief text, #3 Inbox, #4 System Admin), all assigned; every brief specifies **"shadcn/ui + lucide, default styling, dark developer console"** — overriding the earlier Terminal-TTY direction |
| Jul 3 | Briefs persisted verbatim + style contract authored; Lazyweb research pass; parallel build agents launched — **all four killed by the org's monthly spend limit** — so all three prototypes were built inline instead |
| Jul 3 | Pencil file checked (org/admin screens exist for #2; Inbox = zero coverage; new-spec System Admin frames missing — deferred to a Pencil session) |
| Jul 3 | Browser verification (interactions exercised, 2 layout fixes) → formal 3-round ui-ux-audit (4 findings fixed, 2× P0) → GitHub comment drafts → this report |

## 3. Ground truth we established first

Design decisions were made against evidence, not assumptions:

- **The PRD is target-state; the repo is minimal.** `keystone-main` stores a prompt as `{name, description, template, version}` — integer versions with non-destructive restore, f-string `{variable}` (not mustache), `{{secret:KEY}}` tokens, flow-node pinning. It has **no** version-tags, labels, prompt references, compression, direct prompt runs, dollar costs, or Connections. Everything the prototypes mock beyond that is deliberate target-state design, and each gap is a drafted GitHub question.
- **The PRD is a remix of Langfuse and Phoenix** (verified live): Langfuse's text/chat kinds, `{{variables}}`, labels-as-deployment-pointers, protected labels, `@@@prompt@@@` composability; Phoenix's fixed tag set (production/staging/development) and playground energy. Keystone's two bets that neither has: **LLMLingua compression with visible token economics** and **builder-owned credentials with first-class local models**.
- **Reference patterns from research:** Linear/Front-style split list + reading pane with inline row actions (Inbox); status-badged metadata tables (platform consoles).

## 4. The seven principles we designed by

*The huddle rule: principles must be able to lose an argument — each one lists where it forced a real decision in the build.*

**P1 · The loop is sacred.** Write → try → inspect → fix → ship, with the common path one click. *Visible:* quick-run lives inside Edit (no mode switch to try); hover-Run on every list row; ⌘K reaches any prompt/action in two keystrokes; Compare hands winners straight to Promote.

**P2 · Governed by default, not by chore.** Versioning and audit are byproducts of normal work, never a task. *Visible:* every save auto-mints a version ("Saved as v13"); restore is non-destructive ("Restored v9 as v13"); promotion moves a pointer via an explicit, confirmed dialog; every Platform action appends a live audit row.

**P3 · One way to do a thing.** A Tabs trigger switches mode · a toolbar icon acts on the whole object · an ⋯ menu acts on the one item clicked · every pop-up is a centered dialog. *Visible:* the same anatomy across all three prototypes — a reviewer flipping between them sees one product. This is Zaki's "one rule," implemented literally.

**P4 · Show, don't hide, what happened.** Every result is inspectable to its raw internals. *Visible:* Inspect opens the fully-assembled prompt, raw request/response JSON, token breakdown (in/out/cached), cost, latency, and a step timeline; the Platform audit shows before→after on every event.

**P5 · Boundaries are features.** The builder owns keys; the operator can't read tenants; a decision can't be dismissed as a notification. *Visible:* secrets never render anywhere (operator included); Enter-org demands re-auth and wears a persistent audited banner; a pending actionable has no archive button — and clicking where it would be explains why.

**P6 · Cost is a first-class signal** *(prompt-specific)*. *Visible:* the Token-budget card computes size/cost live while typing; every run row carries tokens · cost · latency; Compress leads with Before/After/Saved/ratio/cost-saved — the differentiator gets the most honest arithmetic on the surface.

**P7 · Earn every component** *(craft)*. Nothing ships because a library has it. *Visible:* no stats bar on the list (brief said not yet); no side drawers anywhere; destructive friction scales with risk (archive = one confirm; delete = re-auth + type-the-name with the button gated on an exact match); the Compress "Apply" stays disabled until there is genuinely something to apply.

## 5. The craft floor (applied via the ui-ux-audit framework)

Beneath the product principles, every screen was held to the measurable floor from Practical UI + Refactoring UI: text ≥4.5:1 and UI borders ≥3:1 contrast (both themes); visible keyboard focus everywhere; one primary button per view; destructive actions calm until the decision point; never colour-only meaning (every dot pairs with text); designed empty states on every list; `prefers-reduced-motion` honored; numbers right-aligned tabular; hierarchy carried by weight and muted-foreground rather than ever-bigger text. Four violations were found in the audit (two P0) and fixed — the full table with measurements is in `AUDIT-NOTES.md`.

## 6. Design decisions and why

### Global (all three prototypes)
| Decision | Why |
|---|---|
| **shadcn New York, default styling, dark developer console — not the Terminal-TTY skin** | Every one of Zaki's four briefs specifies it; the client's written instruction supersedes our internal iteration. The Admin prototype became the foundation; the Terminal skin remains the flow-editor exploration |
| One self-contained HTML file per surface, mock-only | Zaki's explicit ask ("token-efficient agent rewiring", no backend); also makes review one double-click |
| Shared style contract file before any code | Three surfaces built (partly in parallel) must read as one product; the contract pins tokens, shell anatomy, and the "one rule" |
| Hash router + screen registry, plain DOM | The Admin prototype's proven pattern; every screen is URL-addressable for review |
| Inline lucide-style SVG icons, zero CDN | Self-contained constraint; icons stay crisp in both themes |
| Mock data tells the brief's scenario story | Maya's triage/tone journey (#1), Sam's triage-in-30-seconds (#3), Northwind's recovery (#4) — every screen demonstrates the narrative the client wrote, not lorem ipsum |
| Dark default with a first-class light theme | The briefs say dark developer console; the foundation demands light parity — both AA-verified |
| Toasts + ⌘K + Esc semantics everywhere | The rhythm of the product: acknowledge every action, reach everything by keyboard |

### Prompts (#1)
| Decision | Why |
|---|---|
| Auto-version on save (blur → "Saved as v13") | P2 made real: governance as a byproduct; also matches the repo's snapshot-on-edit behavior |
| Restore non-destructive ("Restore v9 → creates v13") | Matches shipped `keystone-main` semantics; history is never rewritten |
| `{{variable}}` syntax as the brief writes it, with a GitHub question flagged | The repo ships f-string `{var}` — a real conflict for engineering to settle, not for the mock to hide |
| Kind (chat/text) fixed at creation; toolbar toggle explains and points to Duplicate | The brief fixes the editor shape per kind; a silent convert would lie about the data model |
| Env badges: production green · staging amber · dev blue · draft grey | The brief specifies these exact colors (App. B) |
| References as chips with live resolution (green found / red missing, seeded with one broken) | States must be honest — the unhappy path is designed, not hidden |
| Quick-run panel slides inside Edit (not a dialog) | The brief's one exception to centered dialogs; keeps the loop unbroken |
| Compress: methods as plain-language cards, protected tokens visually locked, Apply mints a version | The differentiator must build trust: show exactly what's removed, prove slots survive, make it instantly reversible |
| Compare limited to 8 visible rows when 50 loaded, labelled | Honesty over spectacle: the label says what's shown rather than faking infinite rendering |
| Batch seeds one deliberate misclassification | Gives Inspect a reason to exist in the demo — debugging from evidence |

### Inbox (#3)
| Decision | Why |
|---|---|
| `item_class` drives everything: read-to-clear vs act-to-clear, enforced in the code paths | The brief's load-bearing rule; mark-read literally cannot resolve an actionable |
| Badge counts **pending actionables**, not unread | The badge answers "is anything waiting on me?" — unread noise would poison trust |
| No Inbox nav item; entry = badge on the account block | Brief App. A; the Inbox follows the person, not the project |
| Needs-action sorts by urgency (severity, then due), All by recency | A queue is not a timeline — the brief calls this out explicitly |
| One approval auto-resolves after ~20s ("Resolved by Dana Ortiz") | Demonstrates first-decision-wins/no-zombie-tasks live, the property that makes the queue trustworthy |
| Expired item shows "expired · missed", out of the queue but in the feed | Expiry never masquerades as done |
| Row-level Reject/Decline calm (outline); full emphasis only in the detail pane | Audit finding: repeated solid red down a queue is noise; destructive stays calm until the decision point |
| Placeholder Build screen included | Proves the "triage and hop back" journey — the Inbox is a surface you leave |

### Platform (#4, with #2 tie-in)
| Decision | Why |
|---|---|
| Login → MFA step first; re-auth again for delete and enter-org | "Even the operator re-proves identity" — JTBD-5 verbatim |
| Delete disabled until suspended, with the hint inline in the menu | The lifecycle rule enforced where the user would break it, not in documentation |
| Soft delete with visible recovery deadline + Restore | Blast radius stated, reversibility designed (30-day window) |
| Enter-org = two-step (re-auth → audited-session consent) → persistent amber banner + Leave | The crossing must feel like a crossing; the banner makes "I am inside a tenant" impossible to forget |
| Entered view is governance-only: members, projects, promote, transfer — nothing else | Control plane vs data plane; "no authoring" and mask-forever secrets stated on the surface itself |
| Audit appends live rows for every session action; operator rows highlighted with a one-click filter | The "prove isolation" view — the answer to "can your operator read our data?" |
| Northwind seeded flagged "no reachable admin" | Makes the recovery journey (enter → promote → leave) demonstrable end-to-end |
| ⌘K "Simulate fresh deployment" | Shows the day-zero empty state without deleting demo data by hand |

## 7. How it was designed — the method

**The three-skill pipeline** (now packaged as the reusable `brief-to-prototype` skill):
1. **Research** (design-new / Lazyweb): quick pattern scans per surface + live-doc verification of Langfuse/Phoenix — research fills gaps but never overrides a detailed client brief.
2. **Build** (frontend-design): briefs persisted verbatim to files; a binding style contract authored first; one self-contained HTML per surface with a "what must work" checklist derived line-by-line from the brief's appendices; scenario-seeded mock data.
3. **Audit** (ui-ux-audit): three rounds — functional verification with real clicks, then dimension-by-dimension against the measurable floor, then re-verification. Findings logged with severity and measurements; deliberate non-changes recorded with reasons.

**Verification discipline:** every file's script extracted and parsed with `node --check` before any browser time; then served over local HTTP (file:// is blocked) with `?v=N` cache-busters bumped on every edit; console watched for runtime errors; the load-bearing interactions clicked, not assumed — approve-clears-queue, badge decrement, the full enter-org journey, the compress compute-then-apply handoff.

**The fallback that saved the day:** all four parallel build agents were killed mid-run by the org's monthly spend limit. The deliverables didn't slip — the three prototypes were built inline in the main session instead, using the same briefs, contract, and checklists the agents had been given. That resilience is now written into the skill.

**Three design-refinement passes followed the audit** (full log in `AUDIT-NOTES.md`, Rounds 4–6), all staying strictly inside Zaki's shadcn / dark-console contract:
- *Round 4 — feel:* a shared elevation + motion layer (two-part tinted shadows, purposeful transitions, dialog/menu entrances, button press states); **streaming run output** so the model appears to "type" its answer with a terminal cursor; **drag-to-reorder** message segments; **count-up** on the compress savings; and the Inbox's signature move — **deciding animates the item out of the queue** with a badge pulse.
- *Round 5 — states & completeness:* **skeleton loading states** on every list (the one missing state from Keystone's 4-states discipline); **custom scrollbars** matching the console aesthetic; **undo in toasts** for reversible actions (Practical UI's "always allow undo"); a **Compare winner tally**; and cross-prototype consistency polish. All reduced-motion-gated; all re-validated in the browser.
- *Round 6 — shadcn New York fidelity + Org Admin:* preceded by fresh shadcn New York research and a written **gap analysis** (`GAP-ANALYSIS.md` — "what Zaki's contract wants vs what I originally planned"; the pre-contract plan assumed the Terminal-TTY skin + maximal PRD model, both overturned) and a fidelity spec (`briefs/SHADCN-NEWYORK-SPEC.md`). Produced **`-v2.html`** for the three surfaces (the material change: badges pill→`rounded-md` per authentic New York, radius ladder + elevation confirmed, **JS/interactions untouched**) and built the previously-missing **Org Admin prototype** (`Keystone-OrgAdmin-Prototype.html`, Issue #2): eight org-scope surfaces — Overview · Members (roles + invite + role matrix) · Projects · Secrets (write-once, masked-forever) · Connections · Billing (budget gate + rate card) · API keys · Audit. Verified dark + light.

## 8. What the client sees (selected screens)

*(Screenshots embedded in the PDF edition: Prompts list · Edit with reference chips and live token budget · Compress with protected slots and savings strip · Inbox Needs-action queue · Platform portfolio with lifecycle states · the entered-org audited banner · light-theme parity.)*

## 9. What remains

1. **Pencil frames** — Inbox and new-spec System Admin screens in `Keystone-Admin-Design.pen` (gap-check done; org-admin coverage exists for #2; best done in a fresh session with Pencil open).
2. **Post the GitHub comments** (drafted, not sent) — ideally after committing the prototypes to the `prompt-management-v1` branch so the comments can link to them.
3. **Issue #2 brief** — waiting on Zaki's Organization Admin text; the Enter-org governance view already covers the structural core.
4. Daily-sync talking points: the three backend questions from issue #1 (variable syntax, chat storage shape, price table).
