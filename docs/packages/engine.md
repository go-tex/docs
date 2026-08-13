# engine

The pure-Go (`CGO_ENABLED=0`) TeX engine. A faithful re-implementation of TeX
that **loads and runs the genuine LaTeX classes** — `\documentclass{article}`,
`{report}` and `{book}` execute the real, embedded `.cls` files (LPPL, verbatim)
— not an emulation — in native builds and in the browser via `js/wasm`, with no
TeXLive and no server.

## What it does

- **Runs the standard base classes for real.** `article`, `report` and `book`
  load and execute the genuine embedded LaTeX `.cls` files, typesetting a
  numbered title, a dotted `\tableofcontents`, numbered sections/subsections,
  `\chapter` (report/book), numbered figure/table captions, `itemize`/`enumerate`,
  and math.
- **A full pipeline.** A category-code mouth and gullet, a scaled-point stomach
  with Knuth–Plass line breaking and a cost-based page builder, `\halign` tables,
  math via [`math`](math.md) (vector output), OpenType fonts (a built-in default
  font, kerning, ligatures), and **PDF + SVG** output — the SVG carries a source
  map for click-to-line.
- **In the browser.** A self-contained `js/wasm` build renders real LaTeX classes
  client-side — no TeXLive, no server.

## Pipeline stages

Each stage is gated by an objective oracle.

| Stage | Status | What it covers |
|---|---|---|
| Mouth + gullet | ✅ done | Category-code tokenizer, `eqtb`, macros (delimited parameters, full expansion), conditionals, registers, grouping. |
| Stomach | ✅ done | Box/glue/penalty model in scaled points, Knuth–Plass line breaking, a cost-based page builder, `\halign` tables. |
| Math | ✅ done | `$…$` and display environments delegated to [`math`](math.md) (vector output). |
| Fonts | ✅ done | OpenType via go-opentype; a built-in default font, kerning and ligatures — runs with no assets. |
| Output | ✅ done | PDF (embedded subset fonts, selectable text) and self-contained SVG pages with a click-to-line source map. |
| Real classes | ✅ done | `\documentclass{article\|report\|book}` loads and runs the genuine embedded LaTeX class, native and `js/wasm`. |

## Fidelity gates

- a byte-exact **conformance ratchet** (`TestConformance`), TeX snippets checked
  byte-for-byte against real-TeX output; and
- a **whole-document prose fidelity check** against a real LaTeX engine
  (`tectonic`). On a sample of real arXiv `article` papers, the real class
  reproduces about **90% (median)** of the reference engine's prose words.

## Roadmap

The engine loads and runs the standard base classes; it is
functional-parity-oriented and held to real-LaTeX fidelity, **not** a claim of
full TeXLive parity.

- `amsart` run as a real class (it additionally needs `amsmath`).
- Heavy packages (`tikz`, `hyperref`, …) run as real files rather than native
  stubs.
- A broader real-document conformance corpus (PDF-diff against pdftex/xetex) and
  the TRIP test.

## Install

```bash
go get github.com/go-tex/engine
```

Requires Go 1.26.4 or newer. `CGO_ENABLED=0`, `go vet` clean, CI green across
three 64-bit arches under QEMU (`riscv64`/`ppc64le`/`s390x`) plus macOS, Linux
and Windows, and both `js/wasm` and `wasip1/wasm`.

## Links

- Source: <https://github.com/go-tex/engine>
- API reference: <https://pkg.go.dev/github.com/go-tex/engine>

!!! note
    See the module's README for full, up-to-date details.
