# Contributing

DeclassRoom is a solo side project. The repository is currently private and
there is no shipped code yet, so the realistic state of contribution is:

- Pull requests are welcome once the project goes public and there is
  something to build on.
- Review will be slow. This is not anyone's day job.
- Issues that come with a reproducer are easier to act on than feature
  requests without one.

If you are reading this inside the private repo, you probably already have
direct contact with the maintainer.

## Prerequisites (planned)

| Tool    | Version       | Why                              |
| ------- | ------------- | -------------------------------- |
| Node.js | 22+           | Sidecar runtime                  |
| pnpm    | 9+            | Package manager                  |
| Rust    | Latest stable | Tauri 2 shell                    |
| Git     | 2.40+         | Version control                  |

No Docker, no database server. SQLite is bundled.

## Setup (planned)

```bash
git clone https://github.com/w1ck3ds0d4/DeclassRoom.git
cd DeclassRoom
pnpm install
pnpm tauri dev
```

There is no `.env`. Everything runs out of the box once the toolchain is in
place.

## Commit conventions

- Commit messages use a `(type)` prefix:
  - `(feat)` - new behavior
  - `(fix)` - bug fix
  - `(refactor)` - internal change, no behavior diff
  - `(docs)` - docs only
  - `(chore)` - tooling, deps, build
  - `(test)` - tests only
- Keep the subject line short and lowercase-ish.
- No em dashes anywhere in commit messages or PR descriptions. Use ` - `,
  `:`, or rephrase.
- No AI attribution footers in commits or PR bodies. If a tool helped you
  write the patch, that is fine, but the commit history should not say so.

Examples:

```
(feat) add OCR fallback for scanned PDFs
(fix) handle PDFs with zero-width text layer
(docs) clarify corpus root behavior in HOW_IT_WORKS
```

## Code conventions (planned)

- TypeScript strict mode on the frontend and sidecar.
- Rust: format with `rustfmt`, lint with `cargo clippy --all-targets`. No
  `unwrap()` on user-facing paths.
- Sidecar logs use a `[ModuleName]` prefix and `console.warn` for recoverable
  failures, `console.error` for things that mean a document failed to ingest.
- No outbound network calls outside an explicit import wizard. If you add
  one, gate it behind a user-visible confirmation.
- Never write inside the corpus root. The corpus root is read-only from
  the app's perspective.

## Pre-release checks (planned)

Before tagging a release or pushing to the main branch:

- `cargo fmt --all -- --check`
- `cargo clippy --all-targets -- -D warnings`
- `cargo test`
- `pnpm tsc --noEmit`
- `pnpm vite build`
- `pnpm tauri build`
- Install the built artifact on a clean profile and smoke-test ingestion,
  search, annotation, and one cross-doc link end-to-end.

Failing CI is not a release blocker only if a human has confirmed the failure
is a false positive and documented why.

## Reporting issues

For security issues, see [SECURITY.md](SECURITY.md). For everything else,
open a GitHub issue with:

- What you did.
- What you expected.
- What happened instead.
- Your platform and the app version (once there is one).
