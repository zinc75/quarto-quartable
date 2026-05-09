# quartable

A [Quarto](https://quarto.org/) extension that adds **column spans, row
spans, partial horizontal lines (clines), full midrules, vertical lines
and per-cell alignment overrides** to Markdown pipe tables. Targets
HTML, Reveal.js and PDF/LaTeX with a single source.

> **Scope.** `quartable` is **PDF/LaTeX-first**: that's the target it was
> designed for and where every feature is fully supported. HTML and
> Reveal.js are supported as best-effort secondary targets — most features
> render correctly there, with a few documented gaps (see the
> [Compatibility matrix](#compatibility-matrix)). The maintainer does not
> intend to grow the extension into a fully-featured table renderer for
> every output format; **contributions are welcome** for HTML/Reveal
> polish and additional features (see [CONTRIBUTING.md](CONTRIBUTING.md)).

## Live documentation

The [project page](https://zinc75.github.io/quarto-quartable/) is a
landing that links to self-documenting renderings of every feature
in all three output formats — each example shows its markdown source
above the rendered table:

- Landing — <https://zinc75.github.io/quarto-quartable/>
- HTML demo — <https://zinc75.github.io/quarto-quartable/test_quartable.html>
- PDF demo — <https://zinc75.github.io/quarto-quartable/test_quartable.pdf>
- Reveal.js slides — <https://zinc75.github.io/quarto-quartable/test_reveal_quartable.html>

## Installation

From the directory of your Quarto project:

```bash
quarto add zinc75/quarto-quartable
```

Then enable the filter in your document's YAML front matter:

```yaml
filters:
  - quartable
```

## Quick example

```markdown
| Group           | Item | Value 1  | Value 2  |
|-----------------|------|---------:|---------:|
| [Group A]{rs=3} | x1   | 0.71     | 2.1      |
| ^               | x2   | 0.68     | 2.0      |
| ^               | x3   | 0.74     | 2.3      |
| ===             |      |          |          |
| [Group B]{rs=2} | x4   | 0.89     | 3.4      |
| ^               | x5   | 0.87     | 3.3      |
```

`[Group A]{rs=3}` merges three rows; `^` marks continuation cells; `===`
draws a midrule between the two groups.

## Syntax reference

### Column span — `{cs=N}`

```markdown
| Item | [Measurements]{cs=3} |    |    |
|------|----------------------|----|----|
|      | T1                   | T2 | T3 |
| x1   | 0.71                 | 0.68 | 0.74 |
```

The cell with `cs=N` merges the next `N-1` placeholder cells. You **must**
write `N-1` empty cells immediately after it (pipe-table syntax requires
all cells on every row).

### Row span — `{rs=N}` + `^`

```markdown
| Group           | Item |
|-----------------|------|
| [Group A]{rs=3} | x1   |
| ^               | x2   |
| ^               | x3   |
```

The cell with `rs=N` merges with the next `N-1` rows. Continuation cells
must be a single `^` character. (Empty placeholder cells also work, but
`^` makes the intent explicit.)

### Midrule — `===`

```markdown
| Group | Value |
|-------|------:|
| ===   |       |
| A     | 12    |
| A     | 8     |
| ===   |       |
| B     | 15    |
```

A row whose first cell contains exactly `===` is replaced with a
full-width horizontal separator.

A `===` placed as the **first row** of the body (immediately after the
Markdown header separator `|---|`) is treated as an explicit head/body
separator. By design, no head/body separator is drawn automatically —
see [Design choices](#design-choices).

### Cline (partial horizontal line) — `===N-M`

```markdown
| A | B | C | D |
|---|---|---|---|
| 1 | 2 | 3 | 4 |
| ===2-3 |  |  |  |
| 5 | 6 | 7 | 8 |
```

`===N-M` draws a partial line over columns `N..M` only. A single column
`N` is shorthand for `N-N`. Multiple ranges and single columns can be
combined: `===1-2,4-5`, `===2-3,5`, `===1,2-3,4`. In LaTeX this emits
one `\cmidrule(lr){…}` per range (booktabs partial rule, lighter weight
with trimmed ends, matching the booktabs aesthetic).

### Vertical lines on colspans — `{cs=N .vl}`, `{cs=N .vr}`, `rspan=K`

```markdown
| Item | [Measurements]{cs=2 .vr} |   | Note |
|------|---------------------------|---|------|
| A    | 1                         | 2 | foo  |
| B    | 3                         | 4 | bar  |
```

`.vl` adds a vertical line on the left boundary of the colspan cell,
`.vr` adds one on the right. The line spans the full table by default;
add `rspan=K` to limit it to the declaring row plus `K-1` rows below:

```markdown
| Item | [Measurements]{cs=2 .vr rspan=2} |   | Note |
```

Vlines are honoured on any cell. On a regular (single-column) cell the
boundary is the cell's own left/right edge; on a colspan cell it is the
left/right edge of the merged span.

### Per-cell alignment override — `align=l|c|r`

```markdown
| Item        |   Value |
|-------------|--------:|
| `[total]{align=l}` | 1234.5 |
```

The `align=` attribute forces an alignment on a single cell, overriding
the column's Markdown alignment. Both short (`l`, `c`, `r`) and long
(`left`, `center`, `right`) values are accepted. Without `align=`,
regular cells inherit the column alignment as usual; colspan cells
default to centered.

## Compatibility matrix

| Feature       | HTML | Reveal.js | PDF / LaTeX |
|---------------|:----:|:---------:|:-----------:|
| `cs=N`        | ✅   | ✅        | ✅          |
| `rs=N` + `^`  | ✅   | ✅        | ✅          |
| `===`         | ✅   | ✅        | ✅          |
| `===N-M`      | ✅   | ✅        | ✅          |
| `align=l\|c\|r` | ✅ | ✅        | ✅          |
| `.vl` / `.vr` | ❌   | ❌        | ✅          |

Tables that use any quartable feature are rendered with a
booktabs-inspired style in HTML/Reveal (no per-row borders, only the
explicit rules added by the filter). Regular Quarto tables that don't
use any quartable feature keep their default Quarto styling untouched.

HTML/Reveal support for vlines is on the roadmap — contributions
welcome (see [CONTRIBUTING.md](CONTRIBUTING.md)).

## Design choices

- **No automatic head/body separator.** quartable does not draw a
  separator between the table header and the body. This follows
  booktabs' philosophy of leaving that decision to the author (the
  separator becomes ambiguous when the header contains column spans or
  row spans). The Pandoc-generated `\midrule\endhead` is removed from
  the LaTeX output for tables that use any quartable feature. To
  request a head/body separator explicitly, place a `===` row as the
  **first row** of the body (immediately after the `|---|` header
  separator). The filter detects the leading `===`, restores the
  natural booktabs `\midrule` in LaTeX, and tags the table with the
  `quartable-head-sep` class for HTML/Reveal so the bundled CSS draws
  the matching border on the last header row.
- **Vertical lines, despite booktabs.** The booktabs author actively
  discourages vertical rules. quartable nonetheless implements `.vl` /
  `.vr` (LaTeX only for now) because they are useful in specific cases
  — typically to demarcate a colspan group, or to draw a partial
  vertical separator alongside a `\cmidrule`. Keep them rare.

## Limitations

- **Vlines are LaTeX-only** in v0.1. HTML and Reveal silently ignore
  them.
- **Rowspan in the table header**: Pandoc does not honour `row_span`
  on cells inside `tbl.head`. When `quartable` detects a rowspan in
  the header, it moves the header rows into the first body so the
  rowspan can be rendered. The visual `<thead>` distinction is lost
  on those tables.
- **Full-height vlines + complex column specs**: when Pandoc generates
  a `p{...}` column spec (e.g. on tables with width hints), the
  full-height vline injection in the column spec is skipped and a
  warning is printed to stderr. Use `rspan=N` instead for those tables.
- **Vlines + booktabs rules**: vertical lines have small visible gaps
  where they cross `\toprule`, `\midrule` and `\bottomrule` (intrinsic
  to booktabs — its author actively discourages vertical rules). The
  effect is more pronounced on heavy `\toprule`/`\bottomrule`. If gap-
  free intersections are critical, override booktabs rules with
  `\hline` in your own preamble.
- **Partial cline + colspan**: in HTML, when a cline range partially
  overlaps a colspan cell, the CSS `border-bottom` is applied to the
  whole cell (HTML can't draw a border on a fragment of a cell). LaTeX
  draws the line correctly because `\cline` operates on column
  boundaries.

## Development

The repository ships with a test document that exercises every feature:

```bash
quarto preview test_quartable.qmd --to html
quarto preview test_quartable.qmd --to pdf
quarto preview test_reveal_quartable.qmd
```

The PDF target keeps the intermediate `.tex` (`keep-tex: true`) so you
can inspect the LaTeX directly when debugging.

## License

[MIT](LICENSE) — see `LICENSE` for the full text.
