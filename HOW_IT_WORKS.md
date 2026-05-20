# How It Works

DeclassRoom is a desktop reader for declassified document collections. You
point it at a folder of PDFs, it indexes them, and from then on you have a
local search engine and annotation layer for the whole corpus. Nothing about
your reading habits leaves the machine.

This document describes the intended end-user experience. No code has been
written yet, so treat every section as "what we want to ship".

## What it does

- Indexes a folder of PDFs into a local SQLite + FTS5 database so the full
  text is searchable in milliseconds.
- Runs OCR (Tesseract) on any scanned PDF that does not already have a text
  layer, so old declassified scans become searchable too.
- Opens documents in a reader pane backed by pdf.js, with highlight, note,
  and tag tools alongside.
- Lets you link a passage in one document to a page in another, so a citation
  in a report can resolve to the source document you also have on disk.
- Groups documents into collections (e.g. "Stargate", "MKULTRA Subprojects",
  "AARO 2024 FOIA release") and lets you filter search by collection or tag.
- Exports your annotations as BibTeX, CSL JSON, or markdown for use in notes
  apps or papers.
- Phones home to nothing. Importing a public dump may fetch a single ZIP if
  you ask, but the app itself has no telemetry, no analytics, no auto-update
  ping back to a server.

## Planned features

Library and ingestion:

- Pick a corpus root on first launch; everything is read relative to that.
- Drop new files into the corpus folder, watch them appear in the library as
  the ingest queue picks them up.
- Per-file progress: parsing, hashing, OCR if needed, indexing.
- Re-ingest a single document if something went wrong (corrupt page, wrong
  OCR language).

Reader:

- pdf.js viewer with smooth zoom, page jump, two-page spread, and outline.
- Selection-based highlights with a color picker.
- Sticky notes anchored to a page region.
- Per-document margin notes for higher-level thoughts.
- Tag picker and collection membership shown inline.

Search:

- Corpus-wide FTS5 search with BM25 ranking.
- Per-result snippet showing the matched passage with surrounding context.
- Filters: by collection, by tag, by source label, by has-text-layer vs OCR.
- Click a result to jump directly to that page in the reader.

Linking and citation:

- Select a passage, "link to" another document and page, attach a note.
- Reader shows back-references: "this page is cited by 3 other documents".
- Export selected annotations as BibTeX / CSL JSON / markdown.

Import wizards (planned later):

- CIA Reading Room ZIP wizard: unpack to corpus, set the source label, pick
  a collection name.
- National Archives release folder wizard: walk the release index, set
  per-document metadata where the release index provides it.

## How to use it (planned flow)

### First run

1. Launch DeclassRoom. A welcome screen asks you to pick a corpus root - a
   folder somewhere on disk that holds your PDFs.
2. Optionally, pick an OCR language (default English). Multiple are allowed.
3. The app scans the corpus root and queues every PDF for ingestion.

### Ingestion

- The library view shows each file with a status: queued, parsing, OCR,
  ready, or failed.
- A status bar at the bottom shows queue depth and current throughput.
- You can keep using the app (search, annotate) while ingestion runs. Newly
  ready documents appear in search results as soon as their FTS5 writes
  commit.

### Searching the corpus

- Type a query into the top search bar.
- Results stream in with snippets. Each row shows: document title, page
  number, matched snippet, and collection / tag chips.
- Filters on the left narrow by collection, tag, or source label.
- Hitting Enter on a result opens the reader at that page with the match
  highlighted.

### Reading and annotating

- The reader pane shows the PDF; a right-side panel shows existing
  annotations for the current page.
- Select text and click Highlight to mark it. Click again to add a note.
- "Doc notes" live at the document level, not anchored to a page.
- Tags and collection membership are editable from the document header.

### Linking documents

- In the reader, select a passage. Click "Link to..." and pick a target
  document and page.
- The link appears in the right-side panel for both source and target. The
  target document's reader shows "Referenced from N other documents" on
  pages that have inbound links.
- A graph view (later) shows the document-to-document link structure for a
  collection.

### Exporting citations

- Pick a document, a collection, or a set of selected annotations.
- Choose BibTeX, CSL JSON, or markdown.
- Output is written to `<app_data>/declassroom/exports/` and revealed in the
  OS file manager.

### Import wizards (later)

- Open the wizard, pick the dump type (CIA Reading Room ZIP, NARA release
  folder).
- The wizard previews what it will import, lets you set a collection name
  and source label, then unpacks into the corpus root or reads from where
  you already have it.
- If the wizard needs to fetch a release archive over the network, it asks
  before doing so. The app's outbound-network capability stays off otherwise.

## Where things live on disk

- `<corpus_root>/`: the PDFs themselves. User-owned, never written to by
  the app.
- `<app_data>/declassroom/declassroom.sqlite`: documents, pages, FTS5 index,
  annotations, tags, collections, citation links.
- `<app_data>/declassroom/ocr-cache/`: transient rasterizations during OCR.
- `<app_data>/declassroom/logs/`: shell + sidecar log files.
- `<app_data>/declassroom/settings.json`: theme, OCR language, reader
  defaults.
- `<app_data>/declassroom/exports/`: BibTeX, CSL JSON, markdown outputs.
