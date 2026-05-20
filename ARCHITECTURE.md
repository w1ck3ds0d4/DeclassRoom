# Architecture

DeclassRoom is planned as a Tauri 2 desktop app: a React + Vite frontend for
the reader and library UI, a Rust shell that owns the window and the file
system permissions, and a Node / Fastify sidecar that handles the heavier
ingestion work (PDF parsing, OCR, FTS5 writes). Everything lives on disk under
the platform `app_data` directory plus a user-chosen corpus root. There is no
remote service.

This document is the design sketch. No code has been written yet; the section
headings describe what we want the shape to be.

## Tech stack (planned)

- Shell: Rust (edition 2021), Tauri 2
- Sidecar: Node 22, Fastify, run as a Tauri sidecar process
- Storage: SQLite via `better-sqlite3`, with the FTS5 extension enabled
- PDF rendering: `pdf.js` in the UI for the reader, with `pdfium-render` (Rust)
  or `pdfjs` headless as a fallback for text extraction on the sidecar side
- OCR: Tesseract, invoked from the sidecar for any PDF that lacks a text layer
- Frontend: React 18, TypeScript, Vite 5, `lucide-react` icons
- Build: pnpm workspace, `tauri-cli` 2

## Component breakdown (planned)

Rust shell (`src-tauri/src/`):

- `lib.rs` / `main.rs`: entry point, window setup, sidecar lifecycle (spawn on
  boot, kill on exit), Tauri command surface that the UI calls.
- `corpus.rs`: resolves the corpus root, validates that the directory is
  readable, watches for new files (via `notify`) so dropping a PDF into the
  corpus folder gets noticed without a manual rescan.
- `permissions.rs`: file-system allowlist enforcement. Reads are scoped to the
  corpus root and the app data dir; writes go to app data only. No outbound
  network.
- `sidecar.rs`: spawns the Fastify process on a loopback-only port, restarts
  it on crash, surfaces its logs to the in-app debug view.

Sidecar (`sidecar/src/`):

- `server.ts`: Fastify boot, route registration, graceful shutdown on SIGTERM.
- `ingest/queue.ts`: per-file ingestion queue with progress events streamed
  back to the UI.
- `ingest/pdf.ts`: opens a PDF, extracts text per page where a text layer is
  present, hashes the file (SHA-256) for provenance.
- `ingest/ocr.ts`: when no text layer is detected, rasterizes pages and runs
  Tesseract. Output goes into the same per-page text store.
- `index/fts.ts`: writes the per-page text into the FTS5 virtual table and
  keeps the metadata tables in sync.
- `search/query.ts`: turns user queries into FTS5 MATCH expressions, returns
  ranked results with snippets and page coordinates.
- `annotate/store.ts`: CRUD for highlights, notes, tags, collections, and
  cross-document links.
- `export/citations.ts`: pulls highlights and notes out as BibTeX, CSL JSON,
  or markdown.

Frontend (`src/`):

- `App.tsx`: shell, sidebar, route switcher.
- `components/Library.tsx`: corpus browser with collection / tag filters.
- `components/Reader.tsx`: pdf.js-backed reader pane with overlay layers for
  highlights, notes, and cross-doc links.
- `components/SearchBar.tsx`, `SearchResults.tsx`: corpus-wide search UI.
- `components/Ingest.tsx`: per-file progress for the import queue.
- `components/Collections.tsx`, `Tags.tsx`: collection and tag management.
- `components/LinkPanel.tsx`: the cross-doc link editor and back-reference view.

## SQLite schema sketch

The intent is a small set of tables plus one FTS5 virtual table for search.

```
documents (
  id            INTEGER PRIMARY KEY,
  path          TEXT    NOT NULL UNIQUE,     -- relative to corpus root
  sha256        TEXT    NOT NULL,            -- file integrity / provenance
  title         TEXT,                        -- pulled from PDF metadata, editable
  source        TEXT,                        -- e.g. 'CIA Reading Room', 'NARA JFK 2025'
  page_count    INTEGER NOT NULL,
  has_text_layer INTEGER NOT NULL,           -- 0 = OCR was used
  ingested_at   INTEGER NOT NULL
)

pages (
  id            INTEGER PRIMARY KEY,
  document_id   INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  page_number   INTEGER NOT NULL,            -- 1-based
  text          TEXT,                        -- extracted or OCR'd
  UNIQUE (document_id, page_number)
)

pages_fts (                                  -- FTS5 virtual table
  text,
  content='pages',
  content_rowid='id'
)

annotations (
  id            INTEGER PRIMARY KEY,
  document_id   INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  page_number   INTEGER NOT NULL,
  kind          TEXT    NOT NULL,            -- 'highlight' | 'note' | 'doc_note'
  rect_json     TEXT,                        -- pdf.js viewport rect for overlays
  text          TEXT,                        -- the highlighted excerpt
  body          TEXT,                        -- user-authored note
  created_at    INTEGER NOT NULL,
  updated_at    INTEGER NOT NULL
)

tags (
  id            INTEGER PRIMARY KEY,
  name          TEXT    NOT NULL UNIQUE
)

document_tags (
  document_id   INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  tag_id        INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (document_id, tag_id)
)

collections (
  id            INTEGER PRIMARY KEY,
  name          TEXT    NOT NULL UNIQUE,
  description   TEXT
)

collection_documents (
  collection_id INTEGER NOT NULL REFERENCES collections(id) ON DELETE CASCADE,
  document_id   INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  PRIMARY KEY (collection_id, document_id)
)

citations (                                  -- cross-document links
  id            INTEGER PRIMARY KEY,
  src_document_id  INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  src_page         INTEGER NOT NULL,
  src_rect_json    TEXT,                     -- selection rectangle in source doc
  dst_document_id  INTEGER NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  dst_page         INTEGER NOT NULL,
  note          TEXT,                        -- why these two are linked
  created_at    INTEGER NOT NULL
)
```

FTS5 triggers keep `pages_fts` in sync with `pages` on insert / update / delete.
Search ranks by BM25 with a `snippet()` extract for the result list.

## PDF / OCR pipeline (planned shape)

1. Watcher or manual import drops a file path onto the ingest queue.
2. Sidecar opens the PDF, reads page count and PDF metadata, hashes the bytes.
3. For each page: try to pull the embedded text layer. If the page yields
   meaningful text, store it directly.
4. If a document has no usable text layer (or fewer than a threshold of words
   per page), the document is flagged `has_text_layer = 0` and goes through
   OCR: rasterize each page at a fixed DPI, run Tesseract, store the result.
5. After all pages are stored, the FTS5 triggers have made the document
   searchable. The UI receives a "document ready" event and the library row
   flips from "ingesting" to "ready".
6. Failures (corrupt PDF, unreadable page, OCR timeout) are recorded against
   the document row so they show up in the library as "needs attention",
   rather than silently disappearing.

## On-disk layout

The user picks a corpus root the first time they launch DeclassRoom. The app
never writes inside the corpus root; it only reads.

```
<corpus_root>/                       user-chosen, read-only from the app's perspective
  cia-reading-room/
    STARGATE/...
    MKULTRA/...
  nara-jfk-2025/
    ...
  aaro-foia-2024-q4/
    ...

<app_data>/declassroom/              writable
  declassroom.sqlite                 documents, pages, FTS5, annotations, tags, collections, citations
  ocr-cache/                         per-page rasterizations kept while OCR is running
  logs/                              sidecar + shell logs
  settings.json                      user prefs (theme, reader defaults, OCR language)
  exports/                           citation export output (BibTeX, CSL JSON, markdown)
```

The app records file paths relative to the corpus root, so moving the whole
corpus folder elsewhere only requires re-pointing the root at first launch.

## Code organization (planned)

- `src-tauri/Cargo.toml`: Rust deps and release profile tuning.
- `src-tauri/tauri.conf.json`: window config, sidecar registration, bundle
  metadata, capability allowlist (no network, scoped FS read on corpus root).
- `sidecar/package.json`: Fastify + better-sqlite3 + tesseract bindings.
- `package.json`: pnpm scripts (`dev`, `build`, `tauri`).
- `vite.config.ts`, `tsconfig.json`: frontend build.
- `.github/workflows/ci.yml`: planned build + test pipeline.
