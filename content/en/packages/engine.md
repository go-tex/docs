---
title: "engine"
weight: 30
---

# engine {#engine}

The pure-Go (`CGO_ENABLED=0`) TeX engine. A faithful re-implementation of TeX
that **loads and runs the genuine LaTeX classes** — `\documentclass{article}`,
`{report}` and `{book}` execute the real, embedded `.cls` files (LPPL, verbatim)
— not an emulation — in native builds and in the browser via `js/wasm`, with no
TeXLive and no server.

## What it does {#what-it-does}

- **Runs the standard base classes for real.** `article`, `report` and `book`
  load and execute the genuine embedded LaTeX `.cls` files, typesetting a
  numbered title, a `\tableofcontents` shaped by the class's own `\l@…` macros
  (the indent and number-box widths are read out of its `\@dottedtocline` calls,
  so `article`'s leaderless bold `\l@section` and `book`'s dotted one at the same
  level come out differently), numbered sections/subsections, `\chapter`
  (report/book, with its entry in the contents list and its opening page's folio
  at the foot), numbered figure/table captions, `itemize`/`enumerate`, and math.
- **A full pipeline.** A category-code mouth and gullet, a scaled-point stomach
  with Knuth–Plass line breaking and a cost-based page builder, `\halign` tables,
  math via [`math`]({{< relref "math.md" >}}) (vector output), OpenType fonts (a built-in default
  font, kerning, ligatures), and **PDF + SVG** output — the SVG carries a source
  map for click-to-line.
- **Floats and two columns are the default.** Real float placement (top, bottom
  and float-page, with the seven parameters read from the *class* rather than
  from `article`'s) and the two-column layouts that classes set themselves
  (revtex `reprint`, acmart `sigconf`/`sigplan`, IEEEtran) are on without a flag;
  `GOTEX_FLOATS=0` and `GOTEX_TWOCOLUMN=0` opt back out. This page described both
  as still behind their environment variables, which they have not been for some
  time.
- **Spacing and breaking commands behave as LaTeX's own.** `\hspace` and `\vspace`
  take a **glue**, not a rigid length, so `\hspace{\fill}`, `\vspace{\stretch{2}}`
  and `\hspace{1cm plus 2mm}` keep their stretch; `\fill`, `\stretch`, `\enskip`
  and `\samepage` are defined, with `\fill` and `\stretch` at the reference's
  *second* order of infinity. `\pagebreak`, `\nopagebreak`, `\linebreak` and
  `\nolinebreak` contribute the penalties LaTeX gives them, the optional degree
  `[0…4]` included. `\marginpar` reads both of its arguments instead of letting
  them reach the page as text. (v0.263.1)
- **`\\` carries its star and its optional length, and `\vadjust` works.** `\\[20pt]`
  used to typeset the five characters `[20pt]` and `\\*` a `*`; both are now scanned,
  and the length produces the reference's 31.88pt baseline step where a bare `\\`
  gives 11.96. `\vadjust{…}` puts vertical material written inside a paragraph into
  the enclosing vertical list **after** the line it was written on — measured against
  `tectonic` as a 41.84pt step against a normal 11.96 — and it is an **error** in
  vertical mode, as TeX requires. Two commands came with it: `\pagebreak` and
  `\nopagebreak` now work when written *on a line* and not only between paragraphs
  (the reference makes two pages on a short first paragraph, and so do we, where we
  made one), and `\\*` refuses a page break after its line.
- **In the browser.** A self-contained `js/wasm` build renders real LaTeX classes
  client-side — no TeXLive, no server.

## Pipeline stages {#pipeline-stages}

Each stage is gated by an objective oracle.

| Stage | Status | What it covers |
|---|---|---|
| Mouth + gullet | ✅ done | Category-code tokenizer, `eqtb`, macros (delimited parameters, full expansion), conditionals, registers, grouping. |
| Stomach | ✅ done | Box/glue/penalty model in scaled points, Knuth–Plass line breaking, a cost-based page builder that keeps TeX's **two** page measurements apart — `page_total`, which the break is decided on, and `page_depth`, which is not charged against the goal and is capped at `\maxdepth` — and `\halign` tables. |
| Math | ✅ done | `$…$` and display environments delegated to [`math`]({{< relref "math.md" >}}) (vector output). |
| Fonts | ✅ done | OpenType via go-opentype; a built-in default font, kerning and ligatures — runs with no assets. |
| Output | ✅ done | PDF (embedded subset fonts, selectable text) and self-contained SVG pages with a click-to-line source map. |
| Real classes | ✅ done | `\documentclass{article\|report\|book}` loads and runs the genuine embedded LaTeX class, native and `js/wasm`. |

## Fidelity gates {#fidelity-gates}

- a byte-exact **conformance ratchet** (`TestConformance`), TeX snippets checked
  byte-for-byte against real-TeX output; and
- a **whole-document prose fidelity check** against a real LaTeX engine
  (`tectonic`). On a corpus of **154 real arXiv papers** with a reference PDF
  each, a vocabulary-bounded word census puts the median at **101% of the
  reference engine's prose words** (measured 2026-10-04); **10** of the 154 fall
  below 95% and 3 below 90%. Pagination lands on the reference exactly for **32**
  of them, with a total page deviation of **304** over the corpus (v0.261.0,
  font-controlled).

  In the **`default`** configuration — a different measurement, not an update of
  the one above, and the two must not be mixed — v0.263.0 lands exactly on **36**
  of the 154 with Σ\|deviation\| **294** and a signed bias of **+72** pages. The
  bias is where the work is: it is a one-column phenomenon
  ([#622](https://github.com/go-tex/engine/issues/622)), and v0.263.0 took 7 pages
  off it by keeping `page_total` and `page_depth` apart in the page builder. A page
  whose material reached *exactly* `\textheight` had been refused over the
  descender of its own last line, and lost a line
  (`tex.web` 19519-19529, 19594-19599).

  **The bias now has attributable causes rather than a shape.** They were found by
  building documents that carry *one* ingredient and counting pages against the
  reference: 300 one-line paragraphs, 80 prose paragraphs and 40 `\section`s all come
  out **equal**, and it is those equalities that make the two that differ
  attributable. 40 display equations cost a page too many — fixed in v0.267.0 — and 40
  `itemize` blocks still do, with every height agreeing, which puts that one in the
  page *breaking* rather than in the spacing
  ([#685](https://github.com/go-tex/engine/issues/685)). None of these probes needs
  two paginations to be aligned, which is why they could answer where the
  alignment-based instruments could not: across the corpus only **6 of 154** documents
  have a first divergence that is attributable at all, and **none** agrees with the
  reference on every comparable page.

  A median above 100% is expected rather than flattering: the engine emits each
  formula's *source* as an invisible text layer so that maths can be found,
  copied and read aloud, and the census counts it. The figure to watch is the
  tail — the papers below 95% — not the median.

  {{< callout type="warning" >}}
  That text layer is also why **a PDF text extractor cannot be used to check
  mathematics** against another engine: on a formula it returns *our source*, not
  glyph positions, so it compares two different things. Use the vector output, or
  the engine's own box metrics, when the question is how a formula is set.
  {{< /callout >}}

  v0.263.1 is **not** measured in these terms yet. It carries the spacing and
  breaking work described above, and the one piece of it measured on its own —
  `\\` no longer ejecting a page in vertical mode — moved Σ\|deviation\| from 294
  to **283**, the bias from +72 to **+59**, and the exact papers from 36 to **39**.
  That figure belongs to *that change against its own control*, not to the release:
  a sum assembled from separate deltas drifts, so the release's own number is left
  blank until the release itself is swept.

  **v0.265.0 was swept whole**, which is the only way one of these numbers earns its
  place: 154 documents in the `default` configuration, Σ\|deviation\| **276**, signed
  bias **+50**, **38** exact. It is not comparable to the 294/+72/36 above, which was
  measured at v0.263.0 on the same corpus but before the footnote work; the figures
  that *are* comparable are its own control — the commit immediately before the last
  of the four, which already carried the other three — at 277/+49/38. That control is
  **not** v0.264.0, and saying so matters: v0.264.0 predates all four changes, so a
  reader who took it for the baseline would credit this release with work three
  earlier merges had already done.

  **v0.267.0 moves the figure the bias is about.** A display now chooses between the
  long and the short pair of skips as `tex.web:22578-22585` does, and on the same
  sweep the signed bias falls **+50 → +35** with the median absolute deviation
  reaching **0**. Σ\|deviation\| moves only **276 → 275**, and that is the number to
  read beside it: every document the change moved got *shorter*, by the right amount,
  which rights the eight we set too long and worsens six we already set too short —
  two of them from exact. The surplus glue had been cancelling a second, opposite
  error, which is now measurable on its own
  ([engine#683](https://github.com/go-tex/engine/issues/683)). A figure of merit that
  improves while its own components get worse is worth printing both ways.

- a **geometry check**, which is the only one of the three that can see text
  drawn *outside* the sheet. A block that should fill a title page used to be set
  as one horizontal list that was never line-broken, so everything past the right
  margin was placed off the paper — up to **x=6619pt on a 612pt page**, author
  names, affiliations and e-mails among it. The page count cannot see that (the
  clearest witness is 15 pages against 15), the ink count cannot (every glyph is
  emitted) and the text layer carries the words. Measured as lines ending beyond
  any page width this corpus uses: **49 papers at v0.260.0, 23 at v0.261.0, 22 at v0.262.0**,
  against a floor of **2** on the reference PDFs themselves. The widest line went
  from 6619pt at v0.260.0 to 3357pt at v0.261.0 and **1788pt** at v0.262.0.

- a **render check**: a sha256 over the bytes of every page's SVG, so it covers
  every coordinate on the page. Every sweep had recorded one since the beginning
  and nothing had ever compared two, so the page-count comparisons were the only
  ones being read — and they are blind by construction to anything that moves
  material *within* a page. That blindness reports such a change as "no document
  moved", which reads as *the change does nothing* when the truth is *the
  instrument cannot see it*. Seven changes measured since: 2, 25, **72**, 40, **72**,
  18 and **73** of the 154 corpus documents render differently; across the first six
  **exactly one document** moved by a page
  ([engine#675](https://github.com/go-tex/engine/issues/675)). That one is worth the
  sentence it takes: `2209.13121` went 36 pages to 37 against a reference of 42, so
  Σ\|deviation\| fell 277 → 276 while the *signed* bias rose +49 → +50 — the bias rose
  because the paper was **short**, not because anything got worse. The seventh is the
  exception that shows what the channel is for: 73 render differently and **14** moved
  by a page, which is the first change here whose page-count effect was worth reading
  on its own — and even there 59 of the 73 were invisible to it. It answers *did the
  output change*, not *is it better*: the reference's PDFs are not our SVGs, so
  there is no third hash to be right about.

## Roadmap {#roadmap}

The engine loads and runs the standard base classes; it is
functional-parity-oriented and held to real-LaTeX fidelity, **not** a claim of
full TeXLive parity.

- Heavy packages (`tikz`, …) run as real files rather than native stubs; TikZ/pgf
  drawing is gated behind `GOTEX_PGF` and in bring-up.
- Front matter that still reaches past the page edge: 22 of the 154 corpus papers,
  down from 49, with five causes closed and the remainder a different population
  each ([engine#629](https://github.com/go-tex/engine/issues/629)). The fifth cause
  is `\unvbox` inside a paragraph: a vbox holds finished *lines*, and handing them
  to the line breaker glues them end to end and re-breaks the lot as one paragraph
  (acmart writes `\noindent\unvbox\mktitle@bx`).
- **How far from the kernel, measured.** `latex.ltx` — the assembled LaTeX kernel
  `tectonic` itself executes — defines **1952** control sequences. This engine and
  `go-tex/math` carry **755** of them; **1197** are absent, of which **642** are
  written by at least one paper in the 21 956-paper corpus. That is the shape of
  "not full TeXLive parity", in numbers rather than in a disclaimer
  ([engine#677](https://github.com/go-tex/engine/issues/677)). The list is a
  *reading order*, not a defect list. An earlier reading of the same inventory put it
  at **1256** absent and **695** used: the difference is three asymmetries in the
  *reader*, each of which made the engine look poorer than it is, and each found by
  taking three names off the head and checking them by hand.

  **Ranking the absences by corpus demand turned out to rank noise.** The first
  **fourteen** entries are names this engine already writes down somewhere — a special
  case in Go (`\multicolumn`, 51 of the 154 measured papers, is *supported* without
  being *defined*), a test file of its own (`\baselinestretch`, 41), or a comment
  saying why the name is deliberately absent (`\@evenfoot` and `\@evenhead`, 42 and
  41: this engine has no two-sided page model). `\do` (46) is a loop hook every caller
  `\let`s to its own action. The measurement tool now marks such a name **cited** —
  absent as a *definition*, but already thought about — and **221** of the 642 used
  absences are cited. Nothing is filtered: the citation is a column beside the demand,
  because dropping a cited name would be a false *presence*, and a false presence
  deletes work for good where a false absence costs one reading.

  **The list has no safe head, and that is the useful thing it says.** The first
  absence this engine writes down nowhere at all is `\@citea`/`\@citeb`, 18 of the
  154 — and defining them would accomplish nothing: they are the loop variables of
  `\@citex`, which this engine never executes, `\cite` being written in Go. A name
  internal to a macro replaced wholesale is invisible to *both* tests, and only
  reading the caller finds it. The defect in that neighbourhood is a real one of a
  familiar shape — a class's `\@biblabel` and `\@cite` were executed into the eqtb and
  then ignored, exactly as `\footnoterule` used to be, now closed — but it is not the
  one the ranking named. Every entry on this list needs its *caller* read before it is
  believed.

  **Walked to the end, both tables, the inventory yielded two changes the corpus can
  see** — the footnote's layout and the citation's brackets — and nothing else. The
  rest is error machinery, internals of macros this engine replaces in Go, and one
  declared `\halign` limitation (`\omit`/`\span`) whose own code comment already
  carries the measurement saying it changes nothing here: all 282 of the corpus's
  `\halign` uses sit in files the engine substitutes rather than executes, and
  compiling a paper that ships one never reaches the word. Those are **conformance**
  work — they are TeX primitives, and `\halign{…\omit…}` is an abort today — but no
  fidelity number justifies them. So the next work comes from the measured channels
  above, not from the inventory; it stays open as a record of five ways an inventory
  can mislead, not as a queue.
- **A footnote is laid out by its class.** The mark is whatever `\thefootnote`
  expands to, so `\renewcommand\thefootnote{\fnsymbol{footnote}}` works and a title
  block's `\thanks` comes out `∗`; and the note itself is now built by calling the
  class's `\@makefntext` instead of placing the mark in Go, so the `1.8em` box that
  right-aligns it (`article.cls:609-612`) is there. Before, every note's text began
  one box-width to the left of the reference's: no box of 14.400pt existed anywhere
  on the page, and one does now. What remains is the mark's *size*:
  `\@textsuperscript` is the identity here, so it is set on the baseline at text size
  where the reference raises it, and `\@fnsymbol` gives an ASCII `*` where the
  reference sets `*` in math mode and draws U+2217.
- **A citation's brackets belong to the class too.** `\@biblabel` and `\@cite` are
  macros in the reference (`latex.ltx:13324`, `:13322`), and `article.cls:568` builds
  the bibliography list's label by calling `\@biblabel`. They were brackets built in
  Go, so a class's redefinition reached the eqtb — this engine runs real `.cls` files —
  and was dropped at the point of use: a document writing `\def\@biblabel#1{<<#1>>}`
  rendered **byte-identically** to one that did not. `22 of the 154` carry a class that
  redefines one of the two, and since only 50 of the 154 ship a class at all, that is
  **44 %** of the papers that bring their own; `18 of 154` render differently. `\@citex`
  — the iteration natbib replaces by `\let`ting it — is **not** executed: exactly one
  paper in 154 ships `natbib.sty`, and no number calls for running the real package in
  place of the Go emulation
  ([engine#681](https://github.com/go-tex/engine/issues/681)).
- **6.1pt of leftover on a page that stretches.** With vertical infinite glue now
  contributing and every page packed `exactly` to `\vsize`, `\vspace{\fill}` puts
  its word at y=658.6 against `tectonic`'s 664.7 — the remaining third of
  ([engine#663](https://github.com/go-tex/engine/issues/663)), which described
  three defects under one title.
- **A `\marginpar`'s note is read but not placed in the margin.** Its arguments no
  longer reach the page as text and the body matches the reference exactly; putting
  the note itself out in the margin goes through LaTeX's float machinery and is still
  to come.
- A broader real-document conformance corpus (PDF-diff against pdftex/xetex) and
  the TRIP test.

## Install {#install}

```bash
go get github.com/go-tex/engine
```

Requires Go 1.26.4 or newer. `CGO_ENABLED=0`, `go vet` clean, CI green across
three 64-bit arches under QEMU (`riscv64`/`ppc64le`/`s390x`) plus macOS, Linux
and Windows, and both `js/wasm` and `wasip1/wasm`.

## Links {#links}

- Source: <https://github.com/go-tex/engine>
- API reference: <https://pkg.go.dev/github.com/go-tex/engine>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
