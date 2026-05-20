# Roadmap

This document is forward-looking. DeclassRoom has no shipped code yet; the
phases below describe what the project intends to deliver, in order. Each
phase ends with something usable on its own.

## Phase 1 - PDF import + text-layer search

The minimum useful product: point at a folder of PDFs that already have a
text layer, get a searchable library.

- Pick a corpus root on first launch.
- Walk the corpus root, queue every PDF for ingestion.
- Per-file: open with pdf.js / pdfium, pull each page's text layer, hash the
  file, write a `documents` row, write per-page rows into `pages`, let the
  FTS5 triggers populate `pages_fts`.
- Library view: list documents, show ingestion status, jump to reader.
- Reader: pdf.js-backed, page jump, zoom, outline.
- Search: corpus-wide FTS5 with snippet previews, click-through to reader.

Exit criteria: a folder of ~1000 typed PDFs (e.g. a CIA Reading Room export)
indexes in a reasonable time and is searchable end-to-end.

## Phase 2 - OCR for scanned docs

Most of the interesting older releases are scans. Tesseract gets us text.

- Detect during ingestion when a PDF has no meaningful text layer.
- Rasterize pages at a configurable DPI; run Tesseract; store the output as
  if it were a normal text-layer extraction.
- Mark `has_text_layer = 0` on the document so the UI can show a badge
  ("OCR'd").
- Surface OCR failures (timeout, language mismatch) in the library as a
  per-document warning, not a silent drop.
- Settings: pick OCR languages, change the rasterization DPI, re-run OCR on
  a single document.

Exit criteria: a folder of scanned, pre-OCR declassified PDFs becomes
searchable with acceptable accuracy on clean scans.

## Phase 3 - Annotations + tags + collections

Now that the corpus is searchable, make it a study tool.

- Annotation model in SQLite: highlights, page notes, doc notes.
- Reader overlay: select text, highlight, note, edit, delete.
- Tags: free-form, per-document, with autocomplete from existing tags.
- Collections: named groupings (e.g. "Stargate", "AARO 2024 release"),
  documents can belong to multiple.
- Library filters by tag and collection; search filters likewise.

Exit criteria: a user can sit down with a release like the JFK records,
group documents into collections, tag them by subject, and leave per-page
notes that survive a restart.

## Phase 4 - Cross-document linking + citation export

The thing that distinguishes a study tool from a viewer: links between
documents and citation export.

- Cross-doc link model: source doc + source page + selection rectangle ->
  destination doc + destination page, with an optional note.
- Reader shows inbound links per page ("referenced from N other documents").
- A simple graph view of links within a collection.
- Export: pull annotations and links out as BibTeX, CSL JSON, or markdown,
  written to `<app_data>/declassroom/exports/`.

Exit criteria: a user can build a chain of "this report cites this earlier
report cites this primary source", click through it in the reader, and
export the chain as a citation block.

## Phase 5 - Import wizards for public dumps

Make the on-ramp painless for the common public collections.

- CIA Reading Room ZIP wizard: unpack into the corpus root, set the source
  label, optionally pre-create a collection per CREST category.
- National Archives release wizard: read the release index, assign titles
  and source metadata from the index instead of from PDF metadata.
- FOIA folder wizard: agency + release date + a label, applied to every PDF
  in the folder.

Network policy: each wizard that wants to fetch over the network asks
explicitly. The default app-wide outbound-network capability stays off.

Exit criteria: a user with a fresh install can pick "Import CIA Reading
Room ZIP", point at the file, and have a tagged, labeled, searchable
collection a few minutes later.

## Things explicitly not planned

- Sync across devices. The corpus is whatever you have on disk.
- A cloud library of pre-indexed documents.
- A login / account system.
- An ML summarizer that paraphrases declassified material. The tool is for
  reading the source documents, not for talking about them.
- A publishing path back to the internet.

## Known open questions

- Where to draw the line between "OCR good enough" and "manual transcription
  needed". For very poor scans (faded, skewed, handwritten margin notes),
  Tesseract output is sometimes worse than useless.
- Whether to ship a default tag taxonomy (suggested tags) per common dump,
  or leave that entirely to the user.
- How to handle very large single PDFs (some FOIA releases are one 5000-page
  file). Page-at-a-time ingestion is the plan, but the reader UX for
  documents that long is its own problem.
- Whether the graph view for cross-document links is worth the complexity
  in Phase 4, or whether a flat back-reference list is enough.
