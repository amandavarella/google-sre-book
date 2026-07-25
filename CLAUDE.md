# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal learning repo for Google's *Site Reliability Engineering* book: markdown notes plus
self-contained interactive HTML diagrams, published to GitHub Pages.

There is no build system, no package manager, no test suite, and no linter. Nothing is compiled
or bundled — what is in the repo is exactly what gets served. Do not add a toolchain unless asked.

## Commands

```sh
python3 -m http.server 8000     # serve the repo; visit http://localhost:8000
```

Diagrams are plain HTML with no imports, so opening the file directly with `file://` also works.

Publishing is automatic: GitHub Pages serves the `main` branch root, so any push to `main` goes
live within a minute or two. `.nojekyll` is present and must stay — Jekyll would otherwise mangle
the hand-written HTML.

## Layout

```
index.html          GitHub Pages landing page — chapter index + featured diagrams
README.md           same content for GitHub readers
chapters/chNN-slug/
  README.md         notes for that chapter
  diagrams/*.html   one self-contained page per diagram
```

All 34 chapters have a folder, most holding only an unfilled note stub. Folder names are
`ch` + zero-padded number + kebab slug (`ch02-production-environment`) so they sort correctly.
The stub sections are Notes / Key takeaways / Interactive diagrams / Questions.

## Adding a diagram

A new diagram must be registered in three places by hand — nothing generates these, and missing
one leaves the diagram unreachable from the site:

1. `index.html` — an `<a class="feature">` card in the feature grid, **and** a
   `<span class="badge">N diagram</span>` on that chapter's row in the chapter list
2. `README.md` — a row in the "Interactive diagrams" table
3. the chapter's own `README.md` — a row in its diagrams table

Chapter rows in `index.html` deliberately link to the GitHub blob URL, not a relative path:
Pages serves raw markdown as a download, whereas GitHub renders it.

## Diagram conventions

`chapters/ch02-production-environment/diagrams/life-of-a-request.html` is the reference
implementation; copy its structure rather than inventing a new one.

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

### Sizing gotcha

Font sizes inside the SVG are in viewBox units, then scaled by the container-width ÷ viewBox-width
ratio. A 13px label in a 1180-unit viewBox rendered in a 760px panel comes out at ~8px on screen.
When text looks small, the fix is usually not bumping the font size alone — check the ratio first,
trim empty space out of the viewBox, and give the stage column more width.

## Verifying changes

The Chrome extension is often not connected. Headless Chrome works and is the reliable path:

```sh
python3 -m http.server 8778 &
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --hide-scrollbars --window-size=1500,1000 --virtual-time-budget=4000 \
  --screenshot=/tmp/shot.png "http://localhost:8778/path/to/diagram.html"
```

To inspect a step other than the first, copy the file and rewrite its initial state —
`sed 's/^let current = 0;$/let current = 11;/'` — then screenshot the copy. Worth doing for at
least one `'res'` step and one `multiEdge` step after touching any coordinates, since those
exercise paths the opening view does not. Check a narrow viewport too; the layout stacks below
1100px.

## Local overrides

`CLAUDE.local.md` and `.claude/settings.local.json` are git-ignored and hold personal,
machine-specific context and settings. Put anything provisional or machine-bound there;
this file is for conventions that apply to everyone reading the repo.

## Repo constraints

- The `amandavarella` token lacks the `workflow` scope, so GitHub Actions workflow files cannot
  be pushed. Pages is configured to build from the `main` branch root instead — keep it that way.
- No book text is reproduced here; the notes and diagrams are original. Keep it that way, and
  link to <https://sre.google/sre-book/table-of-contents/> rather than quoting at length.
