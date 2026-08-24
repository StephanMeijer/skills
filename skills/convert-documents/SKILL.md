---
name: convert-documents
description: Convert documents between office, PDF, markup, ebook, spreadsheet, and presentation formats with local command-line tools while preserving the required layout, structure, fonts, metadata, links, and accessibility. Use for DOCX, ODT, RTF, PDF, HTML, Markdown, LaTeX, EPUB, MOBI, CSV, XLSX, ODS, PPTX, ODP, and related document conversions or batch exports. Do not use for image, audio, or video conversion except when rendering document pages for validation.
license: MIT
---

# Convert Documents

Convert documents with an installed CLI and validate both their structure and rendered result. Choose the conversion path according to whether the user values editable semantics, visual fidelity, archival behavior, or a compact reading format.

## Establish the Conversion Contract

1. Identify every input, the exact output format, destination, naming rule, and intended reader or application.
2. Determine whether layout fidelity or editable structure has priority. Surface unavoidable losses before conversion, including formulas, macros, comments, tracked changes, forms, animations, speaker notes, page geometry, and ebook reflow.
3. Inspect the real input type rather than trusting its extension. Record page or sheet count, document properties, fonts, links, media, and encryption when relevant.
4. Keep the input unchanged. If the requested output path exists, do not overwrite it without explicit permission; select a distinct path or ask.
5. Treat password protection, DRM, signatures, and macros as explicit boundaries. Never bypass access controls, and never claim a converted signature remains valid.

## Discover Capabilities

Probe commands with `command -v`; never assume a converter is installed. Select a backend by behavior:

- LibreOffice (`soffice` or `libreoffice`) for office documents, spreadsheets, and presentations where rendered fidelity matters;
- Pandoc for semantic conversions among Markdown, HTML, LaTeX, DOCX, ODT, EPUB, and similar text-centric formats;
- Chromium or another browser print engine for CSS-driven HTML to PDF;
- Calibre's `ebook-convert` for ebook formats and device-oriented ebook output;
- Poppler, MuPDF, QPDF, Ghostscript, or OCRmyPDF for specific PDF extraction, rendering, repair, optimization, or OCR operations.

Inspect the selected tool's version and help before using format-specific features. Do not install packages, fetch office suites, or download rendering engines unless the user authorizes changing the environment. If no suitable converter exists, report the missing capability without fabricating an output.

## Route the Task

- Read [references/office.md](references/office.md) for DOCX, ODT, RTF, XLSX, ODS, CSV, PPTX, ODP, and office-to-PDF work.
- Read [references/markup.md](references/markup.md) for Markdown, HTML, LaTeX, plain text, and semantic document conversions.
- Read [references/ebooks.md](references/ebooks.md) for EPUB, MOBI, AZW3, and other reflowable reading formats.
- Read [references/pdf.md](references/pdf.md) for PDF input, PDF rendering, PostScript, OCR, linearization, and PDF optimization.

Read multiple references when the safest path has an intermediate format. Avoid unnecessary conversion chains because each boundary can lose information.

## Convert Safely

1. Copy inputs to a temporary workspace when the converter may create sidecar files or chooses its own output basename.
2. Use a temporary output directory on the destination filesystem. Promote only the expected artifact after validation, and reject ambiguous or extra outputs.
3. Isolate stateful office-suite profiles so an existing GUI session, lock file, extension, or user setting cannot affect a headless conversion.
4. Resolve relative images, stylesheets, fonts, bibliography files, and linked resources from an explicit base directory.
5. For batches, prove one representative conversion before processing the set. Preserve a deterministic one-to-one mapping, or document one-to-many outputs such as PDF pages or spreadsheet sheets.

## Verify Structure and Appearance

Require both structural and visual checks when the output has a rendered layout:

1. Confirm the converter exits successfully, produces exactly the expected non-empty file, and reports no truncated-resource or font-substitution errors.
2. Identify the output independently and open it with a second parser or consumer when available.
3. Compare page, slide, sheet, chapter, word, image, link, and metadata counts as applicable. Explain expected differences caused by reflow or format limits.
4. Render the output and inspect representative first, middle, and last pages or slides; inspect every page for a short document. Check clipping, overflow, missing glyphs, image placement, tables, formulas, headers, footnotes, and page breaks.
5. Exercise editable or interactive behavior when required: formulas recalculate, links resolve, navigation works, forms remain usable, and text remains searchable or selectable.

If a renderer or target application is unavailable, mark visual or application-level validation unavailable rather than inferring success from the exit code. Report the command, output path, backend and version, before-and-after properties, known losses, font substitutions, and every validation performed.
