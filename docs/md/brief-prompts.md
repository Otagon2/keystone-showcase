# Issue #1 — Prompt Management — Product & Design Brief (Zaki, 2 Jul 2026, assigned @aman-esberi)

A self-contained brief for designing this feature. No prior Keystone knowledge assumed — every term is defined where it first appears. Part 1 is the product one-pager (background, the jobs to be done as user stories with acceptance criteria, and one end-to-end scenario). Part 2 is the screen-by-screen reference to mock from. Build with shadcn/ui + lucide icons, default styling, dark "developer console" theme.

## PART 1 — THE ONE-PAGER

### 1. Background — what is a "prompt," and who is this for?
Keystone is a platform where developers build AI features. The thing they write is a prompt — the instructions given to an AI model (e.g. "Read this customer message and decide if it's billing, tech, or account.").

Two kinds, and the editor differs slightly for each:
- **Chat prompt** — written as a conversation: a "system" instruction (standing orders) plus user/assistant turns. For assistants and chatbots.
- **Text prompt** — a single block of instructions, no conversation. For one-shot tasks (rewrite, summarize, classify).

The user is the "builder" — a developer using Keystone to build AI features for their company; technical but time-pressured. They want to write a prompt, try it, see if it's good, fix it, and ship it safely.

### 2. The problem
Builders juggle two bad options: playgrounds that let you try a prompt but not manage it (no history, no comparison, no safe way to ship), or heavy tools that manage prompts but make simple things slow. This feature is one place to write, run, and govern prompts — where govern means every change becomes a numbered version you can compare, ship, or undo.

### 3. Core concepts (needed to read every screen)
- **Variable** — a fill-in-the-blank slot written `{{message}}`. Typing it auto-creates a named input the prompt requires each run. (Like a form field the prompt defines for itself.)
- **Placeholder** (chat only) — a slot that injects a whole conversation, e.g. `chat_history`. Like a variable, but holds many messages.
- **Reference** — one prompt pulling in another's text, shown as a chip (e.g. `Safety Guidelines`) so common instructions are reused, not copy-pasted. Green if found, red if missing.
- **Version & version-tag** — each save = a new numbered version (v11, v12…). A version-tag marks which version is live in which environment, from a fixed set: production · staging · dev · draft. Moving a version up an environment = promoting. (Shown in the list as the Env badge.)
- **Tag (label)** — separately, the builder can add free-form labels to a prompt (e.g. `team:support`, `pii`, `experimental`) for organizing and filtering. These are just metadata — unrelated to environments. (Shown in the list as the grey Tags pills; edited via "Manage tags.") Throughout: "version-tag" = environment; "tag/label" = free-form organizing.
- **Run** — executing the prompt on some input to get the AI's output; inspectable afterward.
- **Three modes (shadcn Tabs)**: Edit (write) · Run (try + inspect) · Manage (govern). Everything happens in one of these.

### 4. Jobs to be done — user stories & acceptance criteria

**JTBD-1 — Find & organize my prompts.** As a builder, I want to see all my prompts in one place and act on any of them, so that I can navigate my work quickly.
- Given the Prompts list, I can see each prompt's name, kind (chat/text), labels (free-form tags), current version, environment (its highest version-tag), last-run status, and run count.
- I can search (⌘K), sort, and create a new prompt (choosing chat or text).
- From any row I can open, run, duplicate, rename, edit its labels, promote (a version-tag), export, archive, or delete it.
- Deleting requires typing the prompt's name to confirm; archiving is a simple confirm.

**JTBD-2 — Author a chat prompt.** As a builder, I want to write a conversational prompt with reusable pieces, so that I can build an assistant without rewriting common instructions.
- I can add/reorder/delete system, user, and assistant message segments.
- Typing `{{variable}}` auto-creates a matching input, listed in a side panel.
- I can add a conversation placeholder (e.g. `chat_history`).
- I can insert a reference to another prompt; it shows as a chip (green found / red missing).
- I can set the prompt's output type and see a live token/cost estimate.

**JTBD-3 — Author a text prompt.** As a builder, I want to write a single-block instruction with variables, so that I can make a reusable one-shot tool.
- I can write one text body with `{{variables}}` that auto-create inputs.
- I can set output type and see the live token/cost estimate.

**JTBD-4 — Try a prompt instantly while writing.** As a builder, I want to run a prompt on the spot without leaving the editor, so that I can iterate fast.
- From Edit, a quick-run panel lets me fill inputs, pick model/temperature/max-length, optionally attach an image/PDF (if the model supports it), and see the output.
- The quick run is temporary; I can Capture it to keep it.

**JTBD-5 — Run a prompt over many inputs at once.** As a builder, I want to run one prompt across a batch of inputs, so that I can check it on real data, not one example.
- In Run → Batch I can add input rows manually, paste, or load from a dataset.
- Running produces a results table (one row per input) with status, output, and metrics.
- I can capture all results.

**JTBD-6 — Compare versions/models side-by-side.** As a builder, I want to run two or three variants on the same inputs, so that I can pick the best before shipping.
- In Run → Compare I can set 2–3 columns, each its own version/model/settings.
- The same inputs run through all columns; each cell shows output + metrics.
- I can toggle a diff against the first column and mark a winner.

**JTBD-7 — Understand why a run behaved as it did.** As a builder, I want to inspect a single result's internals, so that I can debug without guessing.
- Clicking any result opens Inspect: the fully-assembled prompt sent, the raw request & response, token breakdown (input/output/cached), cost, latency, and a step timeline.

**JTBD-8 — Shorten an expensive prompt.** As a builder, I want to automatically compress a long prompt, so that it costs less and runs faster without me hand-trimming it.
- A compress mode shortens the prompt toward a target (keep-rate or token count).
- `{{variables}}` and references are always protected (never removed).
- I see before/after token counts, ratio, and cost saved, then Apply (creates a new version).

**JTBD-9 — Govern versions and ship safely.** As a builder, I want version history and controlled promotion, so that I can ship confidently and roll back.
- In Manage → Versions I can see history, diff two versions, restore an old one, and promote a version to dev/staging/production.
- Promotion is an explicit, confirmed action.

**JTBD-10 — Control access & lifecycle.** As a builder, I want to manage who can use a prompt and retire it safely, so that the project stays governed.
- In Manage I can set each member's role (Owner/Editor) and invite members.
- I can archive (reversible) or delete (type-name-to-confirm) the prompt.

### 5. End-to-end scenario (one relatable project)
Maya is building an AI support assistant for Acme's help desk: (a) triage incoming tickets, (b) reply in the right tone.

*Part 1 — the triage brain (a chat prompt):* Maya opens the Prompts list, clicks New → Chat prompt. In Edit she writes a system instruction ("You are a support assistant for Acme…"), inserts a reference to her shared Safety Guidelines prompt (a chip), adds a `chat_history` placeholder, and a user message: "Classify the intent of this message: `{{message}}` — billing, technical, account, or other." An input named "message" appears automatically. She hits quick-run on a sample ticket → "billing", and Captures it. She moves to Run → Batch, loads 50 real tickets, runs them, scans the results table. One ticket is misclassified — she clicks Inspect to see exactly what was sent and returned. She fixes the wording (v12), then in Run → Compare runs v11 vs v12 on the same 50 tickets, confirms v12 is better, marks it winner. In Manage → Versions she promotes v12 to production.

*Part 2 — the reply polish (a text prompt):* New → Text prompt: "Rewrite the following message in a `{{tone}}` tone, keeping the meaning: `{{message}}`." Inputs "tone" and "message" appear. She quick-runs with tone = "friendly" — good. The prompt got long and pricey, so she opens compress, targets ~60% keep (the `{{tone}}`/`{{message}}` slots stay protected), sees the cost drop, and Applies (a new version). She promotes it to staging for the team to test.

### 6. Screens to design
List (App. B) · Edit + quick-run (App. C) · Run + Inspect (App. D) · Compress (App. E) · Manage (App. F) · Dialogs (App. G) · ⌘K (App. H). Placement cheat-sheet: App. I.

### 7. Design rules
- Pop-ups are always centered dialogs — never side drawers. (Version history and label-editing are sections in Manage.)
- Delete = type the name to confirm; Archive = simple confirm.
- No stats bar on the list for now.
- One consistent pattern: a Tabs trigger = switch mode · toolbar icon = act on the whole prompt · ⋯ menu = act on the one item clicked.

### 8. Out of scope for these mockups
Separate Compare/Debug tabs · side drawers · a stats bar · the "Optimize with DSPy" flow (show the button in Manage → Settings only) · editors for other entity types (this is prompts only).

## PART 2 — APPENDICES (screen & component reference)
shadcn/ui + lucide + default style, dark theme. Components named inline.

### Appendix A — App shell (frame around every screen)
- Sidebar (`Sidebar`): top = Project switcher; middle = navigation (the product's stages — Prompts sits under "Build"); bottom = Account menu (avatar, name, light/dark toggle). Collapses to an icon rail.
- Header: left = breadcrumb; right = actions (change by screen).
- Content area: shows either the List or the Editor.
- shadcn components across screens: Button, DropdownMenu, Dialog, Command, Badge, Avatar, Separator, Input, Checkbox, Table, Tabs.

### Appendix B — Prompts list (the landing screen)
- Header actions (right): Search (magnifier, opens ⌘K) · Sort (dropdown) · New (dropdown → "Chat prompt" / "Text prompt").
- Table columns: Name · Kind (Badge: chat/text) · Labels (free-form tag pills, "+2" overflow) · Version (e.g. "v12") · Env (EnvBadge = version-tag: green production, amber staging, blue dev, grey draft) · Last run (colored status dot + "2m ago") · Runs (a count) · ⋯.
- Row hover: a Run button appears as the primary quick action.
- Row ⋯ menu (grouped, with dividers): Open · Run — Duplicate · Rename · Edit labels — Promote · Export — Archive · Delete (red text, last).
- Empty state: a single "New prompt" call-to-action.

### Appendix C — Edit mode (write the prompt)
- Header: breadcrumb `Prompts ▸ {name}` (double-click name to rename) + EnvBadge + a version dropdown (shows current version, lets you view an older one) · then icon toolbar · then the mode Tabs (Edit · Run · Manage — shadcn TabsList/TabsTrigger, active highlighted).
- Icon toolbar (lucide icons, grouped by dividers): chat/text toggle · preview · compress ‖ undo · redo ‖ quick-run ‖ import · export ‖ search.
- Canvas (left ~70%):
  - Chat prompt: a vertical stack of message segments (each labeled system / user / assistant, each an editable text area with `{{variable}}` words highlighted). Placeholders (like `chat_history`) show as a distinct inline chip. Buttons at the bottom to "add message" and "add placeholder." Reference chips appear inline where inserted.
  - Text prompt: one large editable text area.
- Inspector panel (right ~30%, stacked cards):
  - **Output** — what type the prompt returns (a ToggleGroup: Message / Text / JSON).
  - **Inputs** — the auto-generated list of variables/placeholders (each is a labeled "port").
  - **References** — list of referenced prompts (green = found, red = missing).
  - **Token budget** — model picker + a live count of how big/expensive the prompt is (a bar showing % of the model's limit + a cost estimate).
- Right-click a message segment → menu: move up/down · duplicate · delete · copy · insert reference · convert to/from placeholder.
- Quick-run panel (slides in when the quick-run icon is clicked, inside Edit): model + temperature + max-length controls · fields to fill each input · an attachments row (drag image/PDF chips, only enabled if the model supports them) · an output area · a Capture button. Before running, output area reads: "Run to preview. This run is temporary — Capture it to keep it."

### Appendix D — Run mode (try it and inspect results)
- Header: breadcrumb + a run dropdown (pick a past saved run) + icon toolbar + mode Tabs.
- Icon toolbar: Run · Stop ‖ [Single | Batch | Compare] toggle ‖ add-input · add-variant · load ‖ Capture ‖ search.
- **Single**: fields for each input → output + small metrics (tokens · cost · latency).
- **Batch**: a list of input-sets (Add row · Load from Dataset · paste) → a results table (one row per input: status dot, output preview, metrics); Capture all button.
- **Compare**: 2–3 columns, each with its own version/model picker; the same inputs run through all columns; each cell shows output + metrics; a "diff vs first" checkbox highlights differences; each cell can be marked winner; a "Run as Experiment" button (hands off to another feature).
- **Inspect** (opens when any result is clicked): the fully-assembled prompt sent · a Request/Response toggle (Tabs) · a token breakdown (input / output / cached) · cost · latency · a timeline/waterfall of the internal steps.
- Result-row ⋯ menu: Inspect · Capture this · Add to Dataset · Copy output · Mark winner.

### Appendix E — Compress view (inside Edit)
Opens when the compress icon is clicked; the canvas switches to this layout.
- **Method** — three selectable cards (LLMLingua · LongLLMLingua · LLMLingua-2), each with a short description.
- **Question field** — appears only for "LongLLMLingua."
- **Target** — a toggle between a keep-rate slider ("keep ~60%") and a target-token number; plus a "Preserve digits" checkbox; plus a Compress button.
- **Preview** — the prompt text with removed words struck-through/greyed, `{{variables}}` and reference chips highlighted as protected (never removed); above it a stats strip: Before / After / Saved / shrink-ratio / cost-saved.
- **Actions**: Cancel · Apply compression (disabled until a compression has been computed).

### Appendix F — Manage mode (govern the prompt)
A sectioned page (like a settings page):
1. **Details** — name · description · labels (free-form tags; add/remove chips).
2. **Versions** — a list (version number · its version-tag · when saved); each row can diff against another, restore, or promote; a row ⋯ menu with those actions.
3. **Evaluators** — a small panel shown only if this prompt is used to grade other prompts.
4. **Associated Datasets** — a read-only list of linked test datasets.
5. **Access** — a list of people (avatar · name · email · a role dropdown: Owner / Editor) + an Invite member button; note: "Roles are project-scoped."
6. **Settings** — an Archive card (neutral) and a Delete card (red border); plus an "Optimize with DSPy" button (just show the button).

### Appendix G — Dialogs (all pop-ups are centered Dialogs)
- **Rename** — one text field + Save.
- **Promote** — a dropdown to pick the target environment (dev/staging/production) + Confirm.
- **Export** — a JSON/YAML toggle + a preview area + Copy / Download buttons.
- **Archive** — plain confirm ("Archive {name}? You can restore it later." · Cancel / Archive).
- **Delete** — danger confirm: "This permanently deletes {name} and all its versions. This cannot be undone." + a field labeled "Type {name} to confirm" + a red Delete button disabled until the typed text exactly matches.

### Appendix H — ⌘K command palette (Command)
Grouped: **Go to** (any prompt by name · a section · a mode) · **Create** (New chat prompt · New text prompt) · **Do** (on the open prompt: Run · Compress · Compare · Promote · Version history · Edit labels · Export · Duplicate · Archive · Optimize) · **Insert reference** (search a prompt name to drop in a chip) · **Recent** (recently opened prompts).

### Appendix I — Placement cheat-sheet
| Screen | Toolbar icons (top) | Item ⋯ menu | Pop-ups |
|---|---|---|---|
| List | Search · Sort · New | row: Open · Run · Duplicate · Rename · Edit labels · Promote · Export · Archive · Delete | Rename/Promote/Export/Archive = dialog; Delete = type-name-to-confirm |
| Edit | chat/text · preview · compress ‖ undo · redo ‖ quick-run ‖ import · export ‖ search | message: move · duplicate · delete · copy · insert-ref · to/from-placeholder | — |
| Run | Run · Stop ‖ Single/Batch/Compare ‖ add-input · add-variant · load ‖ Capture ‖ search | result: Inspect · capture · add-to-Dataset · copy · winner | — |
| Manage | import · export ‖ search | version: diff · restore · promote · member: change-role · remove | Archive = simple; Delete = type-name-to-confirm |

The one rule: a Tabs trigger switches mode · a toolbar icon acts on the whole prompt · an ⋯ menu acts on the one item you clicked · every pop-up is a centered dialog.
