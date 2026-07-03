# Keystone Prototype Style Contract — binding for ALL surfaces (Issues #1–#4)

*This is the shared alignment contract for every prototype built in this batch. The client (Zaki) states in every brief: "Build with **shadcn/ui + lucide icons, default styling, dark 'developer console' theme — the same shell as the rest of Keystone**." That instruction supersedes the earlier Terminal-TTY skin. The visual foundation is `Keystone-Presentation\Keystone-Admin-Prototype.html` (a faithful shadcn New York implementation) — READ IT before writing any code and reuse its patterns.*

## 1. Non-negotiables
- **One self-contained HTML file per surface.** No frameworks, no build, no external network requests (fonts/icons inlined). Plain DOM + template literals.
- **shadcn/ui New York look, default styling, dark theme default** with a working light/dark toggle. Do NOT invent a new visual language.
- **lucide icons** — inline SVG via an `icon(name)` helper (copy the approach from the Admin prototype; add any missing icons as 24×24 lucide paths, stroke 2, `stroke="currentColor"`, `fill="none"`).
- **Same shell everywhere** (see §3). A reviewer flipping between the four prototypes must believe they're one product.
- **Mock data only, honest interactions** — no network calls; runs/actions animate with small staged delays; state lives in JS objects; nothing pretends to hit a backend.

## 2. Design tokens (copy verbatim from Keystone-Admin-Prototype.html)
Light `:root`:
`--background:#ffffff; --foreground:#020817; --card:#ffffff; --primary:#0f172a; --primary-foreground:#f8fafc; --secondary:#f1f5f9; --muted:#f1f5f9; --muted-foreground:#5b6472; --destructive:#dc2626; --border:#e2e8f0; --input:#7e889c; --ring:#0f172a; --success:#15803d; --warning:#b45309; --running:#2563eb; --radius:8px`
Dark `.dark` (the DEFAULT for these prototypes):
`--background:#020817; --card:#0b1220; --foreground:~#f8fafc; --secondary/muted: dark slate; --success:#4ade80; --warning:#fbbf24; --running:#60a5fa; --destructive:#f87171; borders: dark slate`
- Read the actual file for the full token block and copy it — do not approximate.
- Type: system/Inter-style sans for UI; `ui-monospace/Consolas` ONLY for IDs, versions (`v12`), tokens/cost numbers, code, JSON.
- Radius 8px; shadcn spacing scale (4/8/12/16/24px); `tabular-nums` on numeric columns.
- Status colors: success=green, warning=amber, destructive=red, running/info=blue. Env badges (Issue #1): production=green, staging=amber, dev=blue, draft=grey — exactly as Zaki's brief states.

## 3. The app shell (per Zaki's Appendix A, all briefs)
```
┌─────────────┬────────────────────────────────────────────┐
│  Sidebar    │  Header (breadcrumb + per-surface actions) │
│  (collapse) ├────────────────────────────────────────────┤
│             │  Content area                              │
└─────────────┴────────────────────────────────────────────┘
```
- **Sidebar** (project shell): top = **Project switcher** (dropdown, mock projects "acme-support", "internal-tools"); middle = stage nav — groups **Build** (Flows, **Prompts**), **Evaluate**, **Operate**, **Observe**, **Manage**; bottom = **account block** (avatar "AK" · name · light/dark toggle) — and for the Inbox integration, a **pending-actionable count badge** rides on the account block ("99+" cap). Collapsible to an icon rail.
- **System Admin exception** (Issue #4): the operator sees NO project shell — sidebar is just `KEYSTONE / Platform` + three items (Organizations · Platform Settings · Platform Audit) + Operator menu at bottom. Build it as a variant of the same sidebar component.
- **Header**: left breadcrumb; right = the surface's actions (each brief specifies its own).
- **Navigation**: hash router + `SCREENS` registry (the Admin prototype's `go(key)` / `location.hash` / `render()` pattern). Every screen reachable by URL hash.

## 4. Interaction rules (Zaki's "one rule", verbatim)
- A **Tabs** trigger switches mode · a **toolbar icon** acts on the whole object · an **⋯ menu** acts on the one item clicked · **every pop-up is a centered Dialog** — never a side drawer.
- Destructive friction scales: **Delete = type-the-name-to-confirm (button disabled until exact match)**; Archive/Suspend = simple confirm.
- **⌘K command palette** (shadcn Command look): groups per the brief; also opens via the header search icon; Esc closes.
- Every list screen ships its **empty state** (and where the brief calls for it, loading/error).
- Keyboard: ⌘K palette, Esc closes dialogs/panels, ⌘↵ runs (where a Run exists).

## 5. Code conventions
- Name the shadcn primitive in a comment above each component's CSS block (e.g. `/* shadcn: Dialog */`) — the dev-handoff contract.
- CSS custom properties only — no hardcoded colors outside the token block.
- One `<script>`; state objects at top (`DATA`), pure render functions, event delegation.
- `prefers-reduced-motion` gate on animations. AA contrast in both themes.
- Cache-buster note: when testing with Playwright, always navigate with `?v=N`.
