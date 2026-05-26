<p align="center">
  <img src="assets/banner.svg" alt="DeclassRoom" />
</p>

A personal study tool for declassified government documents. Ingest a folder of PDFs (or scans), let it index the text, then search, annotate, and link the documents while reading them offline. The corpus is whatever you point it at - CIA Reading Room exports, National Archives release packages, FOIA dumps you saved off an agency website, scanned papers from a public archive.

Status: early prototype (pre-code)

---

## What this is

- A local-first reader and study tool for PDF / scanned document collections.
- Full-text search across the whole corpus via SQLite FTS5.
- Annotations, highlights, tags, and per-doc notes.
- Cross-document linking, so a citation in one report can point at the page in another.
- OCR fallback for scans that ship without an embedded text layer.
- All data on disk, no cloud, no telemetry, no account.

## What this is not

- Not a host for the documents themselves. You bring your own corpus, sourced from public release channels (CIA Reading Room, National Archives, agency FOIA portals).
- Not a claim about the content. DeclassRoom is a reading and search tool. It does not endorse, refute, or interpret what the documents say. Read them yourself.
- Not a cloud service. There is no sync, no shared library, no account system.
- Not a publishing tool. There is no upload path back to the internet.

## Subject matter

DeclassRoom is targeted at the kinds of document collections that the US government has officially released, including:

- CIA Reading Room / CREST exports - Stargate Project / Gateway Experience papers, MKULTRA and related human-research files, general Cold War history.
- National Archives newly-released collections, including the JFK records and other declassification batches.
- FOIA responses from agencies covering UAP / UAS reporting, AARO and predecessor program files, and other historic programs as they get released.
- Any other PDF corpus that fits the same "scanned reports plus typed appendices" shape.

The tool does not editorialize. If you want to study the Stargate files for what they say about remote viewing protocols, or the MKULTRA files for what they say about institutional ethics review, that is your call.

## Vision feature list

What we want to ship, in roughly the order we plan to build it:

- **PDF import + indexing** - point at a folder, get a queue, watch each file get parsed, hashed, and added to the FTS5 index.
- **Full-text search** - corpus-wide queries with snippet previews, page jumps, and per-collection filtering.
- **OCR for scans** - any PDF without a text layer goes through Tesseract before it lands in the index.
- **Reader with annotations** - highlight, sticky-note, and per-document margin notes, all persisted locally.
- **Tags + collections** - group documents into named collections (e.g. "Stargate", "MKULTRA Subprojects", "AARO 2024 release") and filter search by them.
- **Cross-document links** - select a passage, link it to another document or page, render the back-references in the reader.
- **Citation export** - pull annotations and quotes out as BibTeX / CSL JSON / plain markdown for use in notes apps.
- **Import wizards** - opinionated importers for the major public dumps (CIA Reading Room ZIPs, National Archives release folders) that get the metadata right on first try.

None of this exists in code yet. This repo is currently the design notes.

---

## License

This project is dual-licensed:

AGPL v3 - free for open-source use. Derivatives and SaaS deployments must release their source under AGPL.
Commercial license - for proprietary / closed-source use or hosted services that do not want to comply with AGPL source-disclosure requirements. Contact for terms.
