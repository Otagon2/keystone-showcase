# Issue #4 — System Admin — Product & Design Brief (Zaki, 2 Jul 2026, assigned @aman-esberi)

Build with shadcn/ui + lucide icons, default styling, dark "developer console" theme — the same shell as the rest of Keystone. Part 3 (architecture & boundaries — the control-plane / tenant-isolation model) is load-bearing.

## PART 1 — PRODUCT BRIEF

### 1. Thesis
Keystone is a multi-tenant platform: many organizations (companies) run on one deployment, and each org's data is invisible to the others. Someone has to sit above all of them — to bring a new org into existence, hand it its first administrator, and keep the platform healthy — without being able to reach inside and read any tenant's work. System Admin is that seat. It is the smallest, highest-altitude surface in Keystone: it operates the platform, not the content. Done right, it makes provisioning an org a two-minute job and makes the operator's power auditable and bounded.

### 2. Who this is for
The system admin — the operator of the Keystone deployment. There is exactly one, and the account is defined in deployment configuration, not created in the app. Not a member of any org, not tied to any email domain; doesn't author prompts or flows. Their job: create orgs, seed their first admins, watch the portfolio, and step in — visibly and rarely — when an org needs operator help.

### 3. The problem
- A new company needs to start using Keystone — someone above must create the org and appoint its first admin.
- Each org must be sealed from the others — yet the operator must keep the platform running and occasionally help an org that's stuck.
- An org loses access (its last admin is gone) — someone above must restore an admin.
- Regulators ask "can your operator read our data?" — the answer must be a defensible "no, and here's the audit trail for the rare times we act."

The design bet: a tiny control-plane surface with a hard tenant boundary beats a powerful god-mode console.

### 4. Design principles
1. **Control plane, not data plane.** The system admin operates containers (orgs) and platform config — never tenant content by default.
2. **The tenant boundary is a feature.** Any crossing is deliberate and logged, so isolation is provable, not promised.
3. **Power is bounded and visible.** Even the operator can't read raw secrets. Every cross-tenant action is loudly audited. Break-glass, not god-mode.
4. **Bootstrapped, not managed.** The single operator identity comes from deployment config — nothing to "create" or self-serve.
5. **Consistent with the rest of Keystone.** Same shell and patterns; entering an org reuses the existing Organization Admin surface.

### 5. Core concepts
- **Platform** — the whole deployment: one platform, many orgs.
- **Organization (org)** — a company's account; contains members and projects; sealed from other orgs.
- **System admin** — the single operator, defined in deployment config (email + password via env var). On login they see ONLY the Platform surface — no project shell, no stage nav. Exactly one at any time.
- **Control plane vs. data plane** — control = create/suspend/delete orgs, seed admins, platform settings, platform audit. Data = the prompts/flows/secrets inside an org (never ambiently visible).
- **Seed the first admin** — at org creation the operator names the org's first org_admin by email; invited; org sits in `pending_setup` until they accept and set a password, then `active`. An org is never live without a reachable admin.
- **No authoring** — the operator never writes prompts/flows/labels. Not read-only though: governing an org (members, transfers) is writing — allowed under assume-role.
- **Enter an org (assume-role)** — explicit, re-authenticated, audited: the operator steps into a specific org's Organization Admin surface to govern it. Governance only — never authoring, never raw secrets. Works on active AND suspended orgs.
- **Three surfaces**: Organizations · Platform Settings · Platform Audit.

### 6. Jobs to be done

**JTBD-1 · Provision a new organization. (P0)**
- Create an org with a name and its email domain.
- Seed its first org_admin by email; org is `pending_setup` until first login, then `active`.
- See which orgs are still pending_setup; resend the first-admin invite.
- The org appears in the portfolio immediately; creation and activation are audited.

**JTBD-2 · Oversee all organizations. (P0)**
- Every org with metadata only — name, domain, member count, project count, status, activity — not contents.
- Search, sort, open an org's platform-level detail.

**JTBD-3 · Manage an organization's lifecycle. (P1)**
- Suspend/reactivate (reversible): members can't sign in, endpoints pause, data retained.
- Delete — only after suspended. Soft: a ~30-day recovery window, then purged. Requires re-authentication + type-the-name; shows blast radius (N members, N projects, all data).
- All lifecycle changes audited.

**JTBD-4 · Step into an org to help govern it. (P1)**
- Enter (after re-authenticating) → org_admin powers in that org only (its Organization Admin surface).
- The session and every action are loudly audited as operator cross-tenant activity.
- Recovery: while entered, promote an existing member to org_admin, or invite one (domain-bound). Then leave.
- Govern (members/projects) but never author prompts/flows, never see raw secrets.

**JTBD-5 · Keep the operator account secure. (P0)**
- MFA on login. Destructive and cross-tenant actions re-authenticate (delete org, enter org), even mid-session. Sessions time out; the operator's own activity is in the audit stream.

**JTBD-6 · Answer "what happened across the platform." (P1)**
- The platform-scope audit lens: org created/suspended/deleted, admins seeded, operator cross-tenant entries, settings changes. Filter, search, drill in, export; never editable.

**JTBD-7 · Configure the platform. (P2)**
- Minimal settings (name, available regions/defaults). Operator identity read-only (lives in deployment config).

### 7. Scenario
The operator provisions Northwind: Create org → name, domain @northwind.com, seed Priya as first org_admin → `pending_setup` until Priya's first login → `active`. Weeks later Northwind is locked out (last admin left): operator opens Northwind → Enter → re-authenticates → (as org_admin in Northwind only, audited) promotes a member to org_admin → leaves. Decommissioning: suspend an old org → later delete (re-auth + type-name) → 30-day recovery window → purge. Security asks for proof: Platform → Audit, filter operator cross-tenant entries, export.

### 8. Screens
Organizations (App. C) · Platform Settings (App. D) · Platform Audit (App. E) · Dialogs (App. F) · Placement cheat-sheet (App. G).

### 9. Interaction rules
- Same shell: Sidebar + Header + Content. Reached via a "Platform" item in the user nav, visible only to the system admin (in this mock: it IS the whole app after operator login).
- List + panel/dialog surfaces (no rich editors); changes open centered dialogs.
- Delete-org = type-the-name; suspend = simple confirm.
- Entering an org is explicit and audited — a deliberate assume-role, not a view toggle.
- Audit is read-only.

### 10. Scope & non-goals
**In:** Organizations, Platform Settings, Platform Audit, the audited Enter-org assume-role.
**Out:** ambient access to tenant content · reading raw secrets (write-once/mask-forever is absolute, operator included) · managing system-admin accounts (config-defined; no in-app reset) · billing/SSO/usage metering · authoring anything.

## PART 2 — APPENDICES

### Appendix A — App shell & entry
The operator's shell is NOT the project shell. On login they land directly in Platform — no project switcher, no stage nav.
```
WHAT A NORMAL USER SEES            WHAT THE SYSTEM ADMIN SEES
┌───────────┬──────────────┐      ┌───────────┬──────────────┐
│[Project ▾]│  Header      │      │  KEYSTONE │  Header      │
│ Build     │              │      │  Platform │              │
│ Evaluate  │              │      │           │              │
│ …         │  Content     │      │ Organizat.│  Content     │
│ Manage    │  (project)   │      │ Settings  │  (platform   │
│ ───────── │              │      │ Audit     │   table)     │
│ ◦ Inbox   │              │      │           │              │
│ [Account ▾]│             │      │ [Operator ▾]│            │
└───────────┴──────────────┘      └───────────┴──────────────┘
```
- Sidebar (Platform): three-item nav — **Organizations · Platform Settings · Platform Audit** — with an **Operator menu** at the bottom (identity read-only, sign out).
- Header: breadcrumb `Platform ▸ {surface}`; right = the surface's primary action (Create org) + Search.
- shadcn: Sidebar, Breadcrumb, Table, Button, DropdownMenu, Dialog, Badge, Input, Select, Command.

### Appendix B — Entities
**B.1 Platform (singleton)**: `name` · `available_regions` (enum[]) · `system_admin_email` (read-only, from deployment config).
**B.2 Organization (platform view — metadata only)**: `id/name` · `email_domain` · `status` (pending_setup | active | suspended | pending_delete) · `member_count/project_count` (rollups) · `first_admin` (email) · `created_at/last_activity`.
**B.3 Audit Event (platform lens)**: timestamp · actor · action (org.created, org.suspended, org.deleted, admin.seeded, operator.entered_org, …) · target · scope · before→after · source. Read-only.
**B.4 Relationships**: Platform 1—* Organization; Organization 1—1 first_admin; System Admin (singleton, config) —▶ Platform; System Admin —(Enter, audited)—▶ Organization Admin[org]; all actions —* AuditEvent.

### Appendix C — Organizations (the portfolio)
- Header: breadcrumb + **Create org** button + Search.
- Empty state (fresh deployment): a single prominent "Create your first organization" CTA.
- Table columns: Name · Email domain · Members (count) · Projects (count) · Status (Badge: pending-setup / active / suspended / pending-delete) · Created · Last activity · ⋯.
- Row ⋯ menu: Open (platform detail) · Enter org (re-auth → assume org_admin, audited) · Resend first-admin invite (only while pending_setup) · — · Suspend / Reactivate · — · Delete org (red, last; only enabled once suspended; type-name-to-confirm).
- Create-org dialog: name + email domain + first admin email → Create (org starts pending_setup; invite sent).
- Org platform-detail: metadata + lifecycle actions + an **Enter org** button. If pending_setup: shows invited admin + resend. If pending_delete: shows recovery deadline + **Restore**. Explicitly NO view of the org's prompts/flows/secrets; note: "Contents are tenant data — enter the org (audited) to govern it; contents are never shown here."

### Appendix D — Platform Settings
A single sectioned form, minimal: Platform name (Input) · Available regions (Select/multi) · Operator identity (read-only email; hint "Defined in deployment configuration; change it there"). Changes audited. No password field.

### Appendix E — Platform Audit (broadest lens, read-only)
- Header: breadcrumb + Search + Export.
- Reverse-chronological stream/table + filter bar (actor · action · scope · target · time).
- **Highlights the operator's own cross-tenant actions** — `operator.entered_org` and anything done under assume-role are first-class, filterable events (the "prove isolation" view).
- Row → event detail (before→after). No create/edit/delete. Export = the filtered view.

### Appendix F — Dialogs (centered)
- **Create org** — name + email domain + first-admin email → Create.
- **Enter org** — re-authenticate (password/MFA), then confirm: "You'll act as an administrator in {org}. This session is audited." → Enter.
- **Suspend / Reactivate** — simple confirm; suspend states the effect: "Members can't sign in and running endpoints pause. Data is retained; you can reactivate anytime."
- **Delete org** — only on a suspended org. Re-authenticate, then type-the-name: "Suspends into deletion: {org}, its {N} projects, and all data enter a {30}-day recovery window, then are permanently purged. Recoverable until {date}." Red button gated on exact match. (Soft delete.)
- **Restore org** — (pending_delete, within window) simple confirm → returns it to suspended.

### Appendix G — Placement cheat-sheet
| Surface | Header actions | Row ⋯ menu | Overlays |
|---|---|---|---|
| Organizations | Create org · Search | Open · Enter org (re-auth) · Resend invite (if pending) · Suspend/Reactivate · Delete org (suspended only) · Restore (if pending-delete) | create/suspend = dialog; enter/delete = re-auth; Delete = suspended-first + type-name (soft) |
| Platform Settings | (Save) | — | — |
| Platform Audit | Search · Export | (row → detail) | — (read-only) |

The one rule: the platform sub-nav switches surface · Header actions are surface-level verbs · ⋯ acts on one org · every overlay is a centered dialog · entering an org is an audited assume-role · Audit is read-only.

## PART 3 — ARCHITECTURE & BOUNDARIES (load-bearing)
- **Control vs data plane**: control (orgs, seeded admins, settings, audit) = standing power; data (each org's prompts/flows/projects/members/secrets) = none by default; only via audited Enter (governance, not authoring; never raw secrets). "System admin is org admin for every org" is true ONLY through the audited Enter — never as an ambient flat view.
- **Assume-role (Enter an org)**: explicit (one org at a time) · scoped (org_admin powers in that org only — the existing Organization Admin surface, reused) · audited (`operator.entered_org` + every action) · bounded (governance only; no authoring; no raw-secret reads). While entered, show a prominent operator banner ("Acting as administrator in {org} — audited") with a Leave button.
- **Identity & bootstrap**: exactly one system admin, config-defined (root-account posture); not created/listed/reset in-app; not a tenant member; not domain-bound.
- **Tenant isolation & secrets**: orgs sealed from each other and from the operator's ambient view; secrets write-once/mask-forever, operator included.
- **One audit stream — the platform lens**: the broadest lens on Keystone's single scope-tagged stream (primitive → project → org → platform); the only lens that surfaces operator cross-tenant actions.
- **Boundaries locked**: MFA + re-auth on destructive/cross-tenant · operator sees only Platform · provisioning has a real handoff (pending_setup → active) · delete is guarded (suspend-first, soft, re-auth + type-name) · "no authoring" = no prompts/flows/labels; governance writes allowed · three surfaces · platform altitude (above orgs; not a project or lifecycle stage).

## NOTE for the prototype (from Amaan)
Issue #2 (Organization Admin) has no brief text yet — only the assignment. The Enter-org flow should land on the existing Organization Admin surface as designed in `Keystone-Admin-Design.pen` / `Keystone-Admin-Prototype.html` (org overview, members, projects). Include a governance-only Org Admin view (members list with promote-to-org_admin, projects list) shown under the operator banner, reusing those existing patterns — enough to prove the assume-role recovery journey end-to-end.
