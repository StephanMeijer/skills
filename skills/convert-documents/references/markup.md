# Markup and Semantic Document Conversion

## Choose the Rendering Model

Prefer Pandoc for conversions where headings, paragraphs, lists, tables, citations, notes, and links are the primary contract. Prefer a browser print engine for HTML whose CSS layout and web fonts define the desired appearance. Prefer LibreOffice for an office document whose native layout is authoritative.

Inspect Pandoc's installed readers, writers, and version:

```sh
pandoc --version
pandoc --list-input-formats
pandoc --list-output-formats
```

Do not assume PDF output works merely because Pandoc is installed. Probe the requested PDF engine and required fonts separately.

## Pandoc Patterns

Perform a direct semantic conversion:

```sh
pandoc "input.md" --output "output.docx"
pandoc "input.docx" --output "output.md"
pandoc "input.md" --standalone --output "output.html"
```

Use a reference document when Word styles are part of the contract:

```sh
pandoc "input.md" --reference-doc="reference.docx" --output "output.docx"
```

Resolve linked assets from an explicit location:

```sh
pandoc "input.md" --resource-path="source-directory" --standalone --output "output.html"
```

For citations, filters, templates, math, or raw HTML/LaTeX, inventory every required sidecar and extension first. Avoid enabling arbitrary filters or executing embedded code from untrusted documents.

## PDF Rendering

Use an explicitly available PDF engine:

```sh
pandoc "input.md" --pdf-engine=xelatex --output "output.pdf"
```

Select the engine from the content and installed dependencies; XeLaTeX is only an example. Capture missing-glyph, overfull-box, and resource warnings.

For CSS-driven HTML, prefer an installed Chromium-family browser and print the fully resolved page:

```sh
chromium --headless --print-to-pdf="output.pdf" "file:///absolute/path/input.html"
```

Do not add sandbox-disabling flags as a convenience. Wait for required local resources and web fonts, and do not fetch remote resources without network authorization. Use `wkhtmltopdf` only when its older rendering engine is compatible with the page.

## Validate

Parse the output with another consumer. Compare headings, links, images, tables, footnotes, citations, math, language, and metadata. For HTML, load the result with networking disabled and check for missing local resources and console errors. For PDF or office output, render representative pages and check styles, page breaks, overflow, glyphs, and image resolution. Semantic conversions may reflow; distinguish intended reflow from missing content.
