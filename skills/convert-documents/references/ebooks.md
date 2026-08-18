# Ebook Conversion

## Choose the Path

Prefer Calibre's `ebook-convert` for device-oriented EPUB, MOBI, AZW3, and related conversions. Prefer Pandoc for a clean, semantic source such as Markdown, HTML, DOCX, or LaTeX going to EPUB. Do not treat fixed-layout PDF as a good reflowable source; extraction order, headers, columns, footnotes, and scanned pages usually require editorial work or OCR.

Never bypass DRM or access controls. Stop and report the boundary when the source cannot be lawfully read by the installed tool.

Inspect the source with `ebook-meta`, `unzip -t` for EPUB container integrity, and a format-specific parser when available. Record title, author, language, identifier, cover, table of contents, spine order, chapters, embedded fonts, and fixed-layout or reflowable behavior.

## Command Patterns

Use Calibre for a direct supported conversion:

```sh
ebook-convert "input.epub" "output.azw3"
```

Inspect `ebook-convert --help` and the input/output plugin options before adding heuristics. Avoid blanket typography, margin, line-unwrapping, or chapter-detection options; they can silently rewrite a well-formed book.

Use Pandoc for semantic EPUB generation:

```sh
pandoc "input.md" --standalone --toc --output "output.epub"
```

Provide explicit metadata, language, cover, CSS, and resource paths when they are part of the request. Ensure cover and font licenses permit embedding.

## Validate

Validate EPUB package integrity and parse its manifest, spine, navigation, and metadata. Open the output in an available ebook reader and test navigation, chapter boundaries, links, footnotes, images, tables, code blocks, cover, fonts, and light/dark themes. Check at narrow and wide viewport sizes for reflowable output. Compare chapter and image counts and explain changes introduced by the target format.

For legacy formats, test on the named target device or application when compatibility is the reason for conversion. Report when such a consumer is unavailable.
