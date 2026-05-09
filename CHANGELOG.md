# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

(nothing yet)

## [0.1.0] — 2026-05-07

Initial public release.

### Added

#### Core table constructs

- `[text]{cs=N}` — column span on Markdown pipe tables (HTML, Reveal,
  PDF/LaTeX). Empty placeholder cells must follow in the source so the
  total physical-cell count remains constant per row.
- `[text]{rs=N}` + `^` continuation markers — row span with explicit
  continuation rows (HTML, Reveal, PDF/LaTeX). Continuation rows can
  use either `^` or empty cells at the spanned column position.
- `===` (alone in the first cell of a row) — full-width horizontal
  separator (booktabs `\midrule` in LaTeX; separate `<tbody>` blocks
  with a CSS border in HTML/Reveal).

#### Partial horizontal rules (clines)

- `===N-M` — partial cline spanning columns `N..M`.
- `===N` — single-column cline (shorthand for `===N-N`).
- `===1,3-5,7` — multiple ranges and singletons combined on the same
  row (each comma-separated entry produces its own cline).
- LaTeX emits one `\cmidrule(l|r|lr){N-M}` per range. The trim option
  (`(l)`, `(r)`, `(lr)`) is chosen automatically based on adjacency
  to other ranges on the same line, so isolated single-column ranges
  keep their full natural width while adjacent segments drop the trim
  on the touching side (booktabs aesthetic preserved without the
  double-trim that would shrink one-column ranges to nothing).

#### Vertical lines (LaTeX only)

- `[text]{cs=N .vl .vr}` — vertical line(s) on the boundary of a
  colspan cell, full-height by default.
- `[text]{cs=N .vl rspan=K}` — limit the vline to the declaring row
  plus `K-1` rows below.
- Vlines also work on regular (cs=1) cells: `.vl` / `.vr` then refer
  to the cell's own left / right edge.
- LaTeX implementation strategy:
  - Full-height vlines are injected into the `\begin{longtable}{...}`
    column spec.
  - Limited-rspan vlines are wrapped in `\multicolumn{1}{<align>|}{...}`
    via marker injection + post-processing.
  - `@{}` markers at the table edges are preserved when wrapping
    cells in column 1 or the last column.

#### Per-cell alignment override

- `[text]{align=l|c|r}` (or `align=left|center|right`) — force a
  specific alignment on a single cell, overriding the column's
  Markdown alignment (`|---|`, `|:---:|`, `|---:|`, etc.).
- Works on any cell. Colspan cells default to centered when no
  `align=` is given; regular cells inherit their column's alignment
  unless overridden.
- LaTeX implementation: cs=1 cells are wrapped in
  `\multicolumn{1}{<align>}{...}` via marker injection (Pandoc only
  honours `cell.alignment` natively for cs>1 cells).

#### Head/body separator handling

- The first `===` of the first body, when placed immediately after
  the Markdown header separator (`|---|`), is interpreted as an
  explicit request for a head/body separator: LaTeX preserves the
  natural booktabs `\midrule\endhead`, and HTML/Reveal apply a CSS
  border on the last `<thead>` row via the auto-injected
  `quartable-head-sep` class.
- Without that explicit `===`, `quartable` does **not** draw an
  automatic head/body separator. The Pandoc-generated
  `\midrule\endhead` is removed from the LaTeX output, and the
  bundled CSS does not add a border between `<thead>` and the first
  `<tbody>`. This matches booktabs' philosophy of leaving the
  separator decision to the author.

#### Header cells with rowspan

- When the table header contains a cell with `rs=N`, `quartable`
  detects this (Pandoc does not honour rowspans inside `tbl.head`)
  and moves the header rows into the first body so the rowspan can
  cross into body rows correctly. The visual `<thead>` distinction
  is lost on such tables — use a leading `===` if a head/body
  separator is still desired.

#### Styling and packaging

- Tables that use any `quartable` feature receive the auto-injected
  `quartable` CSS class so the bundled stylesheet can switch them to
  a clean booktabs-inspired look (no per-row Bootstrap borders, only
  the explicit rules from the filter). Regular Quarto tables that
  don't use any feature keep their default Quarto/Bootstrap look.
- LaTeX: `multirow` and `booktabs` packages are auto-loaded via
  `quarto.doc.use_latex_package`.
- HTML/Reveal: the bundled CSS is auto-injected via
  `quarto.doc.add_html_dependency`, so the extension works when
  loaded purely through `filters: - quartable/quartable` (no
  `format:` block required in the user's YAML).

### Documentation

- README with full syntax reference and a feature-by-format
  compatibility matrix.
- CONTRIBUTING.md with development setup, code style, testing
  checklist and a roadmap of good-first-PR features.
- Two self-documenting test documents (`test_quartable.qmd` for
  HTML/PDF, `test_reveal_quartable.qmd` for Reveal.js): each example
  shows the markdown source above its rendered output.

### Design choices

- **PDF/LaTeX-first.** The extension was designed primarily for
  publication-quality LaTeX output. HTML and Reveal.js are
  best-effort secondary targets — most features render correctly,
  with a few documented gaps (notably vlines, currently LaTeX-only).
- **No automatic head/body separator.** See above.
- **Vertical lines despite booktabs.** booktabs' author actively
  discourages vertical rules; `quartable` nonetheless implements them
  because they are useful in specific cases (demarcating a colspan
  group, partial vertical separator alongside a `\cmidrule`). The
  documentation flags the small visible gaps where booktabs
  horizontal rules cross vertical lines as a known limitation
  intrinsic to booktabs.

### Known limitations

- Vlines are LaTeX-only. HTML and Reveal silently ignore them.
- Full-height vlines are skipped (with a stderr warning) on tables
  whose Pandoc-generated column spec uses `p{...}` width hints
  (typically tables with very wide rows). Use `rspan=N` instead
  for such tables.
- Partial cline + colspan in HTML: when a cline range partially
  overlaps a colspan cell, the CSS `border-bottom` is applied to
  the whole cell (HTML can't draw a border on a fragment of a cell).
  LaTeX renders this case correctly because `\cline` operates on
  column boundaries.
- Rowspan crossing a midrule (`\multirow` over `\midrule`) is
  flagged with a stderr warning and rendered best-effort: HTML
  remains correct, LaTeX visual is imperfect.
