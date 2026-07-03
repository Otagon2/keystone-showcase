# UI/UX Audit Log — Keystone Prototypes (Issues #1, #3, #4)

*Method: `ui-ux-audit` skill (Practical UI + Refactoring UI synthesis), audit mode, three rounds. Surfaces reviewed via live browser (localhost:8931, dark + light): Prompts list / Edit / Run / Compress / Manage · Inbox Needs-action / All / detail / post-decision · Platform login / Organizations / Enter-org / entered Members. All fixes verified with `node --check` + re-screenshot.*

## Verdict
Strong baseline: hierarchy and states discipline carried by the shadcn New York token system; empty states, keyboard (⌘K/Esc), and `prefers-reduced-motion` present from the first build. The weak dimension was **accessibility floor details** (focus visibility, border contrast, one light-theme contrast failure) — all fixed in Round 2.

## Round 1 — build verification (functional pass)
| # | Sev | Finding | Fix | Status |
|---|-----|---------|-----|--------|
| 1 | P1 | Phantom horizontal scrollbar in Prompts editor (hidden quick-run panel translated outside content) | `.content{overflow-x:hidden}` | ✅ fixed |
| 2 | P1 | Env badge collided with breadcrumb name in crowded Run header (flex-shrink overlap) | `.header>*{flex-shrink:0}` + header scrolls | ✅ fixed |
| — | — | Interaction checks: approve-clears-queue + badge decrement (Inbox); login→MFA→portfolio→Enter-org→banner→members (Platform); no console errors anywhere | — | ✅ pass |

## Round 2 — dimension-by-dimension audit
| # | Sev | Dimension | Finding (where) | Measurable fix | Source | Status |
|---|-----|-----------|-----------------|----------------|--------|--------|
| 1 | P0 | A11y | No visible keyboard focus on buttons/links (all 3 files) — only inputs had focus outlines | global `:focus-visible` 2px ring (≥3:1 via `--ring`), offset 2px | P (states, WCAG) | ✅ fixed |
| 2 | P0 | Colour | Inbox `.btn.success` hardcoded near-black text → unreadable on dark-green `#15803d` in **light** theme | `color:var(--primary-foreground)` — resolves correctly in both themes (≥4.5:1) | P (button text ≥4.5:1) | ✅ fixed |
| 3 | P1 | Colour | Field/control borders below the ≥3:1 floor: light `#cbd5e1` ≈1.6:1, dark `#334155` ≈1.8:1 | light `--input:#7e889c` (the Admin-prototype foundation's own value, ≈3.5:1); dark `--input:#475569` (≈3:1) | P (field borders ≥3:1) | ✅ fixed |
| 4 | P1 | Buttons/hierarchy | Compress view showed two simultaneous primaries (Compress + Apply); Inbox rows repeated solid-red Reject down the queue (loud destructive before the decision point) | dynamic primary handoff: Compress is primary until computed, then Apply takes primary and Compress demotes to outline "Re-compress"; row-level Reject/Decline/Deny → outline (detail pane keeps full emphasis) | R (one primary; calm destructive) + P | ✅ fixed |

Notes — deliberate non-changes:
- 13–14px body size is developer-console density, consistent with the Admin foundation; the ≥18px rule targets long-form copy, which these surfaces don't have. Kept.
- "Apply compression" stays disabled-until-computed **per Zaki's brief** (client spec over the avoid-disabled-buttons guideline); the disabled state now uses the shared `.btn:disabled` treatment.
- Status dots/severity are never colour-only — each is paired with text (state chips, timestamps, category labels).

## Round 3 — re-verification
- All three scripts re-pass `node --check`; re-screenshots confirm: calm secondary actions in the Inbox queue (one affirmative accent per row), border contrast lifted, focus ring visible on tab-through.
- Remaining findings are P2 nitpicks only (e.g. per-column "winner" tally in Compare, richer light-theme shadows) → logged, not blocking. **Stopped per the early-stop rule.**

## Round 4 — design & interaction enhancement pass (post-audit polish)
*Requested improvement pass, staying strictly inside the shadcn / dark-console contract. All additions reduced-motion-gated; all three files re-passed `node --check` and were re-verified in the browser.*

**Shared elevation + motion layer (all 3):** two-part tinted shadow tokens (`--elev-1` resting cards, upgraded `--shadow-md/lg`); purposeful transitions on buttons/rows/tabs (150–170ms, ease-out); button press states (`translateY(1px)`); dialog/command-palette entrance (rise + fade + subtle scale) and menu entrance; scrim fade. Depth now reinforces hierarchy (Refactoring UI: "shadow = elevation," "light from above") instead of flat borders.

**Prompts:**
- **Streaming run output** — quick-run and single-run reveal the answer progressively with a blinking terminal cursor (`.out.streaming::after`), so the core loop *feels* like a live model call. Verified: streams then lands with token/cost/latency metrics.
- **Drag-to-reorder message segments** — the segment header is a drag handle (grip cursor); drop reorders the messages array with an undo snapshot and a drop indicator. Fills the one genuinely-missing editor interaction (was menu-only move up/down).
- **Count-up on compress savings** — the Saved/Shrink figures animate up when a compression computes, giving the differentiator its hero moment.
- Adding a message now focuses the new segment; resting cards (inspector, segments, tables) carry subtle elevation; list rows get a hover accent bar.

**Inbox (the signature interaction):**
- **Deciding animates the item out of the queue** — the row collapses (height→0) and slides right over 260ms before the feed re-renders, and the account-block badge **pulses** as the count drops. Resolved-in-place items (All view) get a success flash. The auto-resolve (Dana Ortiz) uses the same animation. Verified: Grant → row left, tab + badge 6→5, zero errors.

**Platform:**
- The Enter-org **banner slides down** on the crossing; **audit rows flash** amber when an action appends one live — making the "every crossing is logged" property visible in motion.

## Round 5 — second design pass (states, polish, undo)
*Second improvement pass. Same contract; the goal was closing real gaps, not gilding. All three re-passed `node --check`; skeleton render path verified in-browser (evaluate confirmed `.sk` shimmer elements mount on first paint, then resolve to real data at ~460ms — correct brief-load behavior).*

- **Skeleton loading states (all 3)** — the one missing state from Keystone's own 4-states discipline (loading). First visit to each list (Prompts list, Inbox feed, Platform orgs) shows a shimmer skeleton for ~460ms, then the real data, modelling an initial server fetch. Subsequent navigations are instant (data is in memory) — honest.
- **Custom scrollbars (all 3)** — thin, token-tinted `::-webkit-scrollbar` matching the developer-console aesthetic; also removes the raw native scrollbar from the crowded Run-mode header (the last flagged P2).
- **Undo in toasts (Prompts + Inbox)** — reversible actions (archive) now surface an inline **Undo** for ~5s (Practical UI: "always allow undo"). Toast gained an optional action affordance.
- **Compare winner tally** — each Compare column header now shows a live "N winners" count as cells are marked, so the reviewer sees which variant is ahead at a glance.
- **Consistency + micro-polish** — unified sidebar width to 240px across all three (Platform was 230); tokened `::selection` colour; resting elevation extended to more surfaces.

## Round 6 — shadcn New York fidelity redesign + Org Admin (research-driven)
*Preceded by fresh research (web + Lazyweb) and a written gap analysis — `GAP-ANALYSIS.md` ("what Zaki's contract wants vs what I originally planned") and `briefs/SHADCN-NEWYORK-SPEC.md`. Finding: the shipped prototypes already obey the contract's big reversals (shadcn default dark, lucide, sans+mono, Message/Text/JSON, flat labels, amber-staging), so this pass raises **New York fidelity** rather than course-correcting. 4 frontend-design agents were launched; the org spend limit killed them (as before) — the Prompts agent's edits had already landed, and the rest were finished inline.*

**Produced `-v2.html` for the three surfaces + a new Org Admin prototype:**
- **New York fidelity (all v2):** the material change is **badges pill→`rounded-md`** (shadcn New York badges are rounded rectangles, not full pills) — applied to Kind/Env/label badges (Prompts), state/outcome chips (Inbox), org-lifecycle + operator badges (Platform), keeping the status dot + exact colour mapping. Radius ladder confirmed (6px controls / 8–10px cards / 12px dialogs); card-family elevation broadened to `--elev-1`; table headers tightened to `font-medium`; avatars/dots stay round. **JS untouched** in all three — every audited interaction (streaming, drag-reorder, animated queue-resolution, skeletons, undo toasts, Enter-org, ⌘K) is preserved; the baselines are kept for comparison.
- **New — `Keystone-OrgAdmin-Prototype.html` (Issue #2, previously uncovered):** built at NY fidelity from the Pencil/Admin lineage + the synthesized `briefs/ISSUE-2-org-admin.md`. Eight org-scope surfaces — Overview (rollups + Needs-attention), **Members** (Owner/Admin/Editor/Viewer roles, invite, pending invites, role matrix), Projects, **Secrets** (write-once/masked-forever centerpiece), Connections, Billing (budget gate + rate card + invoices), API keys, Audit (with the `operator.entered_org` transparency row). Verified dark + light in-browser; `node --check` clean.

All four re-verified in the browser (Members, Secrets, Billing dark+light, v2 badge fidelity), zero console errors.

## Round 7 — responsive pass + weird-layout cleanup + doc viewer
*"The designs need to be responsive," "clean any weird transitions or layouts," "cover all flows." A layout-only layer — no interaction logic changed, so every audited flow (streaming, drag-reorder, queue-resolution, skeletons, undo, Enter-org, ⌘K, Secrets/Billing) survives untouched. All four re-passed `node --check`; verified live at 390×780 and 1440 on the pushed site.*

- **Off-canvas sidebar drawer (all 4)** — below 900px the fixed sidebar becomes a hamburger-triggered drawer that slides in over a scrim (`toggleSB()` + `.sb-scrim`); tapping the scrim or a nav item closes it. Above 900px the hamburger is hidden and the sidebar is permanent — desktop is unaffected.
- **Stacked splits + fluid grids** — Prompts editor (canvas + inspector) and Inbox (feed + reading pane) stack vertically on narrow screens; stat grids collapse 4→2→1; the Compare grid forces a single column so variant cards never crush.
- **Table overflow fix (the real bug)** — flex table rows were compressing below content width and *overlapping* on mobile. Fixed by giving every `.table`/`.resulttable` a `min-width` (720–760px) inside an `overflow-x:auto` content wrapper, so wide tables scroll horizontally instead of colliding. Verified fixed on Prompts, Members, Platform orgs.
- **Doc viewer (`docs/docs.html`) made responsive too** — sidebar stacks above the reading pane below 820px; reading padding tightened; tables scroll.

**Doc viewer — "Copy markdown" per doc.** Each document now has a **Copy markdown** button in the breadcrumb bar. It copies the current doc as *clean, well-formatted* Markdown — a fence-aware `cleanMd()` normalizer trims trailing whitespace, guarantees blank lines around headings, collapses runs of blank lines to one, and ends on a single newline (code fences are preserved verbatim). Async Clipboard API with a `execCommand` textarea fallback; button flips to "Copied" for 1.7s. Verified: renders, fires with zero console errors, and the normalizer passes a 6-assertion unit test. (The two Keystone-PromptManagement context docs — the execution plan and domain guide — were already in the viewer as `execution-plan.md` / `domain-guide.md`.)

## What already works (don't lose in future refactors)
Squint test passes on every screen (one focal point); group-gaps > inner-gaps throughout; numbers right-aligned `tabular-nums`; destructive friction scaled (type-name-to-confirm with gated red button); designed empty states everywhere; the Inbox act-to-clear vs read-to-clear distinction is enforced in code, not just styled.
