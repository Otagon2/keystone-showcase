# Redesign brief — shadcn New York fidelity pass (v2)

*Applies to all four prototypes. Goal: raise the surfaces to **authentic shadcn/ui New York** fidelity (per `SHADCN-NEWYORK-SPEC.md`) and apply the `GAP-ANALYSIS.md` §F actions, while **preserving every audited interaction** (no regression). Keep Zaki's contract: shadcn default styling, dark developer-console theme, lucide icons, the "one rule" interaction model, centered dialogs, type-name-to-confirm.*

## Every agent reads (in this order)
1. `briefs/SHADCN-NEWYORK-SPEC.md` — the fidelity contract (radius ladder, component anatomy, elevation, type, checklist).
2. `GAP-ANALYSIS.md` §F — the specific redesign actions (F1 badges→rounded-md, F2 radius ladder, F3 elevation, F4 8pt+type, F5 anatomy, F6 no-regression).
3. `briefs/STYLE-CONTRACT.md` — the shared shell/token/interaction contract.
4. Its own **issue brief** (`ISSUE-1/2/3/4-*.md`) — the source of truth for scope.
5. Its **baseline prototype** (the shipped `Keystone-*-Prototype.html`) — the starting point.

## The rules
- **Refine, don't rebuild.** Start from the baseline file; keep all mock data, routing, state, and the signature interactions (streaming run, drag-reorder, animated queue-resolution, skeleton loaders, undo toasts, count-ups, ⌘K, keyboard). Change the **skin to true New York**, not the behaviour.
- **Write to a new file** `Keystone-<Surface>-Prototype-v2.html` so the baseline is preserved for comparison. (Org Admin is a brand-new `Keystone-OrgAdmin-Prototype.html`.)
- Apply the fidelity checklist (SPEC §7): badges `rounded-md`; radius ladder 6/8-10/12; card `--elev-1`, menu `shadow-md`, dialog `shadow-lg`; buttons/inputs ~h-9 text-sm; tabs muted-list + active bg-background shadow-sm; tables muted header + `bg-muted/50` hover + tabular numbers; sans UI text, mono only for tokens/versions/cost/code; focus-visible ring; AA contrast both themes; every element comments its shadcn primitive; empty/loading/error states on every list.
- **Self-contained HTML**, no external requests, dark default + working light toggle, `prefers-reduced-motion` gated.
- **Validate:** extract the `<script>` and run `node --check` via a temp file; fix any parse error. **Do NOT use Playwright or open a browser** (the orchestrator runs the ui-ux-audit + browser verification centrally afterward).
- **Return:** file path · a short list of the New-York fidelity changes made · confirmation that all baseline interactions were preserved · any deviations + reasons · known rough edges.
