# Building ISE 5405 Interactive HTML Lectures

Reference for creating new decks (or editing existing ones) in this series. Each lecture is a **single self-contained HTML file** — framework CSS/JS is duplicated per file on purpose so decks can be posted to Canvas or emailed individually.

## Anatomy of a deck

```
<head>
  MathJax v3 config + CDN script (tex-svg)     ← math rendering, needs internet
  Google Fonts: Fraunces (headings) + Public Sans (body)
  <style> … framework CSS … </style>
<body>
  <div id="stage"><div id="deck">              ← fixed 1280×720, scaled to window
    <div id="progress"> <div id="brand"> <div id="counter">
    <section class="slide title">…             ← one <section class="slide"> per slide
    <section class="slide"> …
  </div></div>
  <div id="toc"> <canvas id="ink"> <div id="help"> <div id="cp">   ← chrome overlays
  <ol id="cpA" style="display:none" …>         ← checkpoint question data
  <div id="pentools">                          ← pen color buttons
  <script> framework JS + widget IIFEs </script>
```

**The fastest way to start a new deck:** copy an existing lecture file, delete its
`<section>` slides and deck-specific widget IIFEs (everything after the
`/* ================= LP helpers ================= */` block), keep everything else.
The framework (navigation, pen, laser, handout, checkpoints, TOC, derive players)
comes along for free.

## Slide conventions

- `<section class="slide">` = one slide. `class="slide title"` = title slide with `.title-art` SVG backdrop.
- **Reveals:** add `class="frag"` to any element → revealed by →/Space in DOM order (beamer `\pause`).
- **Kickers:** `<div class="kicker">SECTION · CONTEXT</div>` above the `<h2>` for wayfinding.
- **Boxes:** `.box` (maroon), `.box.orange`, `.box.blue`, `.box.green` — content callouts.
- **Layout:** `.cols` + `.col` flex columns; fix a widget column with `style="flex:0 0 460px"`.
- Colors: VT maroon `#861F41`, orange `#E5751F`, blue `#33658a`, green `#1a7d3c`, red `#b3261e`.
- Math: `\( \)` inline, `\[ \]` display; `{\color{#861F41} …}` for colored math. Typeset once at load; call `MathJax.typesetPromise([el])` after dynamic DOM changes.

## Interactive patterns (copy these, don't reinvent)

1. **Derive player** (click-through math with per-step notes):
   ```html
   <div class="widget derive" id="dpX">
     <div class="dsteps small">
       <div class="step" data-note="explanation shown below">\[ math \]</div>
       …
     </div>
     <div class="dctl"><button class="btn sec dprev">◀ Prev</button><span class="dots"></span><button class="btn dnext">Next step ▶</button></div>
     <div class="dnote"></div>
   </div>
   ```
   Auto-initialized by the framework — no extra JS needed.

2. **Checkpoint pop-quiz:** add `data-checkpoint="cpX"` to a `<section>`, and a hidden data block:
   ```html
   <ol id="cpX" style="display:none" data-why="explanation">
     <li>The question (first li)</li>
     <li>wrong option</li>
     <li data-ok>right option</li>
   </ol>
   ```
   Pops automatically on first arrival; `K` re-opens. Keep questions answerable from material *already shown*.

3. **Canvas widgets:** an IIFE per widget, guarded by `if(!canvasEl)return;`. Wrap interactive markup in `.widget` so clicks don't advance slides. Shared LP helpers available in decks that include them: `clipHP(poly,a,b,c)` (half-plane clip), `regionPoly(cons)`, `dedupe(poly)`.
   For pointer math on the scaled deck use `getBoundingClientRect()` ratios — see any existing widget.

4. **Quizzes / trainers:** hidden `<ol>` with `data-a`/`data-why` per `<li>`; small IIFE renders one at a time (see Lecture 1 "Is it Linear?" or Lecture 3 trainer).

## Built-in features (all decks)

| Key | Feature |
|---|---|
| →/Space/click, ← | navigate & reveals |
| F / M / Home / End | fullscreen, slide menu, first/last |
| P then C | pen overlay, clear ink |
| L | laser pointer |
| H | handout mode (all slides stacked, reveals shown); then Ctrl/Cmd+P → one-slide-per-page PDF |
| K | re-open the slide's checkpoint |
| ? / Esc | help overlay / close |

Decks remember the last slide via localStorage; URL hash `#12` deep-links a slide.

## External content

- **Live demo library** (from the course text, embeddable via iframe, hash-linkable):
  `https://open-optimization.github.io/open-optimization-or-book/visualizations/#<demo-id>`
  (demo ids: modeling-intro, objective-slider, simplex-dictionary, simplex-tableau, concept-quiz, …)
- **Offline figures** in `figures/`: lp-graphical-method, lp-duality-explorer, lp-sensitivity-explorer (self-contained, no CDN).
- Content sources: `ISE_5405_Handoff/02_Lectures_Hildebrand_2025/LaTeX_sources/` and the open textbook repo (`Intro-Math-Programming/baseText/`).

## Verification checklist (do this after every edit)

1. `node --check` every extracted `<script>` block (regex out of the HTML).
2. Tag balance: count `<section|div|svg|g|ol>` vs closers.
3. jsdom smoke test: load with a canvas-context stub, simulate arrow keys, click every
   derive `.dnext`, fire slider `input` events, toggle checkpoints via `deckAPI.show(idx)`,
   confirm "no runtime errors". (`requestAnimationFrame` errors under jsdom are expected —
   browsers have it.)
4. Open in a real browser once: check MathJax renders, widgets respond, H-mode prints.

## Style guardrails

- Model first, formalize second — conversions/algebra land *after* students have models in hand.
- Every abstract definition gets a picture or a widget; every proof-y slide leans on the Lecture 3 toolkit by name (contradiction template, negation practice).
- Don't quiz ahead of the material (e.g., no branch-and-bound questions in early lectures — mention as forward-pointers only).
- Checkpoints: 3 per deck, at natural pause points, answerable from what's on screen.
