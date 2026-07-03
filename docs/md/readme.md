# Keystone Prototypes — folder index

*Design deliverables for Zaki's four GitHub issues (esberi/keystone). Four clickable, self-contained HTML prototypes on **shadcn/ui New York, dark developer-console theme** + the research, briefs, and audit trail behind them. Last updated 3 Jul 2026.*

## The prototypes (one per issue)

| Issue | File | What it is | Status |
|---|---|---|---|
| **#1 Prompt Management** | `Keystone-Prompts-Prototype.html` · **`-v2`** | Write → try → batch → compare → compress → govern loop (Maya scenario) | ✅ audited + NY-fidelity v2 |
| **#3 Inbox** | `Keystone-Inbox-Prototype.html` · **`-v2`** | Cross-project feed + task queue; act-to-clear vs read-to-clear enforced in code | ✅ audited + NY-fidelity v2 |
| **#4 System Admin** | `Keystone-Platform-Prototype.html` · **`-v2`** | Platform control plane; audited Enter-org assume-role | ✅ audited + NY-fidelity v2 |
| **#2 Org Admin** | `Keystone-OrgAdmin-Prototype.html` | Org-scope governance: Members/roles · Projects · Secrets · Connections · Billing · API keys · Audit | ✅ built + verified |

**`-v2` files** are the shadcn New York fidelity refinement of the three originals (behaviour identical; badges pill→`rounded-md`, radius ladder, elevation). Kept **alongside** the v1 baselines for comparison — not overwriting. Pick v1 or v2 per surface; say the word to promote v2 to the canonical names.

## How to run
`file://` is blocked in the test browser, so serve over local HTTP:
```
python -m http.server 8931 --directory "<this folder>"
```
Then open `http://localhost:8931/Keystone-<Surface>-Prototype.html`. Dark is default; toggle light via the account menu. Bump `?v=N` on the URL when re-testing after edits (OneDrive caches). Everything is mock — no backend, no network.

## The docs

| File | What |
|---|---|
| `ENGAGEMENT-REPORT.md` | The narrative record: deliverables, the 7 principles, every design decision + why, the method. **Rendered: `Keystone-Design-Report.pdf`** (with screenshot gallery). |
| `GAP-ANALYSIS.md` | "What Zaki's contract wants vs what I originally planned" — every finding (reversals · expansions · alignments) + the redesign action list. |
| `AUDIT-NOTES.md` | Six rounds: 3 formal ui-ux-audit + 3 refinement passes (feel · states · New York fidelity). Findings, severities, fixes, deliberate non-changes. |
| `GITHUB-COMMENTS.md` | Ready-to-paste replies for issues #1–#4 (coverage + open backend questions). **Drafted, not posted.** |
| `briefs/` | Zaki's four issue briefs (`ISSUE-1/3/4` verbatim; `ISSUE-2` synthesized — #2 had no brief text) + `STYLE-CONTRACT.md` + `SHADCN-NEWYORK-SPEC.md` (the fidelity reference) + `REDESIGN-BRIEF.md`. |

Pre-contract research (the plan + domain guide the briefs later simplified) lives in the sibling `../Keystone-PromptManagement/`.

## Shared contract (all four)
shadcn/ui New York · lucide icons · dark default + light parity (WCAG AA both) · **one interaction rule** (Tabs=mode · toolbar icon=whole object · ⋯=one item · centered dialogs only) · destructive friction scales (archive=confirm, delete=type-the-name) · ⌘K everywhere · every list has empty + skeleton-loading states · `prefers-reduced-motion` gated · every element maps to a named shadcn primitive. Built via the `brief-to-prototype` skill (research → build → audit).

## Open / owed
- **Post the GitHub comments** (drafted here) — ideally after committing the prototypes to branch `prompt-management-v1`.
- **Pencil frames** for Inbox + new-spec System Admin in `Keystone-Admin-Design.pen` (needs the Pencil app open).
- **Confirm the synthesized #2 brief** with Zaki.
- Subagent builds keep hitting the org monthly spend limit — inline builds are the reliable path until it's raised.
