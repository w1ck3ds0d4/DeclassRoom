# DeclassRoom v1 Roadmap

## What v1 is

A local-first study tool for declassified government document collections
(CIA Reading Room exports, FOIA dumps, National Archives releases). User
points at a folder of PDFs; the app indexes them for full-text search,
supports annotations and cross-document linking.

## Current state

Pre-code. README, ARCHITECTURE, ROADMAP, SECURITY, CONTRIBUTING, COMMERCIAL
docs exist. Tech stack not yet committed (likely Electron or Tauri with
SQLite FTS5). No `package.json`, no `src/`, no asset folder. The existing
ROADMAP outlines 5 phases (Phase 1 = index 1000 PDFs + search; later phases
add OCR, annotations, cross-doc linking).

## v1 acceptance criteria

- [ ] App shell chosen and scaffolded (recommend Tauri 2 + React + Vite for
      footprint and the existing toolchain pattern)
- [ ] SQLite + FTS5 wired (better-sqlite3 for sync API in sidecar OR sql.js in renderer)
- [ ] PDF parser: pdfium (via Tauri Rust bindings) OR pdf.js worker for text extraction
- [ ] "Open folder" flow: user picks a directory, app indexes all PDFs in tree
- [ ] Index respects per-doc metadata (filename, declassification date, agency) when filename conventions are recognizable
- [ ] Reader UI: PDF viewer, search-within-doc, search-across-corpus
- [ ] Search ranking: BM25 via FTS5 with snippet preview
- [ ] CI gates (tsc, lint, vite build, cargo fmt + clippy + test, tauri build)
- [ ] Documented "1000 PDFs index in under 5 minutes" benchmark
- [ ] At least 10 unit tests covering parser + indexer + ranker
- [ ] v1.0.0 tag after self-use on a CIA Reading Room export

## Milestones to v1

### M1. Foundation scaffold (M)

- [ ] `flutter create` or `pnpm create tauri-app` (commit to Tauri direction)
- [ ] App shell with file-open dialog
- [ ] CI workflow

**Acceptance:** `pnpm tauri dev` boots an empty shell with a "Pick folder" button that prints the chosen path.

### M2. PDF parser + indexer (M/L)

- [ ] Choose between pdfium (Rust) and pdf.js (JS)
- [ ] Text extraction: per-page text + page numbers + line layout (for snippet preview)
- [ ] Bulk indexer with progress UI
- [ ] FTS5 schema: `documents`, `pages`, `pages_fts`
- [ ] Tests on a curated 10-PDF fixture

**Acceptance:** "Pick folder" -> indexer runs, progress bar updates, search returns hits.

### M3. Reader UI (M)

- [ ] PDF viewer (pdf.js render or platform native)
- [ ] Search bar with results list (doc name + page + snippet)
- [ ] Click result -> jump to page, highlight matches
- [ ] Search within current doc

**Acceptance:** user can find a phrase across the corpus and land on the exact page.

### M4. Metadata recognition (S)

- [ ] Filename-pattern recognizer for common Reading Room / FOIA conventions
- [ ] Per-doc metadata panel (filename, agency, date if detected)
- [ ] Filter search by metadata

**Acceptance:** user can scope search to "CIA, 1985, declassified-by-XYZ" if the filename conveys it.

### M5. Benchmark + tag (S)

- [ ] Run on a 1000-PDF corpus, capture timing
- [ ] Document in README
- [ ] Tag `v1.0.0` after personal smoke

**Acceptance:** documented benchmark; tag pushed.

## Beyond v1 (post-1.0 polish - currently Phases 2-5)

- OCR for scanned PDFs (Tesseract via Rust)
- Annotations (highlights, notes per page)
- Cross-document linking (citations, redaction patterns)
- Tag system + saved searches
- Export annotated set to PDF

## Out of scope for v1

- Cloud sync
- Multi-user collaboration
- Public corpus hosting
