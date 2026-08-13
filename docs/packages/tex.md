# tex

A lightweight, pure-Go (`CGO_ENABLED=0`) processor for a practical subset of
LaTeX documents. It tokenizes with TeX category codes, expands user macros
(`\newcommand`/`\renewcommand` with mandatory and optional arguments, `\def`,
`\let`), parses document structure, and emits **semantic HTML** with math typeset
by [`math`](math.md). No engine binary, no cgo, no server; it compiles to
`GOOS=js/wasm`.

For full TeX — real line breaking, page building, the LaTeX classes, and PDF/SVG
output — see [`engine`](engine.md); `tex` is a lightweight document-to-HTML path.

## Install

```bash
go get github.com/go-tex/tex
```

Requires Go 1.26.4 or newer. `CGO_ENABLED=0`.

## Links

- Source: <https://github.com/go-tex/tex>
- API reference: <https://pkg.go.dev/github.com/go-tex/tex>

!!! note
    See the module's README for full, up-to-date details.
