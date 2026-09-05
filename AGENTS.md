# Agent Guidelines

Guidelines for AI agents working in this repository.

`CLAUDE.md` only imports this file (`@AGENTS.md`). Put shared conventions here, not in `CLAUDE.md`.

If `AGENTS.local.md` exists at the repo root, read it for personal shortcuts and machine-specific paths.

## What this is

A personal learning repo for Google's *Site Reliability Engineering* book: markdown notes plus
self-contained interactive HTML diagrams, published to GitHub Pages.

When this folder is the Cursor workspace, **this file is the source of truth**. Read
"How she learns" before writing notes or answering a stuck question. Do not run `em-learning`
or write chapter notes back to Obsidian.

There is no build system, no package manager, no test suite, and no linter. Nothing is compiled
or bundled: what is in the repo is exactly what gets served. Do not add a toolchain unless asked.

## How she learns

Derived from the Chapter 3–4 sessions. This is the working model for notes, diagrams, and chat.
`em-learning` (Obsidian capture) is not the workflow for this repo. GitHub is the notes home.

### The loop

1. She pastes a book passage, or names a section. Write notes for **that passage only**. Do not
   pull the next unread section. The goal is not to reproduce the book.
2. Answer in chat first (short, concrete). Then put the same rewrite into the chapter `README.md`.
   Chat-only explanations get lost. The note is what she studies later.
3. When she says **"I don't get X"** (or "not clear", "what does that mean", "I don't understand
   this part at all"): rewrite X in the notes. Then scan **only the rest of the current chapter
   after that passage** for the same kind of packed word, and fix those too. Do not rewrite
   earlier sections she has already accepted.
4. When she says she understands, or **"add this block as a whole"**: stop unpacking that
   passage. Move on.

**Capture and organize. Do not add opinions.** The notes are her study copy of the book,
arranged so she can reread it. They are not a playbook you wrote.

- Organize: the book's headings, bullets, short sentences, jargon defined in the same line,
  numbers that are in the passage.
- Do not add advice, `DECISION:` lines, extra process ("at year end, add up hours"), or
  "you should" that she did not paste and the book did not state.
- If the book itself states an action (for example ship or wait on an error budget), keep
  that action. Do not invent a new one.

Organizing uses the safeguards from
[Clarity](https://github.com/addyosmani/clarity/blob/main/SKILL.md) (rewrite of source
material into explanation and reference). Full Clarity modes, interview, lint scripts, and
essay-voice matching are **not** in this repo.

- **Inventory first.** List the passage's claims, numbers, examples, and terms. Then
  organize. Do not add a claim, number, cause, or anecdote that was not in the paste.
- **Register.** These notes are explanation (accurate mechanism at her level) plus
  reference (scannable headings). They are not an argument. Do not invent a thesis.
- **Least invasive.** Cut, reorder, and shorten for comprehension. A clearer sentence must
  still mean what the book meant. Do not strengthen, hedge, or drop a condition.
- **Ask the gap.** If a clearer sentence needs a fact she did not paste, ask in chat. Do
  not invent it, and do not put `[TK]` into the published notes.
- **Attribution.** Keep "the book says" separate from anything else. Do not silently turn
  a Google SRE practice into a rule for her job.
- **Stop on the last useful thought.** Do not add a recap, moral, or send-off after the
  passage is organized. The Key takeaways section at the bottom of the file is the stub
  summary; update it with claims from the passage, not a second essay.

Each stuck question is a permanent style rule. If a metaphor, slash, or word failed once, do
not use it again in this chapter.

### What unsticks her

She does not learn from a second abstract restatement of the book. She learns when the idea
becomes a **decision with numbers** that are already in the passage (or in earlier linked
notes). Do not invent a new number to make the point land.

- **Same running example** across a walkthrough (100 requests and a 1 s timeout; the Friday
  filter meeting `20 + 80 = 100` vs a 50 ms promise; the 28 → 38 → 45 ms afternoon). Reuse
  those numbers. Do not invent a fresh story for each paragraph. The example must already
  exist in the pasted passage or in earlier notes. If it comes from an earlier chapter, link
  that chapter (and its diagram, if there is one) on first use. Do not invent a new fictional
  incident to teach the passage.
- **Two contrasting cases** on the same reading: Team A’s 50 ms mark vs Team B’s 200 ms mark;
  photo app vs archive; pipeline vs click. She gets the idea when two people see one number
  and do different things. Use two cases when the passage or earlier notes already have
  them. Do not invent a Team A and Team B the book did not write.
- **Whether and when.** She tests a sentence by asking what it *does*. "Spent the error budget
  on batch jobs": did we spend it wrong, or on purpose? "DECISION: WAIT": is the goal to give
  SRE time to make the click faster? (No: wait means do not ship; product cuts the extra 80 ms.)
  "CPU idle is not used in this meeting": why is that box on the slide at all? Answer the
  action **only when the passage states it**: ship, wait, page, add servers today, or leave it
  as a graph. Do not invent a `DECISION:` or a weekly ritual the book did not write.
- **Why next to the claim.** Put the reason in the same bullet (a short parenthesis is fine),
  not in a later paragraph. Same for examples: if a line is abstract ("defuses discussions",
  "service tiers at different costs"), attach a concrete example in that line.
- **Step by step, then a diagram.** Chubby, percentiles, the control loop: first unpack in
  order, then a step-through diagram with the same numbers. Link the diagram **inline** next
  to the passage, not only in the chapter table. Once the diagram exists, **do not keep the
  walkthrough in the notes**. The note holds the claim, any jargon, and the link. The diagram
  holds the steps, tables, and Friday-meeting numbers.
- **Draw the shape.** Never say "pretend" or "imagine" a curve. If the claim is "this looks
  like a bell curve," put the curve on the chart. Scaffolding she no longer needs (an extra
  histogram) can be removed after she says she gets it.

### What fails (do not write these)

- Packed metaphors and cinematic lines: "trains every user to wait", "nobody in the room can
  feel", "two humps", "thin tail" with no numbers.
- Slash-as-or: never `p99 / p99.9` or `90% / 1 ms` to mean "or" or "under". Write the full
  phrase.
- "Beat" for a percentile. Say "finished in that time or less."
- Undefined jargon: `p90`, `p99`, HTTP 500, RPC, payload, error budget, "page you" (on-call
  alert). Define the first time, in the sentence that uses it.
- Asking her to hold a picture that is not drawn.
- Opinions, invented `DECISION:` lines, extra process, anecdotes, or numbers that are not in
  the pasted passage.
- Quoting the book at length. She wants a **condensed, clearer** version than the page, in
  her words for later study.

### How she reads the page

- Headings must match the **book’s section titles** and be prominent (`##` for the book’s
  major parts, `###` for the book’s subsections). A wall of `####` under a generic `## Notes`
  hides the outline she is following in the book.
- She notices broken lists, misaligned bullets, and formulas that smash the layout. She will
  paste a screenshot. Fix the render, not only the wording.
- Diagram type: Inter for any caption she must study. House dark palette (copy Chapter 2).
  New-tab live Pages links.
- **Link every cross-reference.** If a note or a chat reply names another chapter, section,
  diagram, or file in this repo, put a clickable link in that same sentence. Do not write
  "see Chapter 4" with no URL. First mention in a file is enough; later "same photo app"
  in that file can stay unlinked. URL shapes are under "Cross-references" below.

### Photos and highlights

If she sends a photographed page with **highlighted text or brackets in the margin**, extract
only those bits. The rest of the page is context, not the note.

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
- **Anticipate the next question.** If a sentence in the *passage* packs two ideas (for
  example "fine at p99 and sluggish at p90"), unpack those words before she has to ask: say
  what p90 and p99 *are*, then say who feels each one, then say why that matters. Do not
  write lines like "nobody in the room can feel" or "trains every user to wait". Say which
  number does not change. Do not write `p99 / p99.9` or `90% / 1 ms` to mean "or" or
  "under": write the full phrase. Say what "page you" means the first time (an on-call
  alert). Never use "beat" for a percentile: say "finished in that time or less." Unpack
  only what the passage already contains. Do not add answers to questions the passage does
  not raise. When she asks "I don't get X", rewrite X in the notes, then scan the *rest of
  the current chapter after that passage* for the same kind of packed word (another
  metaphor, another slash, another undefined `pNN`) and fix those too. Do not rewrite
  earlier sections she has already worked through.

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
README.md           chapter index for GitHub readers (no diagram list)
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

A new diagram must be registered in two places by hand. Nothing generates these, and missing
one leaves the diagram unreachable from the site:

1. `index.html` — an `<a class="feature">` card in the feature grid, **and** a
   `<span class="badge">N diagram</span>` on that chapter's row in the chapter list
2. the chapter's own `README.md` — a row in its diagrams table

Do **not** add a row to the root `README.md`. That file is a chapter index only.

Every diagram link in a chapter README must be the **live Pages URL**, never a
relative `diagrams/*.html` path and never a GitHub blob/source link:

Use an HTML `<a>` with `target="_blank"` and `rel="noopener noreferrer"` so the diagram
opens in a new tab. Plain markdown `[text](url)` stays in the same tab.

`https://amandavarella.github.io/google-sre-book/chapters/chNN-slug/diagrams/<file>.html`

Chapter rows in `index.html` deliberately link to the GitHub blob URL, not a relative path:
Pages serves raw markdown as a download, whereas GitHub renders it. The feature cards in
`index.html` are already live relative paths and stay that way.

## Cross-references

When notes or chat name another part of this repo, link it in the same sentence.

Use an HTML `<a>` with `target="_blank"` and `rel="noopener noreferrer"`. Plain markdown
`[text](url)` stays in the same tab.

- **Diagrams:** live Pages URL, as in "Adding a diagram" above.
- **Chapter notes** (a `README.md` in another chapter, or a heading in this one): GitHub
  blob URL, plus a heading anchor when you mean a section, not the whole file. Same reason
  as the chapter list in `index.html`: Pages would serve the markdown as a download.

  `https://github.com/amandavarella/google-sre-book/blob/main/chapters/chNN-slug/README.md#heading-id`

Same-file anchors (`[The Global Chubby Planned Outage](#the-global-chubby-planned-outage)`)
are fine with no blob URL.

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
