# Agent Guidelines

Guidelines for AI agents working in this repository.

`CLAUDE.md` only imports this file (`@AGENTS.md`). Put shared conventions here, not in `CLAUDE.md`.

If `AGENTS.local.md` exists at the repo root, read it for personal shortcuts and machine-specific paths.

## What this is

A personal learning repo for Google's *Site Reliability Engineering* book: markdown notes plus
self-contained interactive HTML diagrams, published to GitHub Pages.

There is no build system, no package manager, no test suite, and no linter. Nothing is compiled
or bundled: what is in the repo is exactly what gets served. Do not add a toolchain unless asked.

## Teaching language (this book)

These rules apply to notes, diagram captions, step text, and chat explanations **while learning
this book**. They are not a global writing style for other projects.

English is a second language. Write so a careful reader can study without decoding style.

- Prefer short sentences. One idea per sentence when the idea is new.
- Define jargon the first time it appears: σ (sigma, a measure of spread), 3σ (three standard
  deviations from the mean), false outlier, chopped tail, timeout cap, and similar terms.
- Do not say "pretend" or "imagine" a shape on the chart unless that shape is actually drawn.
  If the claim is "this looks like a bell curve," draw the curve.
- Prefer concrete numbers and the same example across a walkthrough (for example the 100-request
  latency set) over abstract restatement.
- Never use em dashes. Use a comma, colon, or a new sentence.

## Commands

```sh
python3 -m http.server 8000     # serve the repo; visit http://localhost:8000
```

Diagrams are plain HTML with no imports, so opening the file directly with `file://` also works.

Publishing is automatic: GitHub Pages serves the `main` branch root, so any push to `main` goes
live within a minute or two. `.nojekyll` is present and must stay. Jekyll would otherwise mangle
the hand-written HTML.

Work on `main` in this checkout. Do not invent a branch/PR workflow unless asked.

## Layout

```
index.html          GitHub Pages landing page: chapter index + featured diagrams
README.md           same content for GitHub readers
AGENTS.md           this file: conventions for agents
chapters/chNN-slug/
  README.md         notes for that chapter
  diagrams/*.html   one self-contained page per diagram
```

All 34 chapters have a folder, most holding only an unfilled note stub. Folder names are
`ch` + zero-padded number + kebab slug (`ch02-production-environment`) so they sort correctly.
The stub sections are Notes / Key takeaways / Interactive diagrams / Questions.

Draft diagrams may first live in the Obsidian tech vault (`zimages/`) next to the book note.
The same language and font rules apply there. When publishing, copy the HTML into
`chapters/chNN-slug/diagrams/` and register it (see below).

## Adding a diagram

A new diagram must be registered in three places by hand. Nothing generates these, and missing
one leaves the diagram unreachable from the site:

1. `index.html` — an `<a class="feature">` card in the feature grid, **and** a
   `<span class="badge">N diagram</span>` on that chapter's row in the chapter list
2. `README.md` — a row in the "Interactive diagrams" table
3. the chapter's own `README.md` — a row in its diagrams table

Chapter rows in `index.html` deliberately link to the GitHub blob URL, not a relative path:
Pages serves raw markdown as a download, whereas GitHub renders it.

## Diagram conventions

`chapters/ch02-production-environment/diagrams/life-of-a-request.html` is the reference
implementation. Copy its structure rather than inventing a new one.

- **Self-contained.** One file, inline CSS and JS, no local imports. Google Fonts
  (JetBrains Mono for labels/UI, Inter for prose) is the only external reference.
- **Shared palette** via CSS custom properties on `:root`. The three signal colors carry
  meaning and are consistent across diagrams: `--req` cyan for the forward/request path,
  `--res` amber for the return/response path, `--gslb` pink dashed for side lookups.
- **Step-through pattern.** A `steps` array drives everything; the SVG itself is static markup.
  Each step object has `phase` (`'req'` or `'res'`), `title`, `desc`, a `nodes` array of node ids
  to highlight, and either `edge` (single edge, animates a dot along the path) or `multiEdge`
  (several edges lit at once, no dot). `render()` clears all state and reapplies from the current
  step, so steps are order-independent and jumping around via the progress bar is safe.
- SVG element ids are `n-<node>` for nodes and `e-<from>-<to>` for edges; the JS looks them up
  by those exact strings, so renaming one means updating the id arrays near the top of the script.

### Readable step text

The reader must be able to study captions without squinting or decoding a display font.

- **Step captions, legends, and any paragraph the reader must study:** Inter regular (sans),
  not italic, not serif. Keep line-height around 1.5 and a readable measure (about 70 characters).
- **Italic serif** is allowed only for a one-line SVG callout, if used at all. Never for a
  multi-sentence caption.
- **Labels and chrome** (axis ticks, buttons, step titles in the UI): JetBrains Mono, as in the
  chapter 2 reference.
- Do not follow `diagram-design`'s Instrument Serif / italic caption default for this book.
  That skill may still draw the first HTML, then restyle to Inter + JetBrains Mono before the
  file is treated as done.
- Open interactive HTML in the **system browser** (`open "path/to/file.html"`), not the IDE
  browser. Obsidian does not run the HTML clicks.

### Sizing gotcha

Font sizes inside the SVG are in viewBox units, then scaled by the container-width ÷ viewBox-width
ratio. A 13px label in a 1180-unit viewBox rendered in a 760px panel comes out at ~8px on screen.
When text looks small, the fix is usually not bumping the font size alone: check the ratio first,
trim empty space out of the viewBox, and give the stage column more width.

## Verifying changes

The Chrome extension is often not connected. Headless Chrome works and is the reliable path:

```sh
python3 -m http.server 8778 &
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --hide-scrollbars --window-size=1500,1000 --virtual-time-budget=4000 \
  --screenshot=/tmp/shot.png "http://localhost:8778/path/to/diagram.html"
```

To inspect a step other than the first, copy the file and rewrite its initial state:
`sed 's/^let current = 0;$/let current = 11;/'` then screenshot the copy. Worth doing for at
least one `'res'` step and one `multiEdge` step after touching any coordinates, since those
exercise paths the opening view does not. Check a narrow viewport too; the layout stacks below
1100px.

## Local overrides

`AGENTS.local.md`, `CLAUDE.local.md`, and `.claude/settings.local.json` are git-ignored and hold
personal, machine-specific context and settings. Put anything provisional or machine-bound there;
this file is for conventions that belong in the repo.

## Repo constraints

- The `amandavarella` token lacks the `workflow` scope, so GitHub Actions workflow files cannot
  be pushed. Pages is configured to build from the `main` branch root instead. Keep it that way.
- No book text is reproduced here; the notes and diagrams are original. Keep it that way, and
  link to <https://sre.google/sre-book/table-of-contents/> rather than quoting at length.
