# GitHub comment drafts — issues #1–#4 (esberi/keystone)

*Ready to paste. Each comment assumes the prototype HTML files get committed to the repo (branch `prompt-management-v1` per the huddle) or attached to the issue — adjust the file link line to wherever they land. Written from Amaan's voice.*

---

## Issue #1 — Prompt Management

Prototype is ready: **`Keystone-Prompts-Prototype.html`** — a single self-contained file (no build, no backend, mock data only). Open it in any browser; everything is clickable.

**Coverage vs the brief** — all ten JTBDs are mockable and reachable:
- **List** (App. B): every column incl. Env badges (green/amber/blue/grey as specified), labels with +N overflow, hover-Run, the grouped row ⋯ menu, search→⌘K, sort, New ▸ chat/text, empty state.
- **Edit** (App. C): chat = reorderable system/user/assistant segments with `{{variable}}` highlighting that auto-creates Inputs, `chat_history` placeholder chip, reference chips (green found / red missing — see `escalation-detector`), right-click segment menu; text = single block editor. Inspector: Output toggle (Message/Text/JSON), Inputs, References, live Token budget (model picker + % bar + cost). Quick-run panel with your exact placeholder copy, attachments row gated on model support, Capture.
- **Run** (App. D): Single / Batch (add rows, paste, Load from Dataset → 50 tickets with one deliberately misclassified) / Compare (2–3 columns, per-column version+model, diff-vs-first, mark winner, Run as Experiment) / Inspect (assembled prompt, Request/Response tabs, token breakdown incl. cached, cost, latency, step timeline).
- **Compress** (App. E): three method cards, question field only on LongLLMLingua, keep-rate ⇄ target-token toggle, preserve digits, struck-through preview with protected `{{vars}}`/references, Before/After/Saved/ratio/cost strip, Apply → new version.
- **Manage** (App. F) + all dialogs (App. G, delete gated on typed name) + ⌘K (App. H, all five groups).

**Decisions to ratify (each traceable to the brief or the current backend):**
1. **Every save auto-versions** (blur after an edit → "Saved as v13"). Versioning as a byproduct of iteration, not a chore.
2. **Restore is non-destructive** ("Restore v9 → creates v13"), matching the shipped behavior in `keystone-main`.
3. **Kind is fixed at creation** — the chat/text toolbar toggle explains itself and points to Duplicate, since the brief fixes the editor shape per kind.
4. Dark is default; light theme is fully implemented and AA-checked.

**Questions before backend wiring:**
- The brief writes variables as `{{message}}`, but `keystone-main` ships f-string `{message}` (single braces, `prompt.py` regex). Which syntax should the UI teach? (Prototype currently follows the brief.)
- The Prompt library entity today stores a single flat `template` — a chat prompt's `messages[]`/`system_message` has nowhere to live. Is a `body` shape per kind the committed direction?
- No price table exists in the repo (token counts only) — is a rate card planned, or should cost stay estimate-labelled?

---

## Issue #2 — Organization Admin

The issue arrived with **no brief text**, so — to avoid blocking — I synthesized one from the Pencil/Admin design lineage (`Keystone-Admin-Design.pen`) and the prior alignment research (the org "Settings" governance rail, the Owner>Admin>Editor>Viewer ladder, the credential taxonomy), then built the prototype. **Please sanity-check the synthesized brief** (`briefs/ISSUE-2-org-admin.md`) against your intent — happy to adjust.

Prototype is ready: **`Keystone-OrgAdmin-Prototype.html`** — self-contained, clickable, a sibling of the other three at shadcn New York fidelity. Eight org-scope surfaces:
- **Overview** — org identity, rollup stats (members/projects/spend/seats), a "Needs attention" panel (pending access request + expiring secret) that deep-links.
- **Members** (the heart) — role dropdowns (Owner/Admin/Editor/Viewer; Owner protected), Invite dialog, a Pending-invites section, and a role matrix. Ties to the Inbox invite/access-request items in #3.
- **Projects** — owner/members/activity, transfer-owner, archive, create.
- **Secrets** — the governance centerpiece: **write-once, masked-forever** (`sk-…••••`, never re-displayed), environment pills, an expiry warning, rotate = type-to-confirm.
- **Connections** — provider bundles (endpoint + a Secret *reference*, never a raw key), add-connection ending in a green check.
- **Billing** — plan, spend ledger + budget gate (402 at cap, Inbox alert at 80%), a per-model rate card, invoices.
- **API keys** — shown-once, masked after; revoke.
- **Audit** — read-only org lens, with the `operator.entered_org` transparency row (the #4 tie-in).

Consistent with the others: same shell, centered dialogs, destructive friction scaled, ⌘K, empty + skeleton-loading states, dark default + light toggle. Note the boundary the surface states on itself: **secrets/keys are shown once and masked forever, owner included.**

---

## Issue #3 — Inbox

Prototype is ready: **`Keystone-Inbox-Prototype.html`** — self-contained, clickable, seeded with the Sam scenario plus volume (16 items across three projects, all catalog types).

**The load-bearing rule is enforced in code, not just drawn:**
- Informational = read-to-clear; actionable = **act-to-clear**. Mark-read/mark-all-read never touches an actionable; a pending actionable has no archive/dismiss affordance (trying explains why).
- Deciding resolves in place: the item leaves Needs-action, stays in All with an outcome chip ("Approved by you"), and the account-block badge decrements live — the badge counts pending actionables, not unread noise.
- **Needs-action sorts by urgency** (severity, then soonest due), All is reverse-chronological.
- One approval is seeded to **auto-resolve ~20s after you open the Inbox** ("Resolved by Dana Ortiz — another approver"): the first-decision-wins / no-zombie-tasks property, demonstrated.
- An expired approval renders "expired · missed" — out of the queue, never masquerading as done.

Also per brief: no Inbox nav item (entry = badge on the account block / account menu), Mail-style split list + reading pane inside Content with the sidebar untouched, header segments + Type/Project filters + Mark all read, full message catalog (mention with excerpt, prompt_promoted with env before→after, operator_entered_org transparency FYI, etc.), all three empty states, deep-links ("Open in context") from every item.

**Deferred exactly per your non-goals:** preferences/channels, bulk approve, threads, grouping/digests.

---

## Issue #4 — System Admin

Prototype is ready: **`Keystone-Platform-Prototype.html`** — self-contained, clickable, starting from the operator login (email + password + MFA step, identity noted as deployment-config-defined).

**Control plane, with the tenant boundary enforced:**
- Operator shell exactly per App. A: no project shell — three surfaces (Organizations · Platform Settings · Platform Audit) + read-only Operator menu.
- **Organizations**: portfolio with all four lifecycle states (pending-setup / active / suspended / pending-delete), search, stats row, the exact row ⋯ menu — with **Delete disabled until suspended** ("suspend first" hint inline), Resend-invite only while pending-setup, Restore only while pending-delete. Create-org seeds the first admin and lands in pending-setup with the invite toast. Org detail shows metadata only, plus the note: "Contents are tenant data — enter the org (audited) to govern it."
- **Enter org (the showpiece)**: re-auth (password/MFA) → "You'll act as an administrator in {org}. This session is audited." → the shell switches to that org's Organization Admin under a persistent warning banner with Leave. The **northwind recovery journey** works end-to-end: flagged "no reachable admin" → enter → promote a member to org_admin → flag clears → leave.
- **Delete is guarded** as specified: suspended-first, re-auth, type-the-name with blast radius, soft with a 30-day window and Restore.
- **Platform Audit**: read-only stream where every action taken in the session appends live events; `operator.*` rows are highlighted and there's an "Operator activity" one-click filter — the "prove isolation" view. Row → before/after detail. Export = the filtered view.
- Secrets are never displayed anywhere, operator included; the entered view states the no-authoring boundary explicitly.

**One demo affordance to note:** ⌘K includes "Simulate fresh deployment," which empties the portfolio to show the day-zero "Create your first organization" state.

---

*All four prototypes: shadcn/ui New York tokens + lucide-style icons, default styling, dark developer-console theme (light toggle included), same shell anatomy, centered dialogs only, type-name-to-confirm on destructive actions, ⌘K everywhere. Audited across six rounds — 3 formal ui-ux-audit + 3 design-refinement passes ending in a shadcn New York fidelity pass (see AUDIT-NOTES.md); WCAG AA contrast and keyboard focus verified in both themes. The three original surfaces also have `-v2.html` fidelity refinements alongside their baselines.*
