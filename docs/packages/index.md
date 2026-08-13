# Packages

The modules in the org. Each is an independent Go module, `CGO_ENABLED=0`.

| Package | What it does | API |
|---|---|---|
| [`engine`](engine.md) | The pure-Go TeX engine — faithful mouth + gullet, scaled-point stomach, math, OpenType fonts, PDF/SVG, and the real LaTeX base classes (`article`/`report`/`book`), native and `js/wasm`. | [pkg.go.dev](https://pkg.go.dev/github.com/go-tex/engine) |
| [`math`](math.md) | TeX math-mode typesetter → vector output via the OpenType MATH table; wasm-ready for offline math preview. | [pkg.go.dev](https://pkg.go.dev/github.com/go-tex/math) |
| [`tex`](tex.md) | A lightweight LaTeX-subset document processor → semantic HTML (macro expansion + structure + math); wasm-ready. | [pkg.go.dev](https://pkg.go.dev/github.com/go-tex/tex) |
