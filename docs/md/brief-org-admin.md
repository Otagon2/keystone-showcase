# Issue #2 — Organization Admin — Design Brief (synthesized)

*Zaki assigned #2 with **no brief text**. This brief is synthesized from the Pencil/Admin design lineage (`Keystone-Admin-Design.pen` + `Keystone-Admin-Prototype.html`), the prior alignment research (`research-notes/ALIGNMENT.md`: the org "Settings" governance rail, the RBAC ladder, the credential taxonomy), and consistency with the sibling briefs #3 (Inbox) and #4 (System Admin). Build with shadcn/ui + lucide, default styling, dark developer-console theme — same shell as the rest of Keystone. Confirm scope with Zaki at the next sync.*

## 1. Thesis
Every Keystone organization needs someone below the platform operator (#4) but above the builder — the **org admin / owner** who governs their company's tenant: who's in it, what projects exist, which shared credentials and model connections it uses, and what it spends. Org Admin is the org-scope governance surface. It is the natural home of the things that *arrive in the Inbox as tasks* (invites, access requests, approvals) and the things the **System Admin operator drops into via the audited Enter-org** — so it must feel like a first-class governance console, not an afterthought settings page.

## 2. Who this is for
The **org_admin** (and **owner**) — governs one organization. Distinct from the platform operator (#4, sits above all orgs) and the builder (#1/#3, works inside projects). Reached via the account/scope switcher → **Organization**.

## 3. Core concepts
- **RBAC ladder:** **Owner > Admin > Editor > Viewer** (org-scoped roles). Owner can transfer ownership; Admin governs members/projects; Editor builds; Viewer reads.
- **Credential taxonomy (never conflate):** **Token** (who you are · Account) · **API key** (call one project's flows · project) · **Secret** (a value a flow uses · org · **write-once, masked forever**) · **Connection** (a provider bundle — endpoint + a Secret ref · org).
- **Safety rails:** default-deny; denied surfaces render a read-only panel **with a reason** (never blank/404); **secrets shown once**; every mutation writes an immutable **audit** row; destructive = type-to-confirm.
- **Scope:** Org → Project. Members and secrets are org-scoped; projects contain the builder's work.

## 4. Screens to design (sidebar nav under an "Organization" scope)
1. **Overview** — org identity (name · email domain · plan) · rollup stat cards (members · projects · this-month spend · seats used) · a **Needs attention** panel (pending approvals, expiring secrets, seats near limit) that deep-links in. *(loading/empty states designed.)*
2. **Members** *(the heart)* — table: avatar · name · email · **role** (dropdown Owner/Admin/Editor/Viewer) · status (active / invited) · last active · ⋯ (change role · remove). **Invite member** dialog (email + role). A **Pending invites** section (resend/revoke). A small **role matrix** note (what each role can do). Owner row is protected; "transfer ownership" is a guarded action.
3. **Projects** — table: name · owner · members · last activity · status · ⋯ (transfer owner · archive). **Create project** dialog.
4. **Secrets** — org secrets table: key · environment (base/prod/staging…) · scope (org) · created · ⋯ (rotate · delete). **Add secret** dialog: name + value + environment, with a **write-once / "you won't see this again"** warning; the value is **never displayed** after creation (shown masked `sk-…••••`). This is the Keystone-specific governance centerpiece — design it carefully.
5. **Connections** — provider bundles: provider (Anthropic/OpenAI/Ollama/…) · endpoint · credential (a **Secret name** ref, never the raw key) · status dot. **Add connection** dialog that ends in a green-check success (guided, not a config dump).
6. **Billing** — plan card · **spend ledger** (this month, trend) · **budget gate** (a cap with a warning state) · **rate card** (per-model In/Out $ per 1k tokens) · invoices list.
7. **API keys** — project/org keys: name · prefix · created · last used · ⋯ (revoke). **Create key** dialog (**shown once**).
8. **Audit** — org-scope audit lens (read-only): reverse-chronological stream + filter bar (actor · action · target · time) + export. Highlights role changes, secret create/rotate, project transfers, and **operator.entered_org** rows (the transparency FYI from #4).
9. **Approvals** *(optional card / small queue)* — pending org-level approvals (access requests, owner transfer, budget override) → Grant / Deny. Ties to the Inbox (#3); can render as a section in Overview or its own row.

## 5. Interaction rules (same as all of Keystone)
- Same shell (sidebar + header + content). A sidebar nav item switches surface · header holds the surface's primary action (Invite / Add secret / Create key…) + search · a ⋯ menu acts on one row · every pop-up is a centered dialog.
- Destructive friction scales: remove-member / revoke-key = confirm; delete-project / rotate-secret = type-to-confirm.
- Secrets and API keys are **shown once, masked forever** — the surface never re-displays a raw value.
- Roles are org-scoped; changing a role is immediate and audited.

## 6. Scope & non-goals
**In:** Overview · Members (roles + invite) · Projects · Secrets · Connections · Billing · API keys · Audit.
**Out (v1):** SSO/SCIM setup wizards (WorkOS-style — deferred), granular per-project ACL editors, notification preferences. Keep it the governance console, not everything.

## 7. Mock data (one coherent org)
Org **acme** (`acme.com`, Team plan, 42 members, 7 projects, $312 this month). Members: Amaan K (Owner), Maya Chen (Admin), Dana Ortiz (Editor), Leo Park (Editor, invited), plus a few Viewers. Projects: acme-support, internal-tools, churn-radar. Secrets: `OPENAI_KEY`, `ANTHROPIC_KEY`, `OLLAMA_ENDPOINT` (all masked). Connections: Anthropic (prod), OpenAI (prod), Ollama-local (dev). Rate card: claude-sonnet-4-6, claude-haiku-4-5, gpt-4.1, gpt-4.1-mini. One pending access request + one expiring secret in Needs-attention. Realistic names/emails/timestamps — never lorem ipsum.
