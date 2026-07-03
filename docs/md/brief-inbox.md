# Issue #3 — Inbox — Product & Design Brief (Zaki, 2 Jul 2026, assigned @aman-esberi)

Build with shadcn/ui + lucide icons, default styling, dark "developer console" theme — the same shell as the rest of Keystone.

## PART 1 — PRODUCT BRIEF

### 1. Thesis
Keystone is a platform where developers ("builders") build AI features across many projects. Things happen to a builder while they work elsewhere: a run they kicked off finishes, a flow pauses waiting for their approval, someone invites them to a project, a credential is about to expire. The Inbox is the one place all of that lands — the builder's personal, cross-project stream of what happened and what needs them. Its job is to make sure nothing that needs a decision gets lost in the noise of things that are merely FYI. Get that one distinction right and the Inbox is trustworthy; get it wrong and it's either a firehose people ignore or a task list that drops approvals on the floor.

### 2. Who this is for
The builder — any Keystone user, in any role. The Inbox follows them, not a project. They come here to triage — clear the noise, act on what's blocking, and jump back to where the work is. A glance should answer "is anything waiting on me?", and acting should take one click without leaving the feed.

### 3. The problem — the real tension
Two kinds of thing want to reach a builder:
- A **notification feed** (run finished, comment posted, member joined) — you clear it by reading it. Feeds are good at volume, bad at accountability.
- A **task queue** (approve this, accept this invite, renew this token) — you clear it only by acting. Queues are good at accountability, bad at ambient awareness.

Ship two separate surfaces and builders miss things in whichever they check less. Merge them naively and an approval gets "marked read" and scrolls away unacted. The design bet: one merged surface is right — but only if actionable items can never be cleared by reading. The load-bearing mechanic is a hard distinction between **informational** (read-to-clear) and **actionable** (act-to-clear), with the actionable queue always one filter away.

### 4. Design principles
1. **One surface, two "done"s.** Informational items clear by being read; actionable items clear only by being acted on (or expiring). Never let a decision be dismissed as if it were a notification.
2. **Blocking work never scrolls away.** Actionable items stay visually distinct and are always reachable via a single "Needs action" filter.
3. **Aggregator, not owner.** The Inbox doesn't define alerts, approvals, or invites — those are owned elsewhere (Observe, the flow shell, Members). Every item deep-links back to its real home.
4. **Act in place, then leave — with enough context to decide.** Every item carries the salient facts inline (who, which entity/version, relevant values, before→after). A deep-link is always present for more — never as the minimum to decide.
5. **Cross-project by nature.** It aggregates from all a builder's projects at once; per-project filtering matters precisely because the default is everything.

### 5. Core concepts
- **Inbox** — a builder's personal, cross-project stream of notifications and tasks. Reached from the user nav; the same surface everywhere.
- **Item class (the primary axis)** — every item is either **informational** (FYI; cleared by reading) or **actionable** (needs a decision; cleared only by acting).
- **Informational item** — e.g. a run finished, an experiment completed, a member joined, a comment posted. State: unread / read.
- **Actionable item** — e.g. an approval, an invite, an expiring token, an admin request. State: pending / approved / rejected / expired — a task lifecycle.
- **HITL approval** — "human-in-the-loop": a running flow pauses and asks a person to Approve/Reject before continuing. The decision resumes the run.
- **Needs-action filter** — isolates just the actionable, still-pending items. The most important filter on the surface.
- **Deep-link** — every item links to the exact source it came from.
- **Aggregator** — the Inbox owns none of these concepts; it's where things defined elsewhere land for you.

### 6. Jobs to be done (P0 = mock first)

**JTBD-1 · See what needs me, instantly. (P0)**
- One filter — "Needs action" — shows only actionable, still-pending items, in isolation.
- Actionable items are visually distinct in the unified feed and don't scroll away when unread-but-seen.
- A badge on the account block (bottom of the Sidebar) shows the pending-actionable count ("99+" capped), visible from every screen.

**JTBD-2 · Act on a blocking item without leaving. (P0)**
- From an actionable item I take its decision inline: Approve / Reject (HITL), Accept / Decline (invite), Renew (token), Acknowledge (alert).
- Taking the decision resolves the item; marking it read does not; a pending actionable can't be dismissed or archived — only the decision (or expiry) clears it.
- Once resolved, the item leaves the Needs-action queue but stays in the feed showing its outcome ("Approved by you"), and can then be archived.
- I can open the item's source in context.

**JTBD-3 · Stay aware of what happened. (P0)**
- A unified feed shows everything, reverse-chronological, informational and actionable together.
- Informational items clear by reading; mark read/unread, mark-all-read, archive (→ an Archived view). Archive applies to informational and resolved actionables — never a pending one.

**JTBD-4 · Cut the feed down. (P1)**
- Filter by Needs action · All · Unread · Archived; by type (Approvals · Alerts · Invites · Access requests · Comments · Runs · System); by project. Search.

**JTBD-5 · Understand an item in full. (P1)**
- Opening an item shows full context (what, where, who/what triggered it, when, severity, due/expiry) plus inline action(s) and a deep-link.

**JTBD-6 · Not miss time-sensitive items. (P1)**
- Time-bound actionables show due/expiry; severity (info/warning/critical) visible for alerts and some approvals.
- An expired item leaves the Needs-action queue but stays in the feed as expired/missed — it never masquerades as "done."
- In Needs action, items sort by urgency (soonest due / highest severity first), not just recency.

**JTBD-7 · Trust the queue reflects reality. (P1)**
- Items resolved at their source (another approver, API, another device) auto-resolve here; no zombie tasks.
- If a decision is offered to several people, the first decision resolves everyone's copy.

### 7. Scenario
Sam returns from a meeting; the account-block badge shows a count. Sam clicks it → Needs action: three items. (1) A HITL approval — a data-cleanup flow paused before writing to production; Sam reads inline context, clicks Approve; the item flips to approved, drops out of the queue, stays in the feed as "Approved by you". (2) An invite to a third project → Accept. (3) A token expiring in 2 days (warning badge) — Sam leaves it pending; it stays in the queue, unlost. Switching to All, Sam skims: two runs finished, a teammate @-mentioned them on a prompt → opens it, deep-links to reply in context; clears the rest with mark-all-read. Thirty seconds, inbox triaged, nothing blocking forgotten.

### 8. Screens
Feed list + filters (App. C) · Item detail + inline actions (App. D) · Empty & zero states (App. E) · Placement cheat-sheet (App. F) · Message catalog (App. G).

### 9. Interaction rules
- Same shell: the Inbox's ambient presence is a badge on the account block (bottom of the Sidebar); the full surface opens from that badge / an Inbox row in the account menu. **No separate Inbox nav item.**
- Two "done" semantics, never confused. Mark-read never resolves an actionable item.
- Inline actions on the item; deep-link for depth.
- A pending actionable can't be dismissed or read away.
- The queue reflects source state; no zombie tasks.
- The Inbox owns nothing.

### 10. Scope & non-goals
**In:** unified feed, item-class distinction, Needs-action queue, inline actions for core actionable types, filtering/search, item detail with deep-links, badge/count.
**Out:** notification preferences · email/push delivery · defining alerts/approvals/invites (owned elsewhere) · bulk approve (decisions are per-item, deliberate; mark-all-read is informational-only) · threaded conversations · grouping/digests (deferred consciously).

## PART 2 — APPENDICES

### Appendix A — App shell & entry
Same shell as everywhere — Sidebar · Header · Content area. The Inbox changes only the Header and the Content layout; **the Sidebar is untouched** (stage nav does NOT swap).
```
┌─────────────┬────────────────────────────────────────────┐
│ [Project ▾] │  Inbox   [Needs action·All·Unread·Archived]│  ← Header
│ ★ Pinned    │              Type▾ Project▾ ⌘K ·Mark all read
│ ───────     ├──────────────────┬─────────────────────────┤
│ Build       │  list of items   │   selected item         │
│ …           │  (feed)          │   (detail + actions)    │  ← Content
│ Manage      │                  │                         │
│ ───────     │                  │                         │
│ [avatar ⦿3] │                  │                         │
└─────────────┴──────────────────┴─────────────────────────┘
```
- **Ambient entry = a badge on the account block** (no standalone Inbox nav item): pending-actionable count rides on the avatar/name block, "99+" capped. Clicking the avatar/name → account menu; clicking the badge (or an "Inbox" row in the menu) → opens the Inbox in the Content area.
- **Header**: Breadcrumb `Inbox` (user-scoped, no project prefix); `Inbox ▸ {item}` when one is open. Primary segments (Tabs/ToggleGroup): **Needs action · All · Unread · Archived**. Right: Type filter (dropdown) · Project filter (dropdown) · Search (⌘K) · Mark all read (informational only).
- **Content**: a list + reading-pane split (Mail-style, layout only). Two-pane on wide screens; collapses to list-then-detail on narrow.
- Default: lands on All; clicking the account badge deep-links straight to Needs action.
- shadcn: Sidebar, Breadcrumb, Tabs/ToggleGroup, DropdownMenu, Command, ResizablePanel, Card/Table, Badge, Button, Avatar, Separator, Tooltip.

### Appendix B — Entities
**B.1 Inbox Item**: `id` · `item_class` (informational | actionable — drives clear-semantics) · `type` (approval | alert | invite | access_request | comment_assigned | mention | prompt_promoted | org_role_changed | operator_entered_org | run_finished | run_failed | experiment_done | token_expiring | system; actionable types: approval, invite, access_request, comment_assigned, token_expiring) · `state` (informational: unread|read · actionable: pending|approved|rejected|expired) · `title/summary` · `source` (project + entity) · `actor` (user|system) · `severity` (info|warning|critical) · `created_at` · `due_at/expires_at` (optional) · `actions` · `deep_link`.

**B.2 Item actions by type**: approval → Approve·Reject (resumes/stops the paused run) · invite → Accept·Decline · token_expiring → Renew (deep-link)·Acknowledge · alert → Acknowledge·View · access_request → Grant·Deny · comment_assigned → Open·Resolve · mention/run_*/experiment_done/system → View (informational).

**B.3 Relationships**: User 1—* InboxItem; InboxItem *—▶ Source (via deep_link); sources owned elsewhere (Observe→alert · flow shell→approval · Members→invite); Approval ⇄ paused Run. Deleting an item never deletes its source.

### Appendix C — Feed list + filters
- Filters live in the **Header**, not the feed body. "Needs action" is the emphasized segment; All is default.
- Feed rows (reverse-chronological): icon (by type) · title/summary · source (project · entity) · actor · timestamp · (if actionable) a state chip + due/expiry · (if alert/approval) a severity dot.
- **Actionable rows are visually distinct** (accent bar/background + inline action buttons right-aligned) — they read as tasks, not FYIs, and stay surfaced even once seen.
- Informational rows are quieter; unread ones carry an unread dot.
- Row inline actions: actionable rows show decision buttons directly; informational rows show View + mark read/unread.
- Resolved actionables keep an outcome chip, drop out of Needs action, remain in All until archived.
- **Needs-action ordering: by urgency** (soonest due_at / highest severity) then recency.
- Archived view holds archived informational + resolved items.
- Live updates: new items increment the badge; a critical alert/new actionable may show a brief toast.
- Selecting a row opens detail in the right pane.

### Appendix D — Item detail + inline actions
- Header: type + title + state chip; severity and due/expiry if present.
- Body: full context/summary; source breadcrumb (project ▸ entity); actor; timestamp.
- Actions: the same inline decision(s), prominent; **Open in context** (deep-link).
- Actionable emphasis: decision buttons are the primary CTA; a note if time-bound ("Expires in 2 days"). After a decision, the detail reflects the resolved state ("Approved by you · just now").
- Informational (and resolved) detail: context + View/Open; mark read/unread; archive. A pending actionable offers no dismiss/archive.

### Appendix E — Empty & zero states
- All caught up: a calm "You're all caught up."
- Needs-action empty: "Nothing needs you right now."
- First-run: a one-line explainer of what the Inbox collects.

### Appendix F — Placement cheat-sheet
| Element | Where | Behavior |
|---|---|---|
| Inbox entry | badge on the account block (no standalone item) | badge = pending actionable count ("99+"); click → full surface (Needs action) |
| Filters | Header (segments + Type/Project dropdowns) | Needs action · All · Unread · Archived · type · project |
| Inline decision | actionable row + detail | Approve/Reject · Accept/Decline · Renew · Acknowledge |
| Mark read / mark-all-read | informational rows + header | clears informational only |
| Deep-link (Open in context) | detail (and row overflow) | jumps to the source surface |

The one rule: informational = read-to-clear; actionable = act-to-clear; the Needs-action filter keeps the task queue visible inside the feed; the Inbox aggregates, never owns.

### Appendix G — Message catalog (per source)
Envelope (every item): **Title** (one line, naming the entity) · **Context body** (enough to decide without leaving: actor · target entity + version/run · relevant values · before→after · comment excerpt · requested role) · **Deep-link** (always) · **Actions** · **Metadata** (actor · source · timestamp · severity · due/expiry).

**Prompt Management**: `mention` (info; "{actor} mentioned you on {prompt} v{n}"; excerpt; → thread; View) · `comment_assigned` (actionable; "{actor} assigned you a comment on {prompt} v{n}"; Open · Resolve) · `prompt_promoted` (info; "{actor} promoted {prompt} v{n} to {env}"; env before→after; → Versions; View).
**Org Admin / Members**: `invite` (actionable; "{actor} invited you to {project}"; offered role; Accept · Decline) · `access_request` (actionable; requester · project · requested role · note; Grant · Deny) · `org_role_changed` (info; old → new role · changed by; View).
**System Admin**: `operator_entered_org` (info; "A platform operator accessed {org}"; "governance session, audited"; → Organization → Audit; View).
Coming as their surfaces ship: `approval` (flow shell), `alert` (Observe), `run_finished/run_failed/experiment_done` (Operate/Evaluate), `token_expiring` (credentials). *(For the mock, seed the feed with realistic examples of ALL types including approval/alert/run/token so the surface is demonstrable.)*

## PART 3 — ARCHITECTURE & BOUNDARIES (the load-bearing part)
- **Two "done" semantics under one roof** via `item_class`: informational → read-to-clear; actionable → act-to-clear (pending → approved/rejected/expired); mark-read changes nothing on actionables. The badge counts pending actionables, not unread noise.
- **Aggregator, not owner**: definition-vs-delivery split (the rule is defined in Observe, it fires into the Inbox). Deleting/dismissing an item never touches its source.
- **HITL approvals are first-class**: paused run → actionable item → decision → resume/stop. A mis-cleared approval would leave a run hung or wrongly resumed — this is why act-to-clear can't be compromised.
- **Source-state sync (no zombie tasks)**: first decision wins across approvers; renewed/accepted-elsewhere auto-resolves; items are a live view of source state.
- **User-scoped, cross-project** — the third altitude alongside project surfaces and Organization Admin; sits beside User Settings in the user nav.
- Needs-action sorts by urgency, not recency — it's a queue, not a timeline.
