> **This is Zaki's published brief for issue #2** (fetched from `esberi/keystone#2`, 3 Jul 2026). It **supersedes the earlier synthesized brief** — when #2 originally arrived with no text we synthesized one and built an 8-surface prototype; that has now been **rebuilt to this real brief** (five surfaces: Members · Projects · Labels · Settings · Audit; Secrets/Connections/Billing/API-keys dropped per §10).

# Organization Admin — Product & Design Brief

*Audience: the designer mocking this surface, plus anyone deciding whether the shape is right. No prior Keystone knowledge assumed — terms are defined on first use. **Part 1** is the product brief (thesis, problem, principles, jobs-to-be-done with acceptance criteria, one end-to-end scenario, scope). **Part 2** is the screen-and-component reference to mock from. Build with **shadcn/ui + lucide icons, default styling**, dark "developer console" theme — the same shell and component vocabulary as the rest of Keystone.*

---

## PART 1 — PRODUCT BRIEF

### 1. Thesis

Keystone is a platform where developers ("builders") build AI features, grouped into **projects**, grouped under an **organization** (a company's account). Almost everything a builder does happens *inside* a project. **Organization Admin is the one place above projects** — where whoever runs the account manages the people, the portfolio of projects, the shared vocabulary, and the record of what happened. It is small on purpose: an admin should be able to add a teammate, hand off a project when someone leaves, set a company-wide policy label, and answer "who changed what" — quickly, and without wading through builder tooling. If it is calm, boring, and trustworthy, it's doing its job.

### 2. Who this is for

The **org admin** — the person who administers the company's Keystone account (often an engineering lead, platform owner, or ops person). They are *not* here to build prompts; they're here to **govern**: onboard and offboard people, oversee the set of projects, keep the organization's classification vocabulary consistent, and audit activity. They touch this surface rarely but at high stakes — the actions here affect *everyone* (removing a member, deleting an account, transferring a project). The design must make the routine effortless and the consequential deliberate.

### 3. The problem

As soon as more than one person and more than one project exist, questions appear that no project-level tool can answer:
- *A new hire needs access; a departing employee needs to be offboarded.* Who does that, and where?
- *A project owner just left — their project is now orphaned.* No project role can reassign it; someone above the project must.
- *Every team labels sensitive data differently* (`pii`, `PII`, `sensitive`) — there's no shared vocabulary, so org-wide policy and reporting break.
- *Something changed and no one knows who did it.* There's no cross-project record.

These are **org-spanning, high-authority** concerns. Keystone answers them in one role-gated area rather than scattering half-features into every project. The design bet: **a small, sharp set of governance surfaces beats a sprawling admin console** — five surfaces that each answer one of the questions above, and nothing more.

### 4. Design principles

1. **Above projects, not inside them.** Everything here spans the org or governs the container; nothing here belongs to a single project's day-to-day. If a project owner can already do it, it doesn't live here.
2. **Rare but weighty — so make consequence visible.** These actions are infrequent and affect many people. Routine steps are one click; irreversible or wide-blast-radius steps (remove a member, delete a project) carry proportional friction and a clear statement of who/what is affected.
3. **One org, no juggling.** An admin governs exactly one organization. No org-switcher, no multi-org context — the whole surface assumes a single org.
4. **Consistent with the rest of Keystone.** Same shell, same component patterns, same interaction rules (modes/whole-object/single-item; dialogs only; type-to-confirm for irreversible). An admin who knows the builder surfaces already knows how this behaves.
5. **The record is read-only and trusted.** The audit log is a lens on one shared event stream — never editable. An editable audit log is worthless.

### 5. Core concepts (needed to read every screen)

- **Organization (org)** — the company's account. It *contains* projects and *contains* members. There is exactly one, per admin. Every org has a fixed **email domain** (e.g. `@northwind.com`), set when the org is created.
- **System admin** — the Keystone platform operator who sits *above* all orgs. They **create the org, set its email domain, and seed its first org admin.** They are not a tenant member and don't appear in the org roster; org admins never see this role. (Org creation happens on a separate platform rail, not in this surface.)
- **Project** — a workspace where builders do their work (prompts, flows, etc.). An org holds many.
- **Member** — a person in the org. Has an **org role** and may belong to specific projects.
- **Org role** — `org_admin` (can govern the org — this surface) or `member` (an ordinary user). An org can have **multiple `org_admin`s** (recommended — no single point of failure), and the system guarantees **at least one always exists**. A separate **`can_create_projects`** flag lets a member start new projects. *(Distinct from **project roles** — Owner/Editor — which say what you can do *inside* one project. Org role = your standing in the company account; project role = your standing in a workspace.)*
- **Label (org taxonomy)** — a shared `key:value` organizing tag (e.g. `compliance:pii`) defined once for the whole org and available (read-only) to every project. The org tier of Keystone's label system.
- **Domain-bound membership** — everyone in the org shares the org's email domain. The org admin can only invite/onboard people at that domain; the domain itself is set by the system admin at creation and is **not editable** by the org admin. (In effect, the org *is* "the people at that domain.")
- **Audit event** — an immutable record of an action (who did what to what, when). Organization Admin shows the org-wide view of these.
- **Five surfaces:** **Members · Projects · Labels · Settings · Audit.** Everything here is one of these.

### 6. Jobs to be done

*Format: As an org admin, I want **[job]**, so that **[outcome]**. Criteria state what the design must let the admin accomplish; screen mechanics live in the appendices. **P0** = core governance, mock first; **P1** = important; **P2** = supporting.*

**JTBD-1 · Manage members and their roles. (P0)**
So that the right people have the right access, and no one is orphaned or locked out.
- I see the full org roster with each member's org role, project-creation permission, status, which projects they belong to, and which they **own**.
- I can **add** someone by email with an org role — but **only at the org's email domain**; other domains are rejected with a clear message. I can resend or **revoke** a pending invite.
- I can change someone's org role — including **promoting a member to `org_admin` or demoting an admin to `member`** — and toggle whether they can create projects.
- I can **remove** someone from the org (offboard): their access ends. Protected by type-to-confirm and recorded in Audit.
- **The org always has at least one admin.** The system blocks any remove / demote / self-demote that would leave zero admins.
- **Removing a project owner never silently orphans work:** if the member owns projects, removal requires reassigning them (inline) before it completes.
- **Membership is admin-controlled** — only an org admin adds, removes, or re-roles members; users cannot request their own deletion.

**JTBD-2 · Oversee the portfolio of projects. (P0)**
So that the org's projects stay owned, tidy, and accountable.
- I see every project with its owner, member count, status, and activity — including ones I'm not a member of.
- I can create a project, archive/unarchive one, and drill into any project for oversight.

**JTBD-3 · Reassign an orphaned project. (P0)**
So that a project survives its owner leaving.
- I can **transfer a project's ownership** to another member — the one capability no project-level role has.
- The transfer is confirmed and audited.

**JTBD-4 · Maintain a shared classification vocabulary. (P1)**
So that policy and reporting are consistent across every project.
- I can define org-wide `key:value` labels (e.g. `compliance:pii`, `data-class:confidential`) once.
- I can rename, merge, or delete org labels; projects can *use* them but not redefine them.
- I can see how widely a label is used before changing it.

**JTBD-5 · Answer "who changed what." (P1)**
So that I can investigate and demonstrate accountability.
- I see an org-wide, reverse-chronological stream of significant actions (member added/removed, role changes, project created/transferred/archived, label and settings changes).
- Each event shows actor, action, target, scope, before→after, and source (UI/API).
- I can filter, search, drill into detail, and export; I can never edit the log.
- I can drill from the org view down into a specific project's events.

**JTBD-6 · Set organization configuration. (P2)**
So that org-level defaults reflect the company.
- I can edit the org's name, default region, and data-residency. The **email domain is shown read-only** (set by the system admin at creation — it defines who can be a member).

### 7. Proof it hangs together — one scenario

**Priya runs the Keystone account for a fintech, Northwind.** A designer joins, an engineer leaves, and compliance wants consistent data labels.

Priya opens **Organization** from the user nav (that nav item is labelled simply "Organization" and appears only for org admins) (she has the `org_admin` role; no one else sees this area). In **Members** she **invites** the new designer at their `@northwind.com` address as a `member` (an invite to a personal Gmail would be rejected — the org is domain-bound), and toggles **can-create-projects** on so they can spin up their own workspace. The departing engineer needs offboarding, so in **Members** she **removes** them; because they own the *Fraud-Signals* project, the remove dialog won't complete until she reassigns it — she hands ownership to another engineer right there, types the name to confirm, and their access ends (audited). Before her own vacation she also **promotes** a co-lead to `org_admin`, so the account is never down to a single administrator. Compliance has asked for consistent tagging, so in **Labels** she defines `compliance:pii` and `data-class:confidential` as **org labels** — instantly available (read-only) in every project, ending the `pii`/`PII`/`sensitive` drift. A week later, asked "who removed access to Fraud-Signals?", she opens **Audit**, filters by that project and action, finds the event with its before→after and actor, and **exports** it for the compliance record.

Priya never wrote a prompt or touched a project's internals. She governed the *container* — people, portfolio, vocabulary, and record — from one small, role-gated area. That containment is the product.

### 8. Screens to design (→ appendix)
Data model (App. B) · Members (App. C) · Projects (App. D) · Labels (App. E) · Settings (App. F) · Audit (App. G) · Dialogs (App. H) · Placement cheat-sheet (App. I).

### 9. Interaction rules (consistent with all of Keystone)
- **Same shell:** Sidebar + Header + Content area. Organization Admin is reached from the **Account menu / user nav**, not the project stage nav — it's a different altitude.
- **List + panel/dialog** surfaces (no rich editors): each surface is a table with row actions; changes open a **centered dialog** (never a side drawer).
- **Destructive friction scales with risk:** remove-from-org / delete-project = **type-the-name to confirm**; archive / revoke-invite = simple confirm.
- **Audit is read-only** everywhere.
- **One item's verbs live in that row's `⋯` menu; surface-level actions (Add member, Create project, New label) sit in the Header.**

### 10. Scope & non-goals (with reasons)
**In:** the five governance surfaces — Members, Projects, Labels, Settings, Audit — each as a list + the panels/dialogs its actions need.

**Deliberately out (and why):**
- **Billing** — a real org surface, but deferred; not needed to prove the governance shape in v1.
- **SSO / SAML** — not in use now; adding identity-provider config would be premature.
- **Org-level Secrets and org-level version-tags** — those stay project-scoped for v1 (only the *label taxonomy* earns an org tier, because a shared classification vocabulary is genuinely org-wide).
- **A multi-org switcher** — an admin governs one org; multi-org is a different product shape.
- **Deep project internals** — Organization Admin *drills into* a project for oversight but doesn't replace the project's own Manage surfaces; governing a project's day-to-day belongs to its owner.
- **External / guest collaborators** — because membership is domain-bound, inviting people from other domains (contractors, partners) is **not supported in v1**; cross-domain collaboration is a deliberate future concern, not this release.
- **Setting the org's email domain** — that's a **system-admin** action at org creation (a separate platform rail), not an org-admin capability; the domain appears read-only in Settings here.

---
---

## PART 2 — APPENDICES (screen & component reference)

*shadcn/ui + lucide + default style, dark theme. Same shell as the rest of Keystone. Components named inline.*

### Appendix A — App shell & entry
```
DEFAULT (in a project)              IN ORGANIZATION (org_admin only)
┌────────────┬─────────────┐        ┌────────────┬─────────────┐
│[Project ▾] │  Header      │        │[Project ▾] │  Header      │
│            ├─────────────┤        │            ├─────────────┤
│ Build      │              │        │ Members    │              │
│ Evaluate   │              │        │ Projects   │              │
│ Operate    │  Content     │        │ Labels     │  Content     │
│ Observe    │  (project)   │        │ Settings   │  (org table) │
│ Learn      │              │        │ Audit      │              │
│ Manage     │              │        │            │              │
│ ─────────  │              │        │ ─────────  │              │
│ ◦ Inbox    │              │        │ ◦ Inbox    │              │
│ ◦ Settings │              │        │ ◦ Settings │              │
│ ───────────│              │        │ ───────────│              │
│ ◦ Organiz. │ ← org_admin  │        │ ◦ Organiz. │ ← active     │
│ ───────────│   only       │        │ ───────────│              │
│ [Account ▾]│              │        │ [Account ▾]│              │
└────────────┴─────────────┘        └────────────┴─────────────┘
```
*Left: a normal project view — the "Organization" entry sits in the bottom cluster (org admins only), fenced by separators. Right: after clicking it, the middle nav swaps from the six project stages to the five Organization surfaces; everything else stays put.*

- **Entry (locked):** a persistent **"Organization"** item in the Sidebar's **bottom user-nav cluster** — the same tier as **Inbox** and **User Settings** (cross-project, above-a-single-project items), *not* in the project stage nav. It is **visible only to members with the `org_admin` role**; everyone else never sees it. Because it's role-gated, set it off with a **`Separator` above and below** so it reads as a distinct, elevated entry. *(The feature is "Organization Admin"; the nav label users see is just "Organization.")*
- **The stage nav is identical for everyone.** Build/Evaluate/Operate/Observe/Learn/Manage are *project* surfaces gated by *project* role — being an org admin changes nothing there. The only sidebar delta for an org admin is the single "Organization" entry appearing in the bottom cluster.
- **Entering Organization swaps the middle nav.** On entry, the sidebar's middle section (the six project stages) is **replaced by the five Organization surfaces** — **Members · Projects · Labels · Settings · Audit** — as the in-area nav (a vertical list of five peers). You've changed altitude, so the nav's context changes with it; exiting (via the Project switcher or a "back to project" affordance) restores the stage nav.
- **Header:** breadcrumb `Organization · {org name} ▸ {surface}`; right = the surface's primary action (Add member / Create project / New label) + Search where the surface is a long list.
- shadcn components: `Sidebar`, `Separator`, `Breadcrumb`, `Table`, `Button`, `DropdownMenu`, `Dialog`, `Badge`, `Avatar`, `Input`, `Select`, `Switch`, `Command` (search).

### Appendix B — Entities (fields · values · relationships)

**B.1 Organization**
| Field | Type / values | Notes |
|---|---|---|
| `id` | string | one per account |
| `name` | string | |
| `email_domain` | string[] (v1: one) | the org's identity boundary; **set by system_admin at creation, read-only to org_admin**; all members must be at this domain |
| `region` | enum | default region |
| `data_residency` | enum | where data lives |

**B.2 Member (org roster entry)**
| Field | Type / values | Notes |
|---|---|---|
| `id` | string | |
| `name` / `email` | string | |
| `org_role` | enum: `org_admin` \| `member` | standing in the org |
| `can_create_projects` | boolean | a flag, not a role |
| `status` | enum: `active` \| `invited` | v1 has no separate "deactivated" state — offboarding = remove |
| `joined_at` / `last_active` | timestamp | |
| `projects` | project-ref[] (rollup) | which projects they belong to |

**B.3 Project (portfolio entry, org-admin view)**
| Field | Type / values | Notes |
|---|---|---|
| `id` / `name` | string | |
| `owner` | member-ref | reassignable via transfer-ownership |
| `member_count` | int | |
| `status` | enum: `active` \| `archived` | |
| `created_at` / `last_activity` | timestamp | |

**B.4 Org Label (taxonomy entry)**
| Field | Type / values | Notes |
|---|---|---|
| `key` / `value` | string | `key:value` form, e.g. `compliance:pii` |
| `description` | string (optional) | |
| `scope` | fixed: `org` | (system/project scopes live elsewhere; see the label model) |
| `usage_count` | int | how many entities across the org use it |
- Projects **consume** org labels read-only; resolution across scopes is **system > org > project**.

**B.5 Audit Event (shared stream; org lens)**
| Field | Type / values | Notes |
|---|---|---|
| `timestamp` | timestamp | |
| `actor` | member-ref | who |
| `action` | string | what (e.g. `member.added`, `member.removed`, `member.role_changed`, `project.transferred`) |
| `target` | ref | the affected entity |
| `scope` | enum: `primitive` \| `project` \| `org` \| `user` | which altitude |
| `before` / `after` | object (for changes) | |
| `source` | enum: `ui` \| `api` | |
- **Read-only.** One scope-tagged stream; Organization Admin shows the org lens and can drill into project-scope events.

**B.6 Relationships (ERD)**
```
Organization 1───* Member          (org roster; org_role + can_create_projects)
Organization 1───* Project         (the portfolio)
Organization 1───* OrgLabel        (org taxonomy; consumed read-only by projects)
Project      *───* Member          (project membership; project role lives in project-Manage)
Project       1───1 owner (Member) (reassignable — transfer-ownership, org-admin only)
(All surfaces) ───* AuditEvent      (one shared, scope-tagged, read-only stream)
```

### Appendix C — Members
- **Header:** breadcrumb + **Add member** button (right) + Search.
- **Table columns:** Name (avatar + name) · Email · **Org role** (`Badge`/inline `Select`: org_admin / member) · **Create projects** (`Switch`) · Status (`Badge`: active / invited) · Joined · Last active · Projects (count, hover → list) · **⋯**.
- **Row `⋯` menu:** View detail · Change org role (member ⇄ admin) · Toggle create-projects · Resend / **Revoke** invite (if invited) · — · **Remove from org** (red, last; type-name-to-confirm).
- **Add-member panel/dialog:** email + org role (`Select`) + optional create-projects toggle → Send. The email **must match the org domain** — inline validation shows the required domain (e.g. "must be @northwind.com") and **Send stays disabled** until it matches.
- **Member detail:** the fields above + the member's project memberships (read-only rollup).
- **Removal:** initiated only by the org admin. The confirm dialog **detects any projects the member owns** and requires reassigning each (a new-owner `Select` per project) *before* the **type-the-name-to-confirm** completes — removal never orphans a project. Access ends; recorded to Audit. No user-initiated deletion request exists.
- **Guardrails:** the system **blocks removing or demoting the last remaining `org_admin`** (the org must always have one). An admin may demote or remove *themselves* only when another admin exists.

### Appendix D — Projects
- **Header:** breadcrumb + **Create project** button + Search.
- **Table columns:** Name · Owner (avatar + name) · Members (count) · Status (`Badge`: active / archived) · Created · Last activity · **⋯**.
- **Row `⋯` menu:** Open (drill into the project) · **Transfer ownership** · Archive / Unarchive · — · (no delete here for v1 — deletion of a project's contents is governed inside the project).
- **Transfer-ownership dialog:** pick the new owner (`Select` of **any org member** — if they're not already a project member they're added as owner) + confirm; audited. *(Signature org-admin power — the only way to reassign a project.)*
- **Create panel:** name + initial owner.
- **Project detail (org-admin oversight view):** the portfolio fields + a link into the project's own surfaces; org-admin oversight, not a replacement for the project's Manage.

### Appendix E — Labels (org taxonomy)
- **Header:** breadcrumb + **New label** button + Search.
- **Table columns:** Label (`key:value` pill) · Description · **Usage** (count across the org) · **⋯**.
- **Row `⋯` menu:** Edit · Rename · **Merge into…** (fold another label into this one — sprawl control) · — · Delete (confirm; warn "N entities use this").
- **New / edit dialog:** key + value + description.
- **Note (shown on the surface):** "Org labels are available read-only to every project. Projects add their own labels in each project's Manage → Tags. Scopes resolve system > org > project."

### Appendix F — Settings (org config)
- A single **sectioned form** (like a settings page), minimal for v1.
- **Fields:** Org name (`Input`) · **Email domain (read-only** — set by the system admin at creation; shown for reference, not editable) · Default region (`Select`) · Data residency (`Select`).
- **Save** per section or a single Save; changes audited.
- **Email domain is read-only here** — it defines who can be a member and is owned by the system admin. Show it with a hint like "Managed by your Keystone administrator."
- *No billing, no SSO/SAML in v1 (note them as "coming later" placeholders only if useful, otherwise omit).*

### Appendix G — Audit (org lens, read-only)
- **Header:** breadcrumb + Search + **Export** button.
- **Layout:** a reverse-chronological **stream/table** + a **filter bar** (actor · action · scope · target · time range).
- **Row columns:** Time · Actor (avatar + name) · Action · Target · Scope (`Badge`: primitive/project/org/user) · Source (`Badge`: UI/API).
- **Row click → event detail:** full actor/action/target + **before → after** diff for changes.
- **Drill-down:** filtering by a project shows that project's scope events within the org view.
- **No create/edit/delete** — read-only. Export = download the filtered view.

### Appendix H — Dialogs (all overlays are centered `Dialog`s)
- **Add member** — email (validated against the org's email domain — Send disabled until it matches) + org role + create-projects toggle → Send.
- **Revoke invite** — simple confirm (cancels a pending, unaccepted invite).
- **Change org role** — role `Select` (member ⇄ admin) + confirm. **Disabled for the last admin** (can't demote the only one).
- **Transfer ownership** — new-owner `Select` (any org member; added as owner if not already a member) + confirm (states current → new owner).
- **New / edit org label** — key + value + description.
- **Merge label** — pick target label + confirm (states "X entities move to {target}").
- **Archive / unarchive project** — simple confirm (reversible).
- **Remove from org** — if the member owns projects, the dialog first lists them with a **new-owner `Select` each** (all must be reassigned to proceed); then **type-the-name-to-confirm**: "Removes {name} from {org} and ends their access. Cannot be undone." Red button disabled until reassignments done *and* name matches. **Blocked entirely for the last admin.**
- **Delete org label** — confirm; warn if in use ("N entities use this").

### Appendix I — Placement cheat-sheet
| Surface | Header actions | Row `⋯` menu | Overlays |
|---|---|---|---|
| **Members** | Add member · Search | View · Change role (member⇄admin) · Toggle create-projects · Resend/Revoke invite · **Remove from org** | add/role = dialog; **Remove = reassign owned projects + type-name-to-confirm**; last admin protected |
| **Projects** | Create · Search | Open · **Transfer ownership** · Archive/Unarchive | transfer/archive = dialog |
| **Labels** | New label · Search | Edit · Rename · Merge · Delete | new/edit/merge = dialog; Delete = confirm + usage warning |
| **Settings** | (Save) | — | — |
| **Audit** | Search · Export | (row → detail) | — (read-only) |

**The one rule underneath the table:** the in-area **sub-nav** switches surface · a **Header action** is a surface-level verb (Add member, Create project, New label) · an **`⋯` menu** acts on the one row you clicked · every overlay is a centered dialog · **Audit is always read-only.**

---
---

## PART 3 — ARCHITECTURE & BOUNDARIES

*The "why it's shaped this way" — for engineers and for anyone deciding whether the shape is right. Part 1 says what to build; Part 2 says how to draw it; Part 3 says why it hangs together and where the edges are.*

### What belongs here — two tests
A surface earns a place in Organization Admin only if it passes **both** (parallel to the project-Manage tests):
1. **Org-spanning** — about the org as a whole or spanning multiple projects (not scoped to one project or one user).
2. **Requires org authority** — gated by `org_admin`; a project owner or plain member can't do it.

If a project owner can already do it, it belongs in the project's own **Manage**, not here. This is why (e.g.) transfer-ownership lives here — no project role can reassign a project — while editing a prompt does not.

### Three identity scopes (don't conflate)
Identity is governed at three altitudes, each owning different things:
- **Organization Admin → Members** — org roles (`org_admin`/`member`, including **promote/demote**), the `can_create_projects` flag, and **membership** (add / remove, reassigning owned projects on removal). Domain-bound; **≥1 admin always exists**. *(Removal is admin-initiated only; users cannot request their own deletion.)*
- **project-Manage → Members** — only **project roles** (`owner`/`editor`) for one project.
- **User Settings** — your **own** profile.

A person has one org membership and may have many project memberships; the two are governed in different places on purpose.

### One audit stream, four lenses
There is **one scope-tagged event stream** across Keystone, not four logs. Each surface is a filtered lens:
| Scope | Lens surfaces at |
|---|---|
| **primitive** | version history / lineage in Build |
| **project** | project-Manage → Audit (one project's events) |
| **org** | **Organization Admin → Audit** (org-spanning; drills into project-scope) |
| **user** | User Settings → Your Activity (your own actions) |

Organization Admin → Audit is the org-scope lens — the broadest administrative view, and the only one that can drill into project-scope events across the whole org. Read-only everywhere (an editable audit log is worthless).

### Surface shape summary
| Surface | Structure | Read/Write | Owns | Distinct from |
|---|---|---|---|---|
| **Members** | list + panels | write | org roles, `can_create_projects`, membership (add/remove) | project-Members (org vs project roles) |
| **Projects** | list + panels | write | the portfolio; transfer-ownership | a project owner (the set vs one project) |
| **Labels** | list + panels | write | the org label taxonomy | project labels (org tier vs project tier) |
| **Settings** | single form | write | org name/region/residency (domain read-only) | project & user Settings (different scope) |
| **Audit** | list/stream | **read** | the org-scope lens | a lens on the shared stream, not a separate log |

### Boundaries locked
- **Single-org** — an admin administers one org; no multi-org switcher.
- **Domain-bound membership** — every member shares the org's email domain; the admin invites only within it. The domain is **set by `system_admin` at org creation**, read-only to the org admin. External/guest collaborators (other domains) are out of scope for v1.
- **Five surfaces** — Members · Projects · Labels · Settings · Audit. Billing, SSO/SAML, and org-level Secrets/version-tags are dropped/deferred; the org **label taxonomy** is in (Labels).
- **Membership is admin-controlled** — an `org_admin` adds, removes, and promotes/demotes users; **there is no user-initiated deletion request.** Removal reassigns any owned projects, is type-name-to-confirm, and is audited.
- **At-least-one-admin invariant** — the org must always have ≥1 `org_admin`; the system blocks any remove / demote / self-demote that would break it. Multiple admins are allowed and recommended (no single point of failure).
- **Audit is one scope-tagged stream, four lenses**; this is the org lens.
- **Three identity scopes** — org-Members · project-Members · User Settings — governed separately by design.
- **Not a lifecycle stage** — Organization Admin is a role-gated area off the user nav (altitude above projects), structurally like Manage but one level up; it is not Build/Evaluate/Operate/Observe/Learn/Manage.
