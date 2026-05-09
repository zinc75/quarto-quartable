# Contributing to quartable

Thanks for considering a contribution. This document explains the
project philosophy, the development workflow, and lists features that
would make good first PRs.

## Project philosophy

`quartable` was built to scratch a specific itch: producing publication-
quality LaTeX/PDF tables from Markdown pipe tables in Quarto, without
having to drop down to raw LaTeX. HTML and Reveal.js are supported as
secondary targets.

The maintainer's intent is to keep the **core minimal and stable**,
not to grow the extension into a fully-featured cross-format table
engine. If you want a feature that isn't here:

- **Open an issue first** to discuss whether it fits the scope.
- For features in the [Roadmap](#roadmap--good-first-prs) below,
  go ahead and open a PR — they're already endorsed.
- For something more ambitious, propose an issue and we'll talk
  before you invest time.

## Development setup

```bash
git clone https://github.com/zinc75/quarto-quartable.git
cd quarto-quartable
```

The extension is a single Lua filter plus a CSS file:

```
_extensions/quartable/
  _extension.yml
  quartable.lua    ← the filter
  quartable.css    ← styling for HTML/Reveal
  latex-header.tex ← multirow + booktabs imports
```

To test changes, render the bundled test documents:

```bash
quarto preview test_quartable.qmd --to pdf     # PDF/LaTeX
quarto preview test_quartable.qmd --to html    # HTML
quarto preview test_reveal_quartable.qmd       # Reveal.js
```

`test_quartable.qmd` exercises every feature. The PDF target keeps the
intermediate `.tex` (`keep-tex: true`) so you can inspect the LaTeX
output directly when debugging.

## Code style

- **Lua**: 2-space indent, snake_case for locals, descriptive names. All
  comments and stderr messages in **English**.
- **CSS**: scope new rules under `table.quartable` so they only affect
  tables that use quartable features (regular Quarto tables must remain
  untouched). Use `currentColor` for borders to inherit from the theme.
- **Lua filter architecture**: keep the same pipeline shape (detect →
  transform AST → render to LaTeX via `pandoc.write` → post-process the
  rendered string). Don't add a new format-specific code path without
  discussing it.

## Testing checklist before opening a PR

1. `quarto preview test_quartable.qmd --to pdf` produces a clean PDF
   with no LaTeX errors and no unexpected stderr warnings.
2. The intermediate `test_quartable.tex` looks reasonable for the
   feature you touched (inspect it manually).
3. `quarto preview test_quartable.qmd --to html` renders without
   visual regressions on the existing examples.
4. `quarto preview test_reveal_quartable.qmd` renders correctly.
5. If you added a new feature, add a corresponding example to
   `test_quartable.qmd` (and `test_reveal_quartable.qmd` if relevant).
6. Update the README's syntax reference and compatibility matrix.
7. Add a line to `CHANGELOG.md` under an `## [Unreleased]` section.

## PR guidelines

- **One feature per PR.** Refactors and feature additions in the same PR
  are hard to review.
- **Keep diffs small** and focused. Don't reformat unrelated code.
- **No new dependencies.** The extension must remain a single Lua filter
  plus the bundled CSS / TeX header.
- **Document edge cases** in the README under "Limitations" if your
  feature has any.
- **Generic test examples.** Don't introduce domain-specific examples
  (acoustics, finance, etc.); use neutral content (groups, items,
  values) so the test document stays universally readable.

## Roadmap / good first PRs

These features would be welcomed and are within scope:

### High value

- **HTML/Reveal vlines.** Implement `.vl` / `.vr` in HTML by adding CSS
  classes (`quartable-vl`, `quartable-vr`) on the cells targeted by
  `collect_vlines` and adding `border-left` / `border-right` rules.
  Most of the infrastructure exists — see `collect_vlines()` in
  `quartable.lua`. Estimated effort: ~1–2 hours.
- **Honour the column alignment in vline-only `\multicolumn{1}{l|}{...}`
  wraps.** When a cs=1 cell carries a vline (`.vl`/`.vr`) but no
  explicit `align=`, the LaTeX wrap currently forces `l` regardless of
  the column's natural alignment. The infrastructure to read
  `tbl.colspecs[c].alignment` is straightforward; threading that
  through `inject_vline_markers` and `expand_vline_markers` (Case B)
  would let vline-only cells inherit the column alignment correctly.

### Medium value

- **Declarative vline syntax** (e.g. `@@N:F-T` row marker), as a sugar
  layer on top of the per-cell API. The original design notes propose
  the syntax `@@col:from-to` to declare vlines without per-cell
  attributes. Would parse and expand into per-cell `.vl`/`.vr`
  attributes before the LaTeX/HTML pipeline.
- **Range validation warnings.** Warn at parse time if `===N-M` has
  `N > n_cols` or `M < N`, or if `rspan=K` extends past the body.
- **Word/docx output.** Pandoc has limited table support for docx but
  some features (colspan, rowspan, midrule via separate sub-tables)
  could be made to work.

### Low value (probably out of scope)

- Support for table styles beyond booktabs in the bundled CSS.
- Programmatic/scripted table generation. `quartable` is intentionally
  a Markdown-syntax extension, not a table generator.

## Reporting bugs

Open a GitHub issue with:

1. The minimal Markdown table that triggers the bug.
2. The expected vs. actual output (HTML / LaTeX / PDF screenshot).
3. The intermediate `.tex` file if the issue is on the LaTeX side
   (rendering with `keep-tex: true` produces it).
4. Quarto version (`quarto --version`) and OS.

## License

By contributing, you agree that your contributions will be licensed
under the [MIT License](LICENSE) of the project.
