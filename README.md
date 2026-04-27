# Loom

A single-file, browser-based tool for viewing, editing, and converting JSON
into Markdown and Excel.

**Run online:** <https://jmkorhonen.github.io/json_loom/loom>

**Run locally:** Head to the [repository](https://github.com/jmkorhonen/json_loom), if you aren't there yet, and download the latest `loom.html` or `loom-vX.Y.ZZ.html`, double-click to open
in any modern browser (Firefox, Chrome, Safari, Edge). 

**No install, no server, no build step. Your data never leaves your machine.**

> Loom is one researcher's side project to produce a tool for personal use. The data formats and pipelines
> have been stable since 0.5, but you should still keep backups of
> anything important. If it eats your data, runs away with your dog, or
> causes the very fabric of the universe to unravel, *that's on you.*

---

## Contents

- [What Loom does](#what-loom-does)
- [The interface at a glance](#the-interface-at-a-glance)
- [Editing JSON](#editing-json)
- [Rules — how each node renders](#rules--how-each-node-renders)
  - [The five roles](#the-five-roles)
  - [Tables from arrays](#tables-from-arrays)
  - [Tables from objects](#tables-from-objects)
  - [Headings](#headings)
  - [Markdown styling](#markdown-styling)
  - [Decimal marks](#decimal-marks)
  - [Label humanisation](#label-humanisation)
- [Annotations — free-form text around any node](#annotations--free-form-text-around-any-node)
  - [Tokens](#tokens)
  - [YAML front matter](#yaml-front-matter)
- [Document front matter](#document-front-matter)
- [Splitting into multiple files](#splitting-into-multiple-files)
  - [Filename templates](#filename-templates)
  - [Slug options](#slug-options)
  - [Per-file front matter](#per-file-front-matter)
- [Presets](#presets)
- [Exporting](#exporting)
- [Tree markers](#tree-markers)
- [Keyboard shortcuts](#keyboard-shortcuts)
- [Worked examples](#worked-examples)
  - [Survey results as a compact report](#survey-results-as-a-compact-report)
  - [Splitting interviews into per-person Markdown files](#splitting-interviews-into-per-person-markdown-files)
  - [Finnish-locale numbers for a policy brief](#finnish-locale-numbers-for-a-policy-brief)
  - [Survey results as a per-persona Excel workbook](#survey-results-as-a-per-persona-excel-workbook)
- [Limitations and quirks](#limitations-and-quirks)
- [Release notes](#release-notes)

---

## What Loom does

Loom turns a JSON file into a Markdown file. That sounds simple, but the
interesting part is how much control you have over the conversion: each node
in the JSON tree can be configured independently. You can decide what becomes
a heading, what becomes a table, what gets skipped, what becomes a bullet
list, what gets styled in bold, what picks up a YAML front matter, and so on.

Loom is useful when you have JSON that is *almost* a document — survey
results, structured interview notes, experiment logs, API dumps, configuration
bundles — and you want a clean human-readable version for a report,
publication, or handover.

The three core concepts are:

1. **Rules** — how individual nodes render (heading, table, list, skipped,
   styled).
2. **Annotations** — free-form Markdown you inject before or after any node.
3. **Splits** — produce multiple output files by chopping an array.

Everything is saved to the browser's `localStorage` as you edit, so closing
the tab won't lose your work.

---

## The interface at a glance

A tab bar at the top shows every open document. Click a tab to switch,
click the **+** at the end to open a new empty document, click the **×**
on a tab to close it. Each tab has fully independent state — opening
two surveys side-by-side, or experimenting with rules in one tab while
keeping a clean copy in another, just works.

Below the tab bar are four resizable panes, each collapsible via the
chevron in its header.

- **Left — Structure.** A tree view of the JSON. Click anywhere on a row to
  select it. Small chips next to each node show its active role, split
  status, and any attached annotation. The type pill is clickable for
  quick type changes.
- **Second — Editor.** Always shows the currently-selected node. A
  breadcrumb at the top shows the path and lets you navigate up. Each
  child appears as a row with a key input, type pill, value input (or
  summary + Open → for containers), and delete button. Clicking a
  container row navigates into it; clicking Open → does the same.
  All value and structure edits happen here.
- **Middle — Output.** Four tabs: **Preview** (rendered Markdown), **Raw
  MD** (the actual Markdown text), **XLSX** (rendered sheet preview),
  **Split plan** (list of files that would be produced).
- **Right — Inspector.** Three tabs: **Rule** (configure the selected
  node), **Split** (only meaningful on arrays), **Document** (document-
  wide settings like YAML front matter).

Above the panes is a toolbar with New / Open / Save / Export / Undo / Redo
plus the preset selector and a filename chip that shows an accent dot
when there are unsaved edits. Below is a status bar showing node count,
rule count, annotation count, and the last autosave time.

---

## Editing JSON

All editing happens in the **Editor pane**. Select a node in Structure,
and the Editor shows it in full:

- **Rename a key** — click the key input, type, Tab or click away to commit.
  Tip: clicking a leaf node in Structure (like `title` or `n_questions`)
  selects its parent in the Editor and auto-focuses that leaf's key
  input — one click and you can start typing the new name.
- **Edit a scalar value** — the value input accepts new text or numbers;
  booleans show true/false toggle buttons; null shows a "change type to
  assign" hint.
- **Edit a long string** — when the selected node *is* a string, Editor
  gives you a full multi-line textarea instead of a single-line input.
- **Reorder children** — grab the `⋮⋮` handle on the left of any row
  and drag it up or down to reorder within the same parent. Works for
  both arrays and objects; key order in objects is preserved through save.
- **Change a node's type** — click the type pill on its row. Container
  values are preserved where sensible: `{}` → `[]` turns object values
  into an array, `"5"` → `#` parses to `5`.
- **Add a key or item** — use the sticky `+ key` / `+ item` button at the
  bottom of the Editor, or use the corresponding button in any
  expanded container in Structure. The newly-added row's input gets
  auto-focused so you can type the name immediately.
- **Navigate into a container** — click anywhere on a container row, or
  the Open → button.
- **Delete** — the × button on each row, or the × Delete button in the
  breadcrumb for the currently-selected node. Non-empty containers
  prompt for confirmation. You can always undo.

The Structure pane is navigation-only — clicking a row just selects it.
If you prefer, you can collapse Structure entirely and navigate purely
via the Editor's breadcrumb.

A dropped `.json` file onto the window replaces the current document
(after confirming if you have unsaved edits). The loaded filename is
shown next to the logo; an accent dot appears when there are unsaved
changes.

### A note on types

JSON allows mixed types in both arrays and objects — you can have
`[1, "two", {three: 3}, null, true]` and that's perfectly valid. Loom
won't stop you from adding values of different types under the same
parent. If you change a node's type, Loom will warn you when the change
is destructive — for example, turning a non-empty object into an array
discards its keys (only the values become items), and turning any
container into a scalar deletes its contents. You can always undo.

---

## Rules — how each node renders

Select any node in the tree and the Rule tab shows its settings. A rule set
on a node applies to that node and, because it is keyed by the *pattern* of
the path (array indices become `*`), to all siblings of the same shape. So
setting a table rule on `runs[0][0]` applies to every persona object in every
run.

### The five roles

- **Auto** (default) — let Loom decide.
- **Heading + contents** — render as a section header with this node's
  contents beneath. For objects: heading then fields; for arrays: heading
  then each child rendered as a sub-section.
- **Table** — render as a Markdown table. Applicable to arrays of objects
  and to objects.
- **Bullet list** — render array items as a flat bullet list.
- **Skip** — omit this node entirely. Handy for noisy metadata.

### Tables from arrays

Select an array-of-objects (or one of its items) and pick **Table**. The
Inspector shows a checkbox list of columns inferred from the data. You can:

- Toggle columns on/off.
- Set a custom **join separator** for each column — useful when a cell
  value is itself an array (e.g. `tags: ["a","b"]` renders as `a, b`).

Example: given

```json
{
  "projects": [
    { "name": "Ecosystems",       "lead": "Anna",  "tags": ["bio","climate"] },
    { "name": "Circular Economy", "lead": "Mikko", "tags": ["materials"] }
  ]
}
```

and a table rule on `projects[*]` with all three columns selected, the
output is:

```markdown
| Name             | Lead  | Tags         |
|------------------|-------|--------------|
| Ecosystems       | Anna  | bio, climate |
| Circular Economy | Mikko | materials    |
```

### Tables from objects

Objects can also be tables, in two modes:

- **Key / Value pairs** — one row per field. Turns `{a:1,b:2}` into a
  two-column table. Great for "metadata" sections.
- **Child objects as rows** — available when every value is itself an
  object. First column is the outer key, remaining columns are picked from
  the children. Turns `{alice: {...}, bob: {...}}` into a single tidy table.

### Headings

When a node renders as Heading + contents, you pick:

- **Heading from** — where the heading text comes from. Default is the key
  name (humanised). If the node is an object with a `title` or `name` field,
  Loom uses that automatically. You can pick any other string/number field
  explicitly (`.label`, `.persona`, etc.).
- **Level** — automatic (based on nesting depth) or fixed `H1`–`H6`.

When Loom auto-picks a field for the heading, it won't also emit that field
as a leaf — no duplicated `**Name:** X` after the `## X` header.

### Markdown styling

Each rule has **Key style** and **Value style** dropdowns: Plain, **Bold**,
*Italic*, ***Bold italic***. These apply to leaves rendered as
`Key: value` pairs (the typical case inside objects and bullet lists).
Child rules override parent rules.

So to produce `*Title*: **Whatever**` across the whole document, set the
root rule's key style to italic and value style to bold. To override just
one specific field, click that field and set a different rule on it.

### Decimal marks

If a subtree contains numbers, the Markdown Styling section exposes a
**Decimal mark** dropdown: `.` (default, `0.25`) or `,` (Finnish/European,
`0,25`). Applied everywhere: leaves, table cells, bullets, joined arrays.

### Label humanisation

Each rule has a **Label formatting** choice:

- **Humanize** (default) — `user_name` → "User name", `climateEU` →
  "Climate EU", `exact_match_rate` → "Exact match rate".
- **As-is** — keep original keys verbatim. Useful when the key names are
  already meaningful identifiers.

---

## Annotations — free-form text around any node

Scroll down the Rule tab to find two textareas: **Before this node** and
**After this node**. Whatever you type is inserted verbatim at that exact
location in the output. No blank lines are added automatically — press Enter
to add them. Newlines in the textarea become literal newlines.

Like rules, annotations are attached to a *pattern path*, so putting an
annotation on `runs[0]` puts it on every run.

Annotations are kept entirely separate from the JSON — saving the JSON file
won't embed them. They live in the preset (and in autosave).

### Tokens

Annotations (and per-split front matter, and document front matter) support a
small set of value tokens that resolve at render time:

- `{.field}` — a field from the *current* node.
- `{.nested.field}` — a nested lookup from the current node.
- `{parent.field}` — a field from the parent node.
- `{$.field}` or `{doc.field}` — a field from the document root.

Missing values resolve to empty strings. Unrecognised tokens are passed
through unchanged.

### YAML front matter

For a single document-wide YAML header, use the **Document** tab (see the
next section). For section-scoped front matter (e.g. `---` dividers between
parts of a report), use Before/After annotations.

---

## Document front matter

The Inspector's **Document** tab holds document-wide settings independent of
tree selection. The main field is a **Document front matter** textarea —
whatever you put here is inserted at the very top of:

- the single output file when no splits are active, **or**
- `index.md` when splits are active.

Tokens resolve against the document root, so something like:

```yaml
---
title: {$.title}
date: {$.timestamp}
tags: survey, environment
---
```

…produces a valid Pandoc/Jekyll/Hugo-compatible header. (Make sure the
closing `---` ends with a newline; press Enter once more if you want a
blank line between the YAML block and the first heading.)

Per-split child files use the Split tab's **Front matter (every file)**
field instead — see below.

---

## Splitting into multiple files

Select any array and open the **Split** tab. Tick *Split this array* and
each item becomes its own output file (or sheet, for XLSX). The remaining
document — with the split array replaced by a link list — becomes
`index.md`.

### Filename templates

The **Filename template** field accepts these tokens:

- `{i}` — 1-based index.
- `{i:02}`, `{i:03}`, … — zero-padded index.
- `{key}` — item index or key.
- `{.field}` — field from the item (e.g. `{.name}`, `{.id}`).
- `{.nested.field}` — nested lookup from the item.
- `{parent.field}` — field from the parent object.
- Literal text works too.

Examples:

- `{i:02}-{.name}.md` → `01-ecosystems.md`, `02-circular-economy.md`, …
- `report-{.year}/section-{i}.md` → produces subdirectory-style names
  (the `/` is preserved).

### Slug options

When rendering a filename, each path segment is slugified. Options:

- **Lowercase** — `Kestävät Kaupungit` → `kestavat kaupungit`.
- **Replace spaces with dashes** — `kestavat kaupungit` → `kestavat-kaupungit`.
- **Strip accents** — `ä` → `a`, `ö` → `o`, `ñ` → `n`, etc. (Essential for
  Finnish and other diacritic-heavy data if you want ASCII filenames.)
- **Max length** — truncate each segment; default 60.

Together with the template, this produces safe, predictable filenames.

### Per-file front matter

The Split tab also has a **Front matter (every file)** textarea. Whatever
goes here is prepended to every split child file, with the same tokens
available (`{.field}` resolves against the child item, not the root).

Common pattern:

```yaml
---
title: {.name}
id: {.id}
---
```

This gives each split file its own header derived from its content.

---

## Presets

A preset bundles your rules, splits, annotations, and document front matter.
Use them to keep your "how this kind of JSON should render" configurations
reusable.

- **Save as…** — save the current configuration under a name.
- **Manage…** — list all presets with Export (downloads a
  `.loom-preset.json`) and Delete. Same dialog has an Import button.

Exported preset files can be shared; importing either merges (if the file
contains a single preset) or imports multiple at once (if the file contains
a `presets` object mapping names to configurations).

---

## Exporting

- **Save JSON** — downloads the current document as JSON. Filename is
  inferred from the source file (or from the document's `title`/`name`
  field).
- **Export MD** — downloads either a single `.md` file or, if splits are
  active, a `.zip` with all the Markdown files.
- **Export XLSX** — produces an Excel workbook. Sheets come from:
  - Arrays/objects with the Table role → one sheet each.
  - Arrays with Split enabled → one sheet per child, plus an `Index`
    sheet at the front showing root metadata and a sheet-map.
  - If neither, a flat key/path sheet as a fallback.

  Within a split-child sheet, nested structure is preserved legibly: the
  item's primitive fields go into a top Field/Value block, each nested
  object becomes its own Field/Value block under a header row, and each
  array-of-objects becomes a proper column-based table with its own header
  row. No more JSON blobs pasted into single cells.

  The **XLSX** tab in the Output pane shows a live preview of every sheet
  as an HTML table before you download anything.

---

## Tree markers

Every tree row shows small chips indicating what will happen:

- **table**, **list**, **heading**, **skip** — the rule's role, colour-coded.
- **split** — this array is a split point.
- **※** — this node has an annotation attached.

Makes it easy to spot something you've accidentally set to *skip*.

---

## Keyboard shortcuts

Document-wide:

- <kbd>Ctrl/⌘+Z</kbd> — undo.
- <kbd>Ctrl/⌘+Shift+Z</kbd> — redo.
- <kbd>Ctrl/⌘+S</kbd> — save JSON.
- <kbd>Ctrl/⌘+O</kbd> — open a JSON file.
- <kbd>Enter</kbd> in an editable key/value — commit the edit.
- <kbd>Esc</kbd> in an editable key/value — cancel.

In the Structure pane (after clicking a node to give the tree focus):

- <kbd>↑</kbd>/<kbd>↓</kbd> — move between visible rows.
- <kbd>←</kbd> — collapse the current node, or move to parent if already collapsed.
- <kbd>→</kbd> — expand the current node, or move to first child if already expanded.
- <kbd>Space</kbd> — toggle expand/collapse.
- <kbd>Home</kbd>/<kbd>End</kbd> — jump to first/last visible row.
- <kbd>Enter</kbd> — move focus to the matching input in the Editor pane.
- <kbd>Esc</kbd> in the search box — clear the filter.

---

## Worked examples

### Survey results as a compact report

You have a JSON like `survey_results.json` with a nested structure:

```
$
├── timestamp, mode, n_runs, n_personas, n_questions  (metadata)
└── runs: [
      [
        { persona, responses: {...}, per_question: [...], exact_match_rate, ... },
        ...
      ],
      ...
    ]
```

You want a single Markdown file with a YAML header, the top-level metadata
as a key/value list, and each run as a compact table of personas with three
columns.

Steps:

1. Load the file.
2. Open the **Document** tab. Paste into Document front matter:
   ```yaml
   ---
   title: {$.description_type} survey — {$.mode}
   date: {$.timestamp}
   n_personas: {$.n_personas}
   ---
   ```
3. Click into the first persona object (`runs[0][0]`). In the Rule tab,
   set **Render as: Table**. Pick the columns `persona`,
   `exact_match_rate`, `adjacent_match_rate`. The rule pattern
   `runs.*.*` means it applies to every persona in every run.
4. Check Preview. Export MD.

### Splitting interviews into per-person Markdown files

Given `interviews.json` with an `interviews` array, each item an object with
`name`, `role`, `date`, and a long `transcript`:

1. Select the `interviews` array. Open the **Split** tab. Tick *Split this
   array*.
2. Filename template: `{i:02}-{.name}.md`. Check **Strip accents** so
   Scandinavian names become ASCII.
3. In *Front matter (every file)*:
   ```yaml
   ---
   interviewee: {.name}
   role: {.role}
   date: {.date}
   ---
   ```
4. Optionally, in the **Document** tab, add a short header for `index.md`.
5. Export MD — you get a zip with `index.md` and one file per interviewee,
   each carrying its own YAML header.

### Finnish-locale numbers for a policy brief

Your data has numbers like `0.2222222222222222` and you want Finnish
convention rounded to two decimals (`0,22`).

1. Select the array containing the numbers. In the Rule tab's Markdown
   Styling section, set **Decimal mark** to `,` and **Decimal places**
   to `2`.
2. The rule propagates to nested content; children inherit unless they
   override. All numbers now show as `0,22` rather than `0.2222222222`.

If you have ISO 8601 timestamps (`2026-03-05T12:40:08.914941`) anywhere
in the document, the Inspector also shows a **Date format** dropdown
with presets like `Mar 5, 2026` or `March 5, 2026`.

### Survey results as a per-persona Excel workbook

Same survey JSON as the first example. This time you want an XLSX where
each persona is its own sheet and the nested `per_question` array renders
as a proper filterable table — not a JSON blob.

1. Load the file.
2. Click into any persona object inside `runs[0]`.
3. Open the **Split** tab. Tick *Split this array*. Template:
   `{i:02}-{.persona}.md` (the `.md` extension is stripped for XLSX
   sheet names).
4. Open the **XLSX** tab in the Output pane. You should see:
   - An `Index` sheet with `Timestamp`, `Mode`, …, and a Sheet/Source map.
   - One sheet per persona, with their scalar fields in a Field/Value
     block, the `Responses` object as a labelled Field/Value block below,
     and `Per question` as a 6-column × 9-row table with real headers
     (`Question id | Predicted | Ground truth | Exact match |
     Adjacent match | Distance`).
5. Click **Export XLSX**. The downloaded workbook is identical to the
   preview. Booleans are native Excel TRUE/FALSE; numbers are numeric, not
   strings.

---

## Limitations and quirks

- **One JSON file at a time.** Loom is for editing/converting one document,
  not for batch pipelines. For batch work, reach for a scripting language.
- **No server, no authentication.** Presets and drafts live in
  `localStorage`. Clearing your browser data wipes them.
- **Rules apply by pattern, not by specific path.** Setting a rule on
  `projects[0]` automatically applies to `projects[1]`, `projects[2]`, etc.
  This is usually what you want, but note there's no per-index override yet.
- **Nested splits work but are lightly tested.** Splitting a top-level
  array usually does what you expect; splitting deep inside a structure
  might surprise.
- **XLSX sheet name edge cases.** Excel forbids `/ \ ? * [ ]` and caps
  names at 31 characters. Loom truncates and deduplicates, but exotic
  names might still complain in Excel.
- **Preview renders the document's Markdown, not the final rendering
  pipeline.** Things like custom Pandoc filters won't show up; the
  preview is a faithful but generic Markdown view.

---

## Release notes

### v1.0

First non-experimental release. The data formats, rules engine, and
output pipelines have been stable for several iterations and the major
feature gaps have been filled. From here, breaking changes will be
called out explicitly in release notes.

**New in 1.0: Multi-document tabs.**

- A tab bar above the panes shows every open document. Each tab has its
  own independent state — doc, rules, splits, annotations, document
  front matter, selection, expanded/collapsed nodes, search query,
  output-pane tab, and undo/redo history.
- Click a tab to switch to it. Click the **+** at the end to open a new
  empty tab. Click the **×** on a tab to close it (you'll be asked to
  confirm if it has unsaved edits).
- Closing the last remaining tab replaces it with a fresh empty one
  rather than leaving the workspace blank.
- Tabs show the doc name in the bar, with an accent dot and accent
  filename when there are unsaved edits — the same dirty-state
  convention as the toolbar filename chip.
- **File open behaviour:** opening a JSON file (via the Open button or
  drag-and-drop) opens it in a new tab if the current tab has any work
  in it; if the current tab is fresh and empty, the file replaces it
  there. No more "Discard current document?" prompt — your existing
  work is preserved automatically.
- **Autosave** persists every open tab. Reopening Loom restores the
  whole tab set, with the previously-active tab focused.
- Old single-document drafts (saved by 0.5.x) restore as a single tab.

### v0.5.13

- **Number rounding.** New rule field "Decimal places" sits next to
  Decimal mark in the Inspector. Options: no rounding (default), 0, 1,
  2, 3, 4, or 6 decimals. Applied via `n.toFixed(d)` before the decimal-
  mark conversion, so `0.2222222222222222` with `decimals: 2` and Finnish
  locale becomes `0,22`. Cascades through inheritance like other rule
  fields. Only shown for subtrees that contain numbers.
- **Date formatting for ISO 8601 strings.** New rule field "Date format"
  appears for subtrees containing strings that look like dates
  (e.g. `2026-03-05T12:40:08.914941`). Options: raw (default), ISO date,
  short (`Mar 5, 2026`), medium (`March 5, 2026`), time only, or date +
  time. Uses `Intl.DateTimeFormat` so output respects the user's browser
  locale. Non-matching strings are left untouched.
- **XLSX column auto-fit.** Each sheet's column widths are now sized to
  match the longest cell content in that column (capped at ~60 chars
  to avoid runaway widths from a single very long value). The Index
  sheet, the field/value blocks, and per-question tables now all open
  in Excel without needing to manually drag column borders.

  *Note: bold headers and frozen first row would require a paid
  SheetJS edition; column widths and native cell types (numbers,
  booleans, dates) are everything the open-source library exposes.*

### v0.5.12

- **Search/filter in the Structure pane.** A search box at the top of
  the tree filters nodes by key name or value content (case-insensitive).
  Matches and all their ancestors stay visible; everything else is
  hidden. Matches are auto-expanded so they're reachable, and matched
  substrings are highlighted with a subtle yellow background. A counter
  shows the number of matches; the × button or <kbd>Esc</kbd> clears the
  filter.
- **Keyboard navigation in the tree.** Once a tree row is selected,
  arrow keys navigate the tree:
  <kbd>↑</kbd>/<kbd>↓</kbd> move between visible rows,
  <kbd>←</kbd> collapses (or moves to parent if already collapsed),
  <kbd>→</kbd> expands (or moves to the first child if already expanded),
  <kbd>Space</kbd> toggles, <kbd>Home</kbd>/<kbd>End</kbd> jump to first/
  last visible row, <kbd>Enter</kbd> moves focus to the corresponding
  input in the Editor pane so you can start typing. Search-and-arrow-down
  combo (type, then <kbd>↓</kbd>) is the fast workflow for jumping to
  a specific node in a big document.
- **Right-click → Copy path / Copy value.** Context menu on any tree
  row offers three options:
  - **Copy path** — friendly dot/bracket notation (`$.runs[0][0].persona`).
  - **Copy pattern path** — the rule key (`runs.*.*.persona`), useful
    when you want to set a rule on every sibling of the same shape.
  - **Copy value as JSON** — the entire subtree pretty-printed.

### v0.5.11

- **Type pill now appears before the key in Editor rows** — matching the
  Structure pane convention (`{}  title`, not `title  {}`).
- **Responsive Editor row layout.** When the Editor pane is wide enough,
  rows stay on a single line: `[handle] [type] [key] [value...] [×]`.
  When the pane gets narrower (below ~460px), the value drops to its own
  line spanning the full pane width, while the handle/type/key/× stay
  compact on top. Fixes the previous overflow that was clipping value
  textareas and delete buttons in narrow panes.
- Word-wrap policy unchanged from 0.5.10 — wrap at word boundaries
  first, mid-word only when necessary.

### v0.5.10

- **Wider default for value textareas.** The value column in editor rows
  has a 200px minimum width now, so short strings like `"New document"`
  no longer wrap to three-character lines in narrow Editor panes. Word-
  wrap also no longer breaks mid-word unless absolutely necessary.
- **Drag-resize the textarea horizontally to widen the Editor pane.**
  String value boxes can now be resized in both directions via the
  bottom-right corner. Dragging right past the textarea's available
  width automatically grows the Editor pane to make room — no more
  having to reach for the pane divider.
- **Friendlier "couldn't open" errors.** Loading a file that isn't valid
  JSON now shows a prominent error banner instead of a quiet message in
  the status bar. Specific cases are detected and labelled — "looks like
  XML/HTML", "looks like YAML", "the file is empty", or the actual parse
  error with line/column from the JSON parser.

### v0.5.09

- **Type menu shows descriptions.** Clicking a type pill opens a menu where
  each type — Object, Array, String, Number, Boolean, Null — has a one-
  line explanation underneath the label, also available as a tooltip.
- **Confirmation before destructive type changes.** Converting a non-empty
  object to an array (which discards keys) or any container to a scalar
  type (which discards everything inside) prompts you first, with a clear
  message of what will be lost. Non-destructive changes (Array → Object,
  Scalar → Container, Empty container → anything) go through without
  prompting. Undo still works either way.
- **String values get an auto-growing textarea.** Long strings now wrap
  and expand to show their full content, up to a comfortable maximum
  height. You can drag the corner to grow further. Numbers and booleans
  keep their compact controls.
- **Narrower index column for arrays.** Array rows show just `[0]`,
  `[1]`, … in a narrow fixed-width column on the left, leaving much more
  horizontal space for the value. Object key columns are unchanged.
- **Removed `× Delete` from the Editor breadcrumb.** Deletion is now done
  exclusively from the row-level `×` button in the parent's view. One
  consistent place for both rename (key field) and delete (× button).

### v0.5.08

- **Click anywhere on a collapsed pane to expand it** — not just the
  chevron. Hovering over a collapsed pane shows a subtle highlight to
  indicate it's clickable.
- **`+ key` and `+ item` buttons restored to the Structure pane.** Adding
  a child to a container no longer requires opening it first; the button
  appears at the bottom of each expanded container in the tree.
- **Editing a key is more direct.** Selecting a leaf scalar in Structure
  now opens its parent in the Editor with that leaf's row highlighted
  and its key input auto-focused — so a single click on, say, `n_questions`
  in the tree lets you immediately rename it.
- **Drag-and-drop reordering.** Each Editor row has a drag handle
  (`⋮⋮` on the left). Drag a row up or down to reorder within the same
  parent — works for both arrays (splice) and objects (key reorder is
  preserved through save). Drop indicators show where the row will land.
- **Adding a new row auto-focuses its input.** After clicking + key /
  + item, the new row's key (or value, for arrays) is focused and
  selected so you can start typing immediately.

### v0.5.07

- **New Editor pane.** A fourth pane between Structure and Output, always
  showing the currently-selected node with full editing capability.
  Breadcrumb at top for navigation; each child of the selected node appears
  as its own row with a key input, type pill, value input (or summary +
  Open → for containers), and delete button. Clicking a container row
  navigates into it.
- **Structure pane is navigation-only now.** Click anywhere on a row to
  select it — no more missing the tiny gap between key and value. All
  editing happens in the Editor pane.
- **All four panes are independently collapsible and resizable.** Output is
  collapsible to the right just like Inspector; Editor is collapsible to
  the left just like Structure. When Output is collapsed the flexible
  1fr role automatically moves to whichever expanded pane is still present,
  so the UI always fills the viewport.
- **Dirty indicator.** When there are unsaved edits, the filename chip in
  the toolbar shows an accent dot, Save JSON gets an accent outline, and
  the browser tab title prepends a `●`. Cleared on open, new, or save.
  A `beforeunload` warning prompts before leaving with unsaved edits.
- **Fixed value-overlaps-key bug.** Tree rows no longer let long values
  push keys off-screen or overlap them in narrow panes — proper flex
  shrinking and overflow rules are now in place.
- **Auto-scroll.** Selecting a node scrolls its Structure row into view.

### v0.5.06

- **XLSX preview tab.** New "XLSX" tab in the Output pane renders every sheet
  as an HTML table, so you can see exactly what the Excel export will look
  like before downloading.
- **Better nested objects in split XLSX.** Fields whose value is an object
  or an array of objects are no longer JSON-stringified into a single cell.
  Each gets its own labelled section in the same sheet: nested objects
  become sub–Field/Value blocks, arrays of objects become proper tables
  with their own columns.
- **Index sheet for split XLSX.** When splits are active, an "Index" sheet
  is placed first, showing top-level document fields plus a Sheet/Source
  map so you can see which sheet corresponds to which path in the original
  JSON.

### v0.5.05

- New **Document** tab in the Inspector, always available.
- Document front matter applies to the single output file or to `index.md`
  when splits are active.
- Placeholder text in annotation and front-matter textareas is now
  lighter and italicised — easier to tell from user input.
- Fixed focus loss while typing in textareas: the Inspector no longer
  re-renders itself out from under you.
- Help modal links to the repo.

### v0.5.04

- Annotations now apply to every node, including scalar leaves.
- Annotation whitespace: verbatim insertion, no forced blank lines.
- Tokens in annotations and split front matter.
- Tree markers for role, split, and annotation.
- Delete confirmation for non-empty containers.
- Toolbar Undo/Redo buttons.
- Per-split front matter.
- Preview shows every split file with labelled banners.
- Decimal-mark selector.
- Version tag in toolbar and help modal.

### Earlier

v0.1 — initial release with three-pane layout, rules engine, tree editor,
MD/XLSX export, splits, and presets.
