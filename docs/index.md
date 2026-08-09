# go-tex

Pure-Go TeX.

go-tex is a pure-Go (CGO_ENABLED=0) TeX effort. It starts from the math component — a TeX math-mode typesetter that lays formulas out to self-contained SVG using the OpenType MATH table (metrics + vector glyph outlines via go-opentype), with no TeX engine, no cgo and no server, and compiles to js/wasm for offline client-side math preview. The full engine will follow in go-tex/tex.

Everything is **pure Go** (`CGO_ENABLED=0`), standard-library-first, and
cross-compiles to every 64-bit Go target. Licensed BSD-3-Clause.

## Packages

<div class="pk-grid" markdown>
<a class="pk-card" href="packages/math.md"><code>math</code><br><small>TeX math-mode typesetter → SVG via the OpenType MATH table; wasm-ready for offline math preview.</small></a>
</div>
