# PDF Conversion and Transformation

## Classify the Request

Determine whether the PDF contains selectable text, scanned images, forms, annotations, attachments, signatures, layers, or encryption. Distinguish format conversion from PDF transformation:

- use Poppler or MuPDF to render pages or extract text and images;
- use OCRmyPDF to add a searchable text layer to authorized scanned documents;
- use QPDF for structural transformations, inspection, linearization, and some repair operations;
- use Ghostscript for PostScript/PDF conversion or deliberate PDF rewriting and compression;
- use LibreOffice or Pandoc only when their source-side model matches the actual input.

Do not promise faithful PDF-to-DOCX or PDF-to-reflowable-ebook conversion. PDF stores positioned page output, not the original semantic document. Treat reconstructed reading order, tables, headers, columns, and styles as content recovery requiring review.

## Inspect

Use available independent tools:

```sh
pdfinfo "input.pdf"
qpdf --check "input.pdf"
pdffonts "input.pdf"
pdftotext "input.pdf" -
```

Do not print sensitive extracted text into logs. Identify encryption, permissions, signatures, page count and size, PDF version, tagging, forms, fonts, and attachments before selecting a path. Never bypass a password or permissions boundary.

## Command Patterns

Render pages with Poppler at an explicit resolution:

```sh
pdftoppm -png -r 150 "input.pdf" "page"
```

Extract text when extraction, rather than layout preservation, is the goal:

```sh
pdftotext -layout "input.pdf" "output.txt"
```

Use QPDF for lossless structural rewriting or web linearization:

```sh
qpdf --linearize "input.pdf" "output.pdf"
```

Use Ghostscript presets only when the user explicitly accepts their quality and resolution tradeoffs:

```sh
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.7 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile="output.pdf" "input.pdf"
```

The `/ebook` preset is an example, not a safe default; it can downsample or recompress content. For OCR, inspect `ocrmypdf --help`, select languages explicitly, and preserve the original alongside the OCR output.

## Validate

Run a structural check with QPDF or another parser, compare page count, page boxes, metadata, attachments, annotations, forms, outlines, links, fonts, text extractability, and file size, and explain intentional changes. Render every page for a short document or representative first, middle, and last pages for a long one. Inspect small text, vector art, gradients, transparency, images, and color. Verify OCR text order and recognition on difficult pages.

Treat any digital signature as invalidated by content-changing conversion or rewriting unless a signature-aware verifier proves otherwise. Report the signature status explicitly.
