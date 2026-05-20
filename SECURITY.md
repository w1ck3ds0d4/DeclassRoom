# Security

## Posture

DeclassRoom is a local-only application. The app makes no network requests
on its own. The only planned outbound traffic is from explicit, per-action
import wizards (e.g. "fetch this public release ZIP"), and even those ask
before doing anything.

There is no telemetry, no analytics, no auto-update ping, no remote
configuration channel.

All user data lives on disk:

- The PDFs themselves live in a user-chosen corpus root. The app reads from
  there; it never writes back into it.
- The SQLite database, annotations, tags, collections, OCR cache, logs, and
  citation exports live under the platform `app_data` directory.

## Threat model

In scope:

- Tampered or substituted source documents. The app hashes every PDF on
  ingestion (SHA-256) and records the hash on the `documents` row, so a
  user can verify a local copy still matches the bytes they originally
  ingested, and can spot-check against a published hash where one exists.
- Accidental data loss. Annotations live in SQLite; the database file can
  be backed up or copied like any other file.
- Untrusted local input. PDF parsers have a long history of bugs; we treat
  every PDF as hostile input and contain parsing inside the sidecar with
  no special privileges.

Out of scope:

- A user with local code execution on the device. Anyone with read access
  to `<app_data>/declassroom/` can read every annotation and every page of
  extracted text in plain form.
- Disk encryption. DeclassRoom does not encrypt its database. Use OS-level
  full-disk encryption if you need that.
- Network-level adversaries. The app is offline by default; import wizards
  that fetch material over the network are the user's call.
- Provenance of the public release itself. DeclassRoom verifies that the
  bytes on your disk match the bytes you ingested, not that those bytes
  match what some agency originally released.

## Document hashing and provenance

Every ingested document gets a SHA-256 of the file bytes recorded in the
`documents` table. This serves two purposes:

- **Local integrity** - re-running ingestion on a file detects whether the
  bytes have changed; a re-ingest with a different hash triggers a fresh
  index for that document.
- **Source verification** - where a public release publishes hashes
  alongside the PDFs (some NARA release indices do this), the user can
  compare. The app does not phone home to fetch external hashes.

The hash is shown in the document detail view so a user can confirm a local
copy matches a hash from the original release page.

## Sidecar permission boundaries (planned)

The Fastify sidecar runs as a Tauri sidecar process. Its planned posture:

- Binds only to a loopback port chosen at launch. Not reachable from outside
  the machine.
- File-system access scoped to:
  - read on `<corpus_root>` and its subtree;
  - read and write on `<app_data>/declassroom/`.
  No other paths are reachable through the sidecar's helpers.
- No outbound network calls by default. The capability is off in
  `tauri.conf.json`'s allowlist.
- Import wizards that need to fetch over the network do so via a Tauri
  command that turns on the outbound-network capability for the duration
  of that single fetch, then revokes it. The wizard tells the user the URL
  before the request goes out.
- Subprocess invocations (Tesseract) are launched with fixed arguments and
  scoped to files inside the OCR cache directory. No shell interpolation
  of user input.

## Hardening posture (planned)

- Tauri 2 capability allowlist tuned to the minimum the UI needs.
- File-system reads on the corpus root mediated through the shell so the
  sidecar cannot escape into other parts of the disk.
- PDF parsing isolated to the sidecar process so a malformed PDF that
  crashes the parser cannot bring down the reader UI; the shell restarts
  the sidecar and surfaces the failing document as "needs attention".
- SQLite opened with `PRAGMA foreign_keys = ON` and `PRAGMA journal_mode =
  WAL`. Parameterized queries everywhere; no string concatenation of user
  input into SQL.
- Release builds strip debug symbols and use `panic = "abort"` on the Rust
  side.

## Vulnerability disclosure

This is a solo-maintained project. If you find a security issue, please
report it privately to `daniel.svs@outlook.com`. Include:

- A reproducer or minimum failing test case.
- Affected version.
- Platform (Windows, macOS, Linux) and OS version.
- Impact assessment.

Public disclosure of unfixed issues is discouraged. Coordinated disclosure
on a reasonable timeline is preferred. There is no bug bounty.

## License + redistribution

DeclassRoom is MIT-licensed (see [LICENSE](LICENSE)). The repository is
currently private; see [COMMERCIAL.md](COMMERCIAL.md) for the current
posture on commercial use.
