# MOBA Tactical Board

Community tool to help Honor of Kings community teams prepare tactical plans more
easily. Heroes and maps belong to Honor of Kings — we do not claim any rights to
them. This project only provides the license to the `index.html` file, which is the
script that handles hero placement and plotting.

More information is available on our Discord page:
<https://discord.gg/DfW5kPKsv5>

**Counters and Synergies** are gathered from publicly available data from late 2025
up to 2026-10-25 pro-team matches. Depending on the hero, not all of its matches
might have been available at that time. Some deviation might appear due to patches.
For instance, during the first data gathering Pei, Chicha and Guan Yu were
dominating the META. During the last patch, Wang Wei and Dunshan were added to the
game. Also, Global and Chinese servers have slightly different METAs.

---

## Table of contents

1. [What the tool does](#what-the-tool-does)
2. [Repository contents](#repository-contents)
3. [Using the tool](#using-the-tool)
   - [Hosted version (recommended)](#hosted-version-recommended)
   - [Running locally](#running-locally)
4. [Using the app](#using-the-app)
5. [Data files explained](#data-files-explained)
6. [How hero names are matched](#how-hero-names-are-matched)
7. [Customising the board](#customising-the-board)
8. [Preparing the project for another game](#preparing-the-project-for-another-game)
9. [Troubleshooting](#troubleshooting)
10. [Known limitations](#known-limitations)
11. [Credits, license & contact](#credits-license--contact)

---

## What the tool does

- **Draft planner** — drop hero tokens onto any MOBA map, annotate with arrows,
  paths, circles, sticky notes and free-form text, and step through multiple
  timeline states per map.
- **Counter / synergy lookup** — click any two heroes in "Compare" mode to see
  their historical win rate together (synergy) or against each other (counter).
- **Win prediction** — the sidebar shows a rough blue-vs-red win probability
  derived from the average synergy and counter rates of the placed heroes.

Everything runs **client-side**: no server-side code, no telemetry. The only
network activity is loading the hero/map images and the two CSV data files
from wherever the site is hosted.

---

## Repository contents

| File / folder             | Purpose |
|---------------------------|---------|
| `index.html`              | The whole application — UI, styles and logic, single file. |
| `data.js`                 | Hero and map registry. Defines `window.HEROES` and `window.MAPS`. |
| `hok_hero_counters.csv`   | Hero-vs-Hero win-rate table (counter/matchup data). |
| `hok_hero_synergies.csv`  | Hero-with-Hero win-rate table (same-team synergy data). |
| `HEROES/`                 | Hero portrait images (e.g. `Agudo_Icon.webp`). |
| `MAPS/`                   | Map background images (e.g. `hok.png`). |
| `LICENSE`                 | License for the `index.html` file. See the file for terms. |
| `readme.md`               | This document. |

There is no build step and no installation. Copy the files to any static host
and open `index.html`.

---

## Using the tool

### Hosted version (recommended)

The tool is **already live on GitHub Pages** — no setup, no downloads:

> **<https://kovaslt.github.io/moba-tactical-board/>**

Just open the link in a browser. Everything (heroes, maps, CSVs, auto-save
via `localStorage`) works out of the box. This is the intended way to use
the tool and is what you should share with teammates.

### Running locally

Only needed if you want to modify the code, replace the data files, or
experiment with the UI. The board is a static site, so any static server
works:

```bash
# from the folder that contains index.html
python -m http.server 8000
```

Then visit `http://localhost:8000/` in your browser. Alternatives if you
don't have Python handy:

```bash
npx serve .          # Node.js
php -S localhost:8000  # PHP
```

Or open the folder directly with **VS Code's Live Server** extension, or
push to a fork and enable GitHub Pages on your fork.

#### `file://` caveat

Opening `index.html` directly from disk (double-clicking the file) *mostly*
works — the UI, images and localStorage persistence all function — but the
browser blocks the auto-load of the two CSV files under `file://`. In that
case the sidebar will show **"No matchup data loaded"** and Compare / Win
Prediction will show no results. Use a static server (above) or the
**Load CSV files** button in the sidebar to add the CSVs manually.

---

## Using the app

### Layout

- **Left sidebar** — map picker, hero library, matchup-data status, win
  prediction, quick help.
- **Top bar** — timeline of states (per map), toolbar (drawing modes, colours,
  active team), save/load/clear buttons.
- **Board** — the map with your tokens, shapes and annotations.

### Timeline / states

Each map carries an ordered list of **states**. Use `+ Add State (Duplicate)`
to branch from the current state (useful for showing "before" vs "after" a
fight). Double-click a state tab to rename it. `‹` / `›` or clicking a tab
switches between them.

### Drawing modes

| Mode        | Pointer action on empty board                     |
|-------------|---------------------------------------------------|
| `move`      | drag tokens, stickies, text boxes                 |
| `arrow`     | drag to draw a line with arrowhead                |
| `path`      | drag to draw a plain line                         |
| `circle`    | drag to draw a gather/area circle                 |
| `note`      | click to drop a sticky note                       |
| `text`      | click to drop a text box (double-click to edit)   |
| `compare`   | click two tokens to compute their matchup         |
| `erase`     | click any shape / token / note to delete it       |

Right-clicking a token removes it regardless of the current mode.

### Compare mode

Two slots are tracked internally. Click a token, then a second token:

- **Same team** → the panel prefers the **synergy** table; falls back to
  counter if that pair has no synergy record.
- **Opposite teams** → prefers the **counter** table; falls back to synergy.

The badge under the percentage tells you which table the number came from
(⚔ Counter / 🤝 Synergy / No data), and the line beneath shows the match
count. Clicking the currently selected token clears the comparison.

### Win prediction

Averages the **synergy** rate across all pairs *within* each team and the
**counter** rate across all pairs *against* the other team, then normalises
blue vs red. Pairs with no data are skipped. The result is a rough,
uncalibrated heuristic — treat it as a sanity check, not a probability.

### Persistence

Every edit auto-saves to `localStorage` (debounced). The header buttons let
you export / import a `.json` snapshot — useful for sharing a draft or moving
it to another machine. **Clear board** wipes only the current state, not the
whole map.

> On GitHub Pages, auto-save persists per browser and per origin, so a user's
> boards are private to them. Nothing is uploaded.

---

## Data files explained

### `hok_hero_counters.csv`

One row per `(hero, opponent)` pair:

```
hero,opponent,matches,wins,win_rate
Agudo,Biron,123,65,0.5285
```

- `win_rate` is `wins / matches`, rounded to 4 decimals.
- The table is **symmetric**: both `(A,B)` and `(B,A)` are present, with
  `wins` for `(B,A)` = `matches − wins` of `(A,B)`.
- **Small samples are noisy.** Rows with `matches < 5` are statistical
  noise. Filter the file if you plan to show it to non-technical users.

### `hok_hero_synergies.csv`

One row per `(hero_1, hero_2)` pair that appeared on the same team:

```
hero_1,hero_2,matches,wins,win_rate
Agudo,Biron,89,52,0.5843
```

- Same schema and rounding as the counters file.
- Mirrored — `(A,B)` and `(B,A)` carry identical stats so the frontend can
  look up the pair in either order without special-casing.

### Why these two files only

Both derived tables are computed outside this project from the raw match
history. `index.html` never reads raw match data — it only consumes the two
aggregated CSVs. That means the frontend is small and fast, and the data can
be regenerated by any process (including by hand, if needed) as long as the
column layout above is preserved.

---

## How hero names are matched

This is the part that trips people up most, so it is spelled out here.

The frontend compares heroes across three different string spaces:

1. **`data.js` `name`** — what the search box filters on and what appears on
   cards and token labels.
2. **CSV hero column** — the exact spelling used in the aggregate tables.
3. **Internal normalised keys** — what the code actually compares.

The normaliser (`normKey`) collapses everything to lowercase alphanumerics
plus spaces:

```js
normKey("Ao'yin")            // → "ao yin"
normKey("Flowborn (Tank)")   // → "flowborn tank"
normKey("Chang'e")           // → "chang e"
```

`nameKeys(name)` then produces up to **three** candidate keys for a hero:

```js
nameKeys("Flowborn (Tank)")  // → ["flowborn tank"]
nameKeys("Loong (Ao'yin)")   // → ["loong ao yin", "loong", "ao yin"]
nameKeys("Garuda")           // → ["garuda"]
```

The rules:

1. Full name, normalised.
2. Name with any `(…)` block stripped out.
3. The **contents** of any `(…)` block, normalised.

`heroLookupKey()` walks those keys in order and returns the first one that
appears in the loaded CSVs. So:

| `data.js` `name`  | CSV spelling          | Resolves via | Works? |
|-------------------|-----------------------|--------------|--------|
| `Flowborn Tank`   | `Flowborn (Tank)`     | rule 1       | ✅     |
| `Loong (Ao'yin)`  | `Ao'yin`              | rule 3       | ✅     |
| `Garuda`          | `Garuda Khageswara`   | *(none)*     | ❌     |

**Rule of thumb for adding a hero:** open `hok_hero_counters.csv`, search
for the hero, and copy the exact string you find there into `name` in
`data.js`. If the CSV says `"Garuda Khageswara"`, use that — a short UI name
would require an alias feature that is not currently implemented.

---

## Customising the board

### Add a hero

1. Drop the portrait into `HEROES/` (`.webp`, `.png` or `.jpg`).
2. Add one line to `data.js`:

   ```js
   { id: "newhero", name: "New Hero", role: "", img: "HEROES/NewHero_Icon.webp" },
   ```

3. Verify the name matches the CSV exactly (see the previous section).
4. Commit and push — GitHub Pages will rebuild in a minute or two.

You can also use the in-app **⚙️ Data file helper** to generate `data.js`
from a folder of images, but note the caveat: the helper keeps filename
suffixes like `_Icon`, so `Agudo_Icon.webp` becomes `"Agudo Icon"` which
will not match `"Agudo"` in the CSVs. Either strip the suffix from the
filenames before feeding them in, or fix the generated names by hand.
Hand-editing an existing correct `data.js` is usually safer.

### Add a map

1. Drop the image into `MAPS/` (any size; the board scales to fit).
2. Add one line to `data.js`:

   ```js
   { id: "newmap", name: "New Map", img: "MAPS/newmap.png" },
   ```

The picker in the sidebar will show it immediately; each map has its own
independent set of states and annotations.

### Themes / colours

The palette and all UI colours are CSS custom properties at the top of
`index.html` (`--bg`, `--accent`, `--blue`, `--red`, etc.). The arrow and
sticky-note palettes are the `ARROW_COLORS` and `STICKY_COLORS` arrays a
few lines into the inline `<script>`.

---

## Preparing the project for another game

The board, drawing tools, timeline, save/load, and the whole UX are
**game-agnostic**. Only two things carry game-specific content:

1. **`data.js`** — hero names, portraits and map list.
2. **The two CSVs** — the aggregate win-rate tables, in the exact column
   format described in [Data files explained](#data-files-explained).

To repurpose the project, replace those three files and drop matching images
into `HEROES/` and `MAPS/`. No changes to `index.html` are required as long
as the CSV headers stay the same. If you need additional columns (skill
data, bans, pick order, etc.), `index.html` is a single file and easy to
extend — the CSV parser is a ~20-line function near the top of the inline
script.

If you need help preparing this project for another game, contact Kovas —
see [Credits, license & contact](#credits-license--contact).

---

## Troubleshooting

### "No matchup data loaded" in the sidebar

- **Running locally via `file://`?** The browser blocks the auto-load of
  the CSVs. Use `python -m http.server 8000` or the **Load CSV files**
  button. On the hosted GitHub Pages version this does not happen.
- **CSVs missing from the hosting folder?** Move them next to `index.html`.
- **CSVs present but malformed?** The first line must be exactly the header
  described in [Data files explained](#data-files-explained).

### A hero shows no data even though it is in the CSV

Almost always a **name mismatch**. Run this in the browser console after
loading the CSVs:

```js
// list every hero string that made it into the loaded tables
[...matchup.names].filter(n => n.includes("garuda"));
```

Compare with the `name` field in `data.js`. If they differ, edit `data.js`.

### Compare shows "No data" for two heroes that should have a rate

`computeCompare` prefers synergy for same-team tokens and counter for
opposite-team tokens, then falls back to the other table. If both tables
genuinely lack the pair, the badge will read *No data*. To verify what the
frontend actually has:

```js
// after loading CSVs — use normalised names
matchup.counters.get("ao yin|garuda khageswara");
matchup.synergies.get("ao yin|garuda khageswara");
```

A single match is still a valid sample — the panel will show `1 matches`.

### Win prediction is stuck at 50 / 50

The prediction needs **both** teams to have at least one hero and at least
one counter or synergy pair to exist between or within the teams. If every
pair is missing from the CSVs, both sides default to the same score and
normalise to 50 / 50.

### Helper-generated `data.js` broke all matchups

The helper keeps filename suffixes (`Agudo_Icon.webp` → `"Agudo Icon"`),
which will not match a CSV name. Revert to your previous `data.js`, or
hand-edit the generated one.

### My boards disappeared on another device

Auto-save uses `localStorage`, which is **per browser and per origin**. A
board created on your laptop is not visible on your phone. Use **Save
scheme** to export a `.json` and **Load scheme** to import it on the other
device.

---

## Known limitations

- **Data is a snapshot** taken up to 2026-10-25 (see the disclaimer at the
  top). It does not update automatically with new patches or heroes.
- **Small samples are noisy.** Rows with very few matches can swing wildly.
  Filter the CSVs (e.g. keep `matches >= 5`) before shipping if the UI is
  meant for a non-technical audience.
- **Win prediction is a heuristic**, not a fitted model. It reacts
  plausibly to drafts but is not calibrated to real win probabilities.
- **No alias system.** A `data.js` hero name must match the CSV spelling
  exactly, modulo the `normKey` normalisation described above.
- **No per-side count constraint.** The board allows any number of tokens
  per team — useful for scrim planning, but the win-prediction math will
  scale with the number of tokens placed rather than enforce 5v5.
- **Boards are per-browser.** `localStorage` is not synced between devices;
  use the save/load `.json` snapshot to move a board around.

---

## Credits, license & contact

- **Heroes and maps** belong to Honor of Kings / TiMi Studio Group /
  Tencent. This project does not claim any rights to them.
- **Match data** is derived from publicly available pro-team match
  history. Attribution to the original data source is expected if you
  redistribute the CSVs.
- **`index.html`** — the license for this file is granted by the
  maintainers; see the `LICENSE` file in this folder for the full terms.
- **Community:** <https://discord.gg/DfW5kPKsv5>

### Contact

If any help is needed in preparing this project for other games, you can
contact **Kovas** on:

- Discord: `andrius.a`
- TikTok: `VDX.Kovas`
