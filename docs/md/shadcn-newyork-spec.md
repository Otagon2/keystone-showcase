# shadcn/ui New York — implementation spec for Keystone prototypes

*Research-grounded reference for building authentic shadcn New York surfaces (dark developer-console theme). Sources: shadcn/ui docs (theming, components), the Default-vs-New-York comparison, and the framework's well-known component anatomy. This is the fidelity contract the redesign builds to — every prototype element should map 1:1 to a real shadcn/ui New York primitive so a developer can rebuild it verbatim.*

## 0. What "New York" is (vs Default)
Same tokens and colours as Default — a **different visual foundation**: **more compact, tighter spacing, smaller controls, and cards that carry a subtle shadow.** Default is softer/roomier; New York is the information-dense, "developer console" register — exactly what Keystone wants. Practical consequences:
- Controls are **smaller** (buttons/inputs `h-9` = 36px, not `h-10`).
- **Cards use `shadow-sm`** (Default cards are flatter). Depth is on-brand here.
- Slightly **tighter radius and padding**; `text-sm` (14px) is the workhorse size.
- Icons: we use **lucide** (Zaki's contract explicitly says lucide), rendered as inline SVG at 16px, `stroke-width:2`.

## 1. Tokens (dark theme — the default for these prototypes)
Keep the shadcn semantic token names; these are the zinc/slate-family values already in the prototypes (verified AA in the audit). Do not invent new names.
```
--background  --foreground        (app ground / primary text)
--card --card-foreground          (raised surfaces)
--popover --popover-foreground    (menus, dialogs)
--primary --primary-foreground    (primary button / emphasis)
--secondary --secondary-foreground
--muted --muted-foreground        (subtle fills / secondary text)
--accent --accent-foreground      (hover fills)
--destructive --destructive-foreground
--border --input --ring           (hairlines / field borders / focus ring)
--success --warning --info(running)  (semantic status)
--radius: 0.5rem                  (base; controls derive 6px, cards 8–10px, dialogs 12px)
```
- **Contrast floor (verified):** text ≥ 4.5:1, UI borders/icons ≥ 3:1, in **both** themes. `--input` must clear 3:1 (dark `#475569`, light `#7e889c`).
- HSL/token-driven — no ad-hoc hex outside the token block.

## 2. Radius ladder (the New York feel)
| Element | Radius |
|---|---|
| Buttons, inputs, badges, menu items, tabs triggers | **6px** (`rounded-md`) |
| Cards, tables, popovers, panels | **8–10px** (`rounded-lg`) |
| Dialogs | **12px** (`rounded-xl`) |
| Avatars, dots, pills-that-must-be-round | full |
Differentiating radius by elevation is a core New York signal — don't flatten everything to one value.

## 3. Component anatomy (build to these)
- **Button** — `h-9 px-4 text-sm font-medium rounded-md gap-2`, `transition-colors`, focus-visible ring. Variants: `default` (primary, subtle `shadow-sm`), `secondary`, `outline` (border-input, hover bg-accent), `ghost` (hover bg-accent), `destructive`. `sm`=`h-8 px-3`, icon=`size-9`. **One primary per view.**
- **Badge** — `inline-flex items-center rounded-md border px-2.5 py-0.5 text-xs font-semibold`. Variants default/secondary/destructive/outline. **Not a full pill.** Status badges = outline + a 6px coloured dot + label (never colour-alone).
- **Card** — `rounded-lg border bg-card text-card-foreground shadow-sm`; header `p-6 pb-2` (tighten to ~14–16px in dense screens), title `font-semibold`, content muted-foreground for secondary text.
- **Input / Select / Textarea** — `h-9 rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-sm`, placeholder `text-muted-foreground`, focus-visible ring.
- **Table** — `text-sm`; header row `text-muted-foreground` + `font-medium`, small uppercase optional; body rows `border-b`, hover `bg-muted/50`; numeric columns right-aligned `tabular-nums`.
- **Tabs** — `TabsList`: `bg-muted rounded-lg p-1`; `TabsTrigger` active: `bg-background text-foreground shadow-sm rounded-md`, inactive `text-muted-foreground`.
- **DropdownMenu / Popover** — `rounded-md border bg-popover shadow-md p-1`; items `rounded-sm px-2 py-1.5 text-sm`, hover `bg-accent`; destructive item `text-destructive`; separators `-mx-1 my-1 h-px bg-muted`.
- **Dialog** — overlay `bg-black/80` (fade-in); content centered `rounded-xl border bg-background shadow-lg` with a zoom+fade+slide entrance; title `font-semibold`, description `text-sm text-muted-foreground`.
- **Command (⌘K)** — dialog-hosted; input row with a leading search icon; grouped results with muted group headings; selected row `bg-accent`; footer hints. (Already implemented — keep.)
- **Sidebar** — the shadcn Sidebar block: header (project/brand) · grouped menu with `SidebarMenuButton` (active `bg-sidebar-accent`) · footer (account). Collapsible to an icon rail. Section labels `text-xs uppercase tracking-wide text-muted-foreground`.
- **Tooltip / Switch / Checkbox / Avatar / Separator / Skeleton / Sonner (toast)** — standard shadcn; Skeleton = `bg-muted` with a shimmer; toasts bottom-right cards.

## 4. Elevation ladder (two-part tinted shadows, light-from-above)
| Token | Use |
|---|---|
| `--elev-1` (tiny) | resting cards, table container |
| `shadow-md` | dropdown menus, popovers, hover-lift |
| `shadow-lg` | dialogs, command palette, floating panels |
Tint shadows with the dark ink, not pure black; make the tight part subtler as elevation grows. (Already tokenised in the prototypes — keep systematic.)

## 5. Typography
- **Inter / system sans** for all UI; **mono** (`ui-monospace`) only for IDs, version numbers (`v12`), token/cost figures, code, JSON.
- Scale: page title ~18–20px `font-semibold`; section ~14–15px `font-medium`; body 14px; small/labels 12–12.5px; micro/uppercase labels 10.5–11px `tracking-wide text-muted-foreground`.
- Hierarchy via **weight + muted-foreground**, not ever-bigger text. ≤3 text colours, ≤2 weights per view.

## 6. Motion (all reduced-motion-gated)
Fast (150–200ms), ease-out. Transitions on bg/color/border/shadow; button press `translateY(1px)`. Dialog/menu/command entrances (zoom+fade). Keep the prototypes' existing signature motion (streaming output, animated queue-resolution, skeletons, count-ups) — motion is behaviour, not skin, so it survives the fidelity pass untouched.

## 7. New York fidelity checklist (apply to every prototype)
- [ ] Badges are `rounded-md` (not full pills), `text-xs font-semibold`, status = outline + dot + label.
- [ ] Radius ladder honoured: 6 controls / 8–10 cards / 12 dialogs.
- [ ] Cards carry `--elev-1`; menus `shadow-md`; dialogs `shadow-lg`.
- [ ] Buttons/inputs ~`h-9`, `text-sm`, one primary per view, calm destructive until confirm.
- [ ] Tabs = muted list + active `bg-background shadow-sm`.
- [ ] Tables: muted header, `bg-muted/50` hover, right-aligned tabular numbers.
- [ ] Sans UI text; mono only for tokens/versions/cost/code.
- [ ] focus-visible ring (+offset) on every interactive element; all five states present.
- [ ] Contrast: text ≥4.5:1, borders/icons ≥3:1, both themes.
- [ ] Every element maps to a named shadcn primitive (comment it).
- [ ] Empty / loading (skeleton) / error states designed on every list.
