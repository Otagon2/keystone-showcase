# Sunday demo script — Datasets & Prompt Management

**Audience:** Zaki + Safi. **Goal:** show Keystone is easier and better than Braintrust,
Arize AX, and Phoenix across the whole user journey — new user *and* power user, from
first run to large datasets to iterating over time.

**Spine (one sentence):** a new user creates a first dataset in seconds → it scales to
10,000 rows and stays instant → a messy import is handled calmly → they curate + version →
run a prompt that scores inline with cost → promote the winner and watch the trend lift.

Live: https://aman-esberi.github.io/esberi.github.com/keystone/

---

## Beat 1 — First run (new user, empty project)  ⟶ *ease of use*

**Click:** Datasets → the "New here?" banner → **Start with sample data** → Upload CSV →
**"use the sample GST FAQ data"** → map (auto-detected) → **Import**.

**Say:** "A brand-new user has a dataset in one click — no concepts to learn first. The
import shows a **live preview** and maps columns to *your own names* (Input / Expected /
Metadata / Tags). We even print how long it took." (Import summary reads `… in 0.4s`.)

**Why we win:**
- **Braintrust** — good drag-bucket import, but ~2 min and its nav is hidden; a new user
  has to find things.
- **Arize / Phoenix** — a single cryptic `reference` field defaulting to
  `attributes.output.value` (OTel jargon); testers called it "confusing / 10 minutes."
- **Keystone** — labeled buckets + live preview + defaults to the user's column names, and
  a one-click sample so there's zero setup friction.

## Beat 2 — Large dataset (power user)  ⟶ *stays usable at scale*

**Click:** open **GST FAQ** (10,000 records) → type in **Search rows** ("e-way bill") →
watch the count update instantly (*Showing 1–50 of 417, filtered from 10,000*) → add a
**Filter** chip (Provenance is captured) → toggle a **Column** → **Sample 100** → header
checkbox → **Select all 417 matching** → Tag reviewed.

**Say:** "This is the moment tools fall over. Ten thousand rows, and search / filter / paging
/ select-all-across-pages are all instant. Power users get a real working grid — not a
frozen table or a 'load more' dead end."

**Why we win:** neither competitor demonstrates a calm large-dataset grid; this is our
clearest flex. Honest counts ("filtered from 10,000"), select-all-across-pages with an
explicit escalation, and a Sample mode for quick scans.

## Beat 3 — All the messy instances  ⟶ *nothing surprises you*

**Click:** New dataset → Upload CSV → **"try a messy export"** → scroll to the amber panel.

**Say:** "Real data is messy. Keystone tells you *before* you commit: no Expected column,
empty inputs will be skipped, duplicate IDs upsert instead of duplicating — and none of it
blocks you. Compare that to a silent mis-import."

**Why we win:** Arize/Braintrust surface little here; a wrong-provider run even failed
*silently* in our teardown. We make every edge state calm and legible.

## Beat 4 — Governance over time  ⟶ *experience as data grows*

**Click:** GST FAQ → **Versions** (v1 1,200 → v2 1,584 → v3 10,000) → **Diff** → a row's
**Origin** chip → the source trace. Then **Manage** → schema shows "N rows excluded from
runs" (the generated missing-expected rows).

**Say:** "Datasets are governed: numbered **versions**, movable **tags** (golden → v3),
**labels**, per-row **provenance** and an **origin** link back to the trace a row was
captured from. Experiments pin a version, so a finished result never changes under you."

**Why we win:** deeper, more legible governance than either tool — and the
capture-from-traces loop ("production failure → permanent test") is unique.

## Beat 5 — Prompt Management golden path  ⟶ *iterate and prove improvement*

**Click:** Prompts (paginated library w/ search + filter + saved views) → **GST QnA** →
**Run** (Batch) → evaluators score **inline** (per-cell pass/partial + **cost** + latency) →
**Save as prompt version** → **Promote to experiment** → Experiments → the **pass-rate-over-
runs** trend shows v12 → v13 = **+16pp**.

**Say:** "Editing, running, scoring, and cost all live on one screen — then promotion and a
regression trend prove the prompt actually got better. That's the loop, made legible."

**Why we win:** Braintrust's playground is ephemeral (you can lose a run); Arize's is
engineer-first. Ours keeps the loop framing, persists runs, and shows cost + score per cell.

---

## Closing line
"Braintrust is clean but hides things; Arize/Phoenix are complete but dense and raw-first.
Keystone gives you their power with a legible loop, real governance, and it stays calm from
row one to row ten-thousand." See `day4-vs-competitors.md` for the capability-by-capability
comparison.
