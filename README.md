# Query Menu

A Tampermonkey userscript and a companion editor for running configurable searches
and tools from any web page.

Two pieces work together:

| File | What it is |
|------|------------|
| `Query_Menu-3_8_user.js` | The userscript you install in Tampermonkey. It adds the right-click menu and does all the work. |
| `query-library-editor-3.6.html` | A local web page for editing your queries without hand-writing the config, and for exporting a **Query Launcher** page. (Compatible with Query Menu 3.8.) |

You can use the userscript on its own (edit its config by hand) or use the editor to
manage everything and paste the result back in.

---

## 1. Install

1. Install **Tampermonkey** in your browser.
2. Open `Query_Menu-3_8_user.js` — Tampermonkey offers to install it. Confirm, and
   approve the permissions it lists.
3. For downloads and for the Query Launcher page, let Tampermonkey work on local files:
   `chrome://extensions` → **Tampermonkey** → **Details** → turn on
   **Allow access to file URLs**.
4. Optional but recommended: Tampermonkey **Settings → Advanced → Inject Mode: Instant**.
   This starts the script as early as possible, which helps on sites that try to block
   custom menus.

---

## 2. The right-click menu

Hold **Ctrl** and right-click. What you get depends on what is under the cursor:

| You Ctrl+right-click on… | The menu shows |
|--------------------------|----------------|
| **selected text** | your *text* queries (the value is the selection) |
| **a link** | your *URL* queries for that link |
| **selected text that is a URL** | *URL* queries, then your *text* queries below |
| **empty page area** | *URL* queries for the current page, plus the bookmarklet library |

The menu also opens inside `<iframe>`s — text or a link selected in an embedded frame
works too (see section 7).

Other ways to open it, for sites that interfere with right-click:

- **Ctrl+Shift+Space** — opens the menu for the selection, else the link under the mouse,
  else the page. Press again to close. (Configurable; see `HOTKEY` in the settings.)
- **The Tampermonkey icon → "Open Query Menu"** — nothing on a page can block this.
- In apps that draw their own text (Google Docs, Sheets…), copy the text with **Ctrl+C**,
  open the menu, and pick **"Use copied text"**.

Inside the menu:

- Click a query to run it (opens a background tab by default).
- For URL queries, a row of buttons at the top picks which **part** of the URL to use:
  **Full URL**, **Domain**, or **Subdomain**. Left/Right arrow keys switch parts.
- Arrow keys move the highlight; **Enter** runs the highlighted row; **Esc** closes.
- A **folder** row runs every query inside it, one every couple of seconds. A **Stop**
  box appears in the corner while they open. The **›** button opens a folder to run its
  queries one at a time.

---

## 3. Configuring queries

The config lives between these banners near the top of the userscript:

```
// ============  CONFIGURE YOUR QUERIES HERE  =================
...
// ==============  END OF CONFIGURATION  ======================
```

You can edit it by hand, or use the editor page (section 5) and paste the result back.

### A query

```js
{ name: 'Google', url: 'https://www.google.com/search?q=%s', on: 'text' }
```

| Field | Meaning |
|-------|---------|
| `name` | the label shown in the menu |
| `url` | the address to open; `%s` is replaced by the value |
| `on` | `'text'`, `'url'`, or `'both'` — which kind of target it runs on (default `'both'`) |
| `raw` | `true` to insert the value **without** URL-encoding (for `%s` inside a path) |
| `transform` | a named transform, or a list of them, applied to the value first |
| `fill` | makes it a **fill query** — see section 4 |

### A folder

```js
{ folder: 'Search engines', on: 'text', queries: [ {query}, {query}, ... ] }
```

A folder's `on` is the default for the queries inside it; a query can override it.
Folders don't nest.

### Transforms

Named functions that change the value before it's inserted. Add your own to the
`TRANSFORMS` block. Built-in ones include:

- **Coordinates:** `lat,lon -> lon,lat`, `coords -> comma`, `coords -> space`
- **Text:** `trim`, `lower`, `upper`, `strip spaces`, `strip newlines`, `quote`, `first line`
- **URLs:** `strip scheme`, `strip www`, `host only`

`host only` turns a full URL into just its hostname — e.g.
`https://sub.example.com/path` → `sub.example.com`. If the value isn't a URL it is left
unchanged.

### Settings (in the config block)

| Setting | Default | What it does |
|---------|---------|--------------|
| `MODIFIER` | `'ctrlKey'` | key held with right-click. `null`, or `'altKey'`/`'shiftKey'`/`'metaKey'` |
| `HOTKEY` | `'Ctrl+Shift+Space'` | shortcut to open the menu. A modifier set plus a letter, digit, or `Space`/`Enter`/`Tab`/`Esc`. `null` to disable. Avoid Alt-based combos on non‑US keyboards (see section 8) |
| `CLIPBOARD_ROW` | `true` | add "Use copied text" to link/page menus |
| `RUN_IN_FRAMES` | `true` | also run inside `<iframe>`s (`false` = top page only) |
| `FRAME_MIN_PX` | `120` | skip frames smaller than this on either side (ad / tracking pixels) |
| `DEFAULT_PART_INDEX` | `0` | which URL part is preselected (0 = Full URL) |
| `COLLAPSE_WHITESPACE` | `true` | collapse newlines/runs of spaces in text (`%s`) values. **Fill values are never collapsed.** |
| `MENU_KEEP_OPEN` | `true` | keep the menu open after running a query |
| `MENU_OPEN_IN_BACKGROUND` | `true` | open query tabs in the background |
| `FOLDER_INTERVAL_MS` | `2000` | gap between tabs when a folder runs all |
| `FILL_TTL_MS` | `20000` | how long a fill value stays valid after you run it |
| `FILL_WAIT_MS` | `10000` | how long to wait for a search box/button to appear |
| `FILL_SUBMIT_MS` | `150` | pause after filling before submitting |
| `DEBUG` | `false` | `true` logs every step to the console as `[QueryMenu]` |

---

## 4. Fill queries (type into a site's box instead of the URL)

Some sites only search through their own box and never put the term in the address.
A **fill query** opens the plain page, types the value into the box, and starts the
search. The value is passed through Tampermonkey's private storage, so it **never
appears in the URL, the request, or any header**.

```js
{ name: 'DNSDumpster', url: 'https://dnsdumpster.com/', transform: 'host only',
  fill: { input: '#target',
          button: 'form[data-form-id="mainform"] button[type="submit"]',
          submit: 'click' } }
```

| `fill` field | Meaning |
|--------------|---------|
| `input` | CSS selector for the box (required) |
| `button` | CSS selector for the search/send button (optional) |
| `submit` | `'click'` (click the button), `'form'` (submit its form), or `'enter'` (press Enter). Default: `click` if a button is given, else `form`. |

Here `url` is the plain page to open and has **no** `%s`. `transform` still applies.

The box can be a normal `<input>`/`<textarea>` **or a rich contenteditable editor**
(ChatGPT, Claude and similar). Fill values keep their line breaks (they are never
whitespace-collapsed), so a multi-line value such as a prompt is typed in intact. For
`submit: 'click'`, the button is clicked once it becomes enabled.

**Finding the selectors:** on the site, right-click the box (or button) → **Inspect**,
and read its `id`, `name` or `aria-label`. `#q` matches `id="q"`; `input[name="q"]`
matches `name="q"`. AI sites change their HTML often, so these may need updating from
time to time; if the button won't trigger, drop it and use `submit: 'enter'`.

Notes:
- `submit: 'enter'` only works on sites whose own code reacts to Enter. Prefer `click`
  or `form` when you can.
- The value is still typed by a script, so a site that inspects its input events can tell
  it wasn't hand-typed. What fill queries remove is the term from the URL and headers.

---

## 5. The editor page

Open `query-library-editor-3.6.html` in your browser (double-click it). It never sends
anything anywhere — it's a local tool.

**Importing:** paste your config block (everything between the two banners) into the box
at the top and click **Import**. Queries, transforms and the bookmarklet library become
editable. Import again to merge in another script's config. Everything else in the block
(URL parts, settings, comments) is preserved and written back on export.

**Editing** — four tabs:
- **Queries** — add/reorder/fold queries by drag or buttons. Each row has: name, URL, a
  **text / url / both** selector, a transform picker, a **raw** box, and a **fill** button
  that opens a panel for the box/button/submit selectors. A filter at the top shows All /
  Text / URL.
- **Transforms** — edit the named functions, with a live tester.
- **Bookmarklets** — edit the bookmarklet library and their per-site match rules.
- **Other config** — the preserved text (banners, URL parts, settings).

**Exporting:** the right-hand panel shows the config to paste back into the userscript.
Choose the scope (full block / QUERIES only / BOOKMARKLETS only), then **Copy** or
**Download**. Paste it back between the two banners in `Query_Menu-3_8_user.js`.

**Load sample** fills the import box with an example config (including fill-query
examples) so you can see the format.

---

## 6. The Query Launcher page

Click **Homepage ↗** in the editor to export a **Query Launcher** — a standalone page
for running many searches at once. Save it and open it as a local file.

Each query group has a box for **search terms** (one per line) and a checkbox per query.
Tick the queries you want, type your terms, and click **Open tabs**: every term × every
ticked query opens in its own tab, spaced out by the **stagger** value. Two extra
sections at the end:

- **Open URLs** — paste a list of addresses and open them all in tabs.
- **Downloads** — paste a list of file addresses and save them all (optionally into a
  subfolder of Downloads), with a per-file progress list.

**The badge at the top:**
- **via Query Menu** (green) — the userscript is active on the page. Tabs, downloads and
  **fill queries** all run through it with no pop-up limit and a Stop box.
- **browser pop-ups** — the userscript isn't reaching the page. Plain queries still open
  (the browser allows one tab per click; an **Open next** button handles the rest), but
  **fill queries and downloads need Query Menu** and are skipped with a note.

For the badge to be green, the launcher must be opened as a local file (`file://`) with
Tampermonkey allowed access to file URLs (install step 3), using Query Menu 3.6+.

Fill queries appear in the launcher with a green **fill → #selector** tag. Each term you
type becomes a value typed into that site's box.

---

## 7. Works on difficult sites, and in iframes

The menu is built to survive pages that try to block or hide custom menus:

- Listeners are registered at `document-start`, so the script sees the right-click before
  the site does; if a site still swallows it, the menu opens from the mouse buttons.
- The menu lives in a closed shadow root in the browser's top layer, so it shows above
  modals and fullscreen video and the page can't read or restyle it.
- Its styles use a method strict security policies don't block; if the page removes the
  menu, it's put back.
- Scrolling inside a panel (carousels, chat, feeds) doesn't close it; only scrolling the
  page does.

**Iframes:** the script also runs inside embedded frames, so a selection or link in an
`<iframe>` opens the menu too (it appears within that frame). The bookmarklet library,
the launcher hand-off and the fill step belong to the top page and are skipped in frames.
Frames smaller than `FRAME_MIN_PX` (ad/tracking pixels) are ignored. Set
`RUN_IN_FRAMES = false` to go back to top‑page‑only.

If the menu doesn't appear on some site, set `DEBUG = true` in the config and watch the
console (`[QueryMenu] …`) to see which step is affected.

**Can't be worked around** (browser limits, not bugs): browser and extension-store pages
(`chrome://`, add-on sites), the built-in PDF viewer, and sandboxed or cross-origin frames
that block script injection.

---

## 8. Troubleshooting

| Symptom | Fix |
|---------|-----|
| Menu doesn't open on a site | Try **Ctrl+Shift+Space** or the Tampermonkey icon; set `DEBUG = true` to see why. |
| The hotkey does nothing | On some keyboards the right **Alt (AltGr)** reports as Ctrl+Alt and types characters, and some systems don't hold the left Alt onto the next key — so avoid Alt-based hotkeys. `Ctrl+Shift+Space` is the robust default. To test a combo, paste in the console: `addEventListener('keydown',e=>console.log(e.code,e.ctrlKey,e.shiftKey,e.altKey),true)` and press it. |
| Fill query opens the page but doesn't fill/send | Re-check the `input`/`button` selectors (Inspect the box/button — AI sites change them). If the button won't trigger, use `submit: 'enter'`. |
| Launcher badge says "browser pop-ups" | Open it as a local file and allow Tampermonkey access to file URLs; use Query Menu 3.6+. |
| Downloads do nothing | Allow the download when Tampermonkey asks; blocked file types are listed under Settings → Downloads → Whitelisted File Extensions. Subfolders need Download Mode "Browser API". |
| Launcher says "Query Menu too old" | Install Query Menu 3.6 or later (3.8 recommended). |

---

## Bookmarklet library

The bookmarklet library bundled in the userscript (shown when you Ctrl+right-click an
empty area of a page) was built from **[tools.myosint.training](https://tools.myosint.training/)**.
It is a snapshot and may not be the most up-to-date version — check the site for the
current tools. Edit or replace it in the editor's **Bookmarklets** tab, or directly in the
`BOOKMARKLETS` array in the userscript.

---

## Version

Query Menu **3.8** · editor / launcher **3.6** (compatible). Install the userscript and
export the launcher from the editor.

*Built with [Claude](https://claude.ai). Bundled bookmarklet library from
[tools.myosint.training](https://tools.myosint.training/). Inspiration from [Selection Search](https://chromewebstore.google.com/detail/selection-search/gipnlpdeieaidmmeaichnddnmjmcakoe) and [Inteltechniques tools list](https://inteltechniques.com/tools/)*
