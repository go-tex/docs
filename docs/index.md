# go-tex

**Pure-Go TeX that runs the real LaTeX classes — in Go and in the browser.**

go-tex is a pure-Go (`CGO_ENABLED=0`) TeX effort. Its engine is a faithful
re-implementation of TeX — the category-code mouth and gullet, a scaled-point
stomach, math, OpenType fonts, and PDF/SVG output — and on top of it, it **loads
and runs the genuine LaTeX classes**: `\documentclass{article}`, `{report}` and
`{book}` execute the real, embedded `.cls` files (LPPL, verbatim) — not an
emulation — in native builds **and** in the browser via `js/wasm`, with no
TeXLive and no server.

A real `\documentclass{article}` document typesets a numbered title, a dotted
`\tableofcontents`, numbered sections and subsections, `\chapter` (report/book),
numbered figure and table captions, `itemize`/`enumerate`, and math.

Everything is **pure Go** (`CGO_ENABLED=0`), standard-library-first, `go vet`
clean, and cross-compiles to every 64-bit Go target as well as `js/wasm` and
`wasip1/wasm`. Licensed BSD-3-Clause.

## The pipeline

The engine is built the way TeX parity is actually reachable — reimplement each
stage faithfully, then gate it with an objective oracle.

| Stage | Status | What it covers |
|---|---|---|
| Mouth + gullet | ✅ done | Category-code tokenizer, `eqtb`, macros (delimited parameters, full expansion), conditionals, registers, grouping. |
| Stomach | ✅ done | Box/glue/penalty model in scaled points, Knuth–Plass line breaking, a cost-based page builder, `\halign` tables. |
| Math | ✅ done | `$…$` and display environments delegated to [`go-tex/math`](packages/math.md) (vector output). |
| Fonts | ✅ done | OpenType via go-opentype; a built-in default font, kerning and ligatures — runs with no assets. |
| Output | ✅ done | PDF (embedded subset fonts, selectable text) and self-contained SVG pages with a click-to-line source map. |
| Real classes | ✅ done | `\documentclass{article\|report\|book}` loads and runs the genuine embedded LaTeX class, native and `js/wasm`. |

## Fidelity gates

Two oracles hold the line:

- a byte-exact **conformance ratchet** (`TestConformance`), TeX snippets checked
  byte-for-byte against real-TeX output; and
- a **whole-document prose fidelity check** against a real LaTeX engine
  (`tectonic`). On a corpus of **154 real arXiv papers** with a reference PDF
  each, a vocabulary-bounded word census puts the median at **101% of the
  reference engine's prose words** (measured 2026-10-04); **10** of the 154 fall
  below 95% and 3 below 90%. Pagination lands on the reference exactly for
  **37** of them, with a total page deviation of 342 over the corpus.

  A median above 100% is expected rather than flattering: the engine emits each
  formula's *source* as an invisible text layer so that maths can be found,
  copied and read aloud, and the census counts it. The figure to watch is the
  tail — the papers below 95% — not the median.

## Scope — honest limits

The engine **loads and runs the standard base classes**; it is
functional-parity-oriented and held to real-LaTeX fidelity, **not** a claim of
full TeXLive parity.

Four classes are embedded as genuine `.cls` files and run as real TeX —
`article`, `report`, `book` and `amsart` — so they need no filesystem and work in
the `js/wasm` build too. Any other resolvable `.cls` or `.sty` is loaded and run
as real TeX as well; what is *not* resolvable falls back to
content-preserving emulation. The heavy packages are the honest gap: TikZ/pgf
**drawing** is gated behind `GOTEX_PGF` and still in bring-up — see the
[roadmap](packages/engine.md#roadmap).

## Packages

<div class="pk-grid" markdown>
<a class="pk-card" href="packages/engine.md"><code>engine</code><br><small>The pure-Go TeX engine — faithful mouth + gullet, scaled-point stomach, math, fonts, PDF/SVG, and the real LaTeX base classes.</small></a>
<a class="pk-card" href="packages/math.md"><code>math</code><br><small>TeX math-mode typesetter → vector output via the OpenType MATH table; wasm-ready for offline math preview.</small></a>
<a class="pk-card" href="packages/tex.md"><code>tex</code><br><small>A lightweight LaTeX-subset document processor → semantic HTML; wasm-ready.</small></a>
</div>
