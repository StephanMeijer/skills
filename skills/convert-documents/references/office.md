# Office Document Conversion

## Choose LibreOffice Deliberately

Prefer `soffice` or `libreoffice` headless conversion for word-processing documents, spreadsheets, and presentations when rendered fidelity matters. Confirm the installed version and available export filters. Expect fonts, macros, proprietary effects, external data sources, tracked changes, and complex forms to vary across office suites.

Use an isolated user profile so a running GUI instance, stale lock, extension, or user preference cannot capture the request:

```sh
workdir=$(mktemp -d)
soffice -env:UserInstallation="file://$workdir/profile" --headless --convert-to pdf --outdir "$workdir/output" "input.docx"
```

Create the output directory before running the command. LibreOffice derives the output basename from the input; move the single expected result to its final path only after checking it. Capture standard output and error because some conversion failures still leave confusing artifacts.

For an editable format conversion, name the desired extension and specify an export filter when ambiguity matters:

```sh
soffice -env:UserInstallation="file://$workdir/profile" --headless --convert-to odt --outdir "$workdir/output" "input.docx"
```

Consult the installed LibreOffice filter list rather than guessing a filter name.

## Word-Processing Documents

Check sections, page geometry, headers and footers, footnotes, fields, comments, tracked changes, tables, lists, images, equations, hyperlinks, embedded objects, and font substitution. Decide whether tracked changes should remain editable, be accepted, or appear in the rendered output before conversion.

Pandoc can be a better route when semantic structure matters more than exact Word or Writer rendering. Read [markup.md](markup.md) when choosing that path.

## Spreadsheets

Treat CSV as a single flat table, not a workbook format. It cannot retain multiple sheets, formulas, formatting, charts, types, comments, merged cells, or macros. Require a delimiter, encoding, decimal convention, and sheet-selection policy when exporting CSV.

For spreadsheet-to-PDF conversion, inspect print areas, repeated headers, page orientation, scaling, manual page breaks, hidden rows and sheets, charts, and formula results. Headless recalculation may differ when external links, volatile functions, locale, or unavailable fonts are involved.

## Presentations

For presentation-to-PDF conversion, inspect slide size, master layouts, fonts, transparency, diagrams, notes, hidden slides, videos, transitions, and animations. PDF is static; agree on whether notes or handouts are required and acknowledge that animation and interactive media will not survive.

## Validate

Open editable output in a compatible office consumer and export it to PDF for a rendered comparison when practical. For PDF output, inspect page count and render representative pages. For spreadsheets, verify sheet count, formulas, displayed values, types, and print layout. For presentations, verify slide count, masters, element placement, and missing media warnings. Compare document properties and accessibility information when the target supports them.
