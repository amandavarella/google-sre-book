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
- **Anticipate the next question.** If a sentence packs two ideas (for example "fine at p99 and
  sluggish at p90"), unpack it before she has to ask: say what p90 and p99 *are*, then say who
  feels each one, then say why that matters. Do not use a metaphor ("trains users to wait")
  when a number will do. Do not write `p99 / p99.9` or `90% / 1 ms` to mean "or" or "under":
  write the full phrase. Say what "page you" means the first time (an on-call alert). Never
  use "beat" for a percentile: say "finished in that time or less." When she asks "I don't get
  X", rewrite X in the notes, then scan the *rest of the current chapter after that passage*
  for the same kind of packed word (another metaphor, another slash, another undefined `pNN`)
  and fix those too. Do not rewrite earlier sections she has already worked through.

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

This repo is the only notes home. Write chapter notes and diagrams here and push to `main`.
Do not keep a parallel copy in Obsidian.

## Adding a diagram

A new diagram must be registered in three places by hand. Nothing generates these, and missing
one leaves the diagram unreachable from the site:

1. `index.html` — an `<a class="feature">` card in the feature grid, **and** a
   `<span class="badge">N diagram</span>` on that chapter's row in the chapter list
2. `README.md` — a row in the "Interactive diagrams" table
3. the chapter's own `README.md` — a row in its diagrams table

Every diagram link in a README (root or chapter) must be the **live Pages URL**, never a
relative `diagrams/*.html` path and never a GitHub blob/source link:

`https://amandavarella.github.io/google-sre-book/chapters/chNN-slug/diagrams/<file>.html`

Chapter rows in `index.html` deliberately link to the GitHub blob URL, not a relative path:
Pages serves raw markdown as a download, whereas GitHub renders it. The feature cards in
`index.html` are already live relative paths and stay that way.

## Diagram conventions

`chapters/ch02-production-environment/diagrams/life-of-a-request.html` is the reference
implementation. Copy its structure rather than inventing a new one.

- **Self-contained.** One file, inline CSS and JS, no local imports. Google Fonts
  (JetBrains Mono for labels/UI, Inter for prose) is the only external reference.
- **Dark house palette only.** Always the `:root` colours from the chapter 2 file
  (`--bg`, `--panel`, `--req` cyan, `--res` amber, `--gslb` pink). Never ship the light
  paper / orange-accent look from `diagram-design`. That skill may still draw a first draft;
  restyle to this palette before the file is treated as done.
- **Shared signal colours** carry meaning: `--req` cyan for the typical / request / success
  path, `--res` amber for the return path or the thing you must notice (a spike, a timeout),
  `--gslb` pink dashed for a side path or a false model.
- **Step-through pattern.** A `steps` array drives everything; the SVG itself is static markup.
  Each step object has `title`, `desc`, and whatever ids to show. `render()` clears all state
  and reapplies from the current step, so steps are order-independent and jumping around via
  the progress bar is safe. For path diagrams, each step also has `phase` (`'req'` or `'res'`),
  a `nodes` array, and either `edge` or `multiEdge`.
- **Chrome.** Header (eyebrow + JetBrains Mono `h1` + Inter `sub`), a `.stage` panel for the
  SVG, a `.side` card for step text / Back / Next / Play. Same button and progress styles as
  chapter 2.
- SVG element ids on path diagrams are `n-<node>` for nodes and `e-<from>-<to>` for edges.

### Readable step text

The reader must be able to study captions without squinting or decoding a display font.

- **Step captions, legends, and any paragraph the reader must study:** Inter regular (sans),
  not italic, not serif. Keep line-height around 1.5 and a readable measure (about 70 characters).
- **Italic serif** is allowed only for a one-line SVG callout, if used at all. Never for a
  multi-sentence caption.
- **Labels and chrome** (axis ticks, buttons, step titles in the UI): JetBrains Mono, as in the
  chapter 2 reference.
- Open interactive HTML in the **system browser** (`open "path/to/file.html"`), not the IDE
  browser.

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
