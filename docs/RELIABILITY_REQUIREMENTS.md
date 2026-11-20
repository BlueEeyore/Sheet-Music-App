# HighScore reliability requirements

Status: engineering requirements derived from the user's explicit direction that reliability is priority number one. Final implementation details and measurable performance limits will be incorporated into the approved design and implementation plan. This document does not authorize additional product features or relax the first-release feature scope.

## Priority and release policy

- Reliability takes precedence over development speed, animation polish, and convenience of implementation.
- All approved first-release features remain required. Use internal validation milestones to establish reliable foundations before expanding the implementation.
- Do not label the release ready with known data-loss, corrupt-library, incorrect-version, or page-navigation defects in supported workflows.
- A passing test suite, including 100 repeated successful runs, is evidence of reliability rather than a mathematical guarantee of zero future failures.
- Validate actual behavior on the user's Galaxy Tab S8+ 5G with S Pen and Pixel 6a. Emulator-only validation is insufficient for stylus input, memory, audio timing, lifecycle, and page-turn performance.

## Saving and recovery

- Preserve immutable imported originals. Rendering, annotations, snippets, crops, and arrangements must not rewrite them.
- Use transactional changes for structured data and stable identities for scores, versions, page occurrences, setlist entries, layers, and references.
- Render immediate interaction feedback while distinguishing pending changes from changes durably committed. Never report a save as complete before persistence succeeds.
- Keep saving independent of internet access and of graceful app shutdown. Handle Android process termination as a normal lifecycle event.
- Define commit boundaries for strokes, text edits, selections, page arrangements, and version changes. Explicitly document the fate of an operation interrupted before its commit rather than promising preservation of an unfinished action.
- Coordinate filesystem and database changes using staging, atomic publication where supported, and recoverable operation records. A database transaction alone does not make associated file writes atomic.
- After interruption, recover to the last committed state, finish or discard incomplete staged operations safely, and avoid exposing partially imported scores or half-created versions.
- On storage exhaustion or a failed write, retain the last committed data, surface an actionable failure, and never imply that unsaved changes are safe. Approved behavior: identify unsaved work, temporarily prevent further editing, allow reading saved content, and offer retry/recovery export where possible.
- Validate backups before restore and use a recoverable restore process. Never overwrite the only working library with an unvalidated archive.
- Version database schemas, test migrations against previous fixtures, and retain a recoverable pre-migration state. A failed migration must not silently reset the library.
- Garbage collection must account for shared source PDFs and all versions, snippet dependencies, retained undo history, trash, backups in progress, and in-progress operations before removing files.

## Performance and input correctness

- Local reading and editing must not depend on online credentials, API availability, or a network connection. Protected scores intentionally require their local PIN/password or recovery key, according to the score/setlist session rules.
- Bound rendering caches and background queues. Prefetch the next likely page/spread and relevant link destinations within the memory budget; do not preload arbitrary entire libraries.
- Keep PDF decoding, import scans, thumbnail generation, and exports off the interaction-critical path.
- Defer automatic folder checks/imports during viewing and performance by default, honoring the user's configurable setting. Essential saves, recovery, and visible-page rendering must never be deferred by that setting. When deferral is disabled, background work must remain bounded and lower priority than interaction and persistence.
- A page turn must never unexpectedly blank the currently displayed usable page while the next one is being prepared. Define recoverable error behavior for pages that cannot render.
- Model single-page, two-page stride-one, two-page stride-two, half-page, link-jump, and setlist-boundary transitions explicitly, with deterministic forward/backward behavior.
- Distinguish intentional repeated input from key-repeat or duplicate device events. Verify actual pedal behavior before choosing a suppression policy that could discard deliberate turns.
- Test transformations and hit targets through crop, rotate, straighten, zoom, stretch, and half-page states; annotations must remain attached to the intended page content.
- Use explicit resource ownership for PDF documents, images, microphone/audio streams, and lifecycle subscriptions; verify disposal and long-running memory behavior.
- Qualify package versions and native binaries with regression tests before updating them. Avoid unbounded dependency ranges in the implementation handoff.
- Respect PDFium's documented threading constraints: its C APIs are not thread-safe. Verify the wrapper's scheduling guarantees and serialize all native PDFium access through an explicitly owned execution boundary; do not assume independent Dart isolates make concurrent native calls safe. Source: https://pdfium.googlesource.com/pdfium/+/refs/heads/main/public/fpdfview.h

## Required validation scenarios

- Terminate/restart at persistence boundaries during import, drawing, notes edits, version duplication, layer merging, snippet cut/paste, page reordering, trash operations, export, and restore.
- Verify previously committed edits and setlist/version choices survive normal exit, process kill, and device restart.
- Exercise full storage, revoked source permissions, expired Drive authorization, disconnected/removable sources, unavailable network, malformed PDFs, and interrupted downloads. Unavailable sources must not be misreported as confirmed source deletion.
- Test duplicate automatic checks and repeated imports for idempotency and correct duplicate reporting.
- Validate backup/restore with originals, all versions, annotations/layers, snippets, shared/version/global notes, labels, nested folders, setlists, placeholders, and settings.
- Check that missing or trashed content and broken references never silently target a different score/page/version.
- Verify export appearance and clickable link destinations in an independent PDF reader. Exercise crop/rotation, reordered and imported pages, clipping, layer visibility/opacity, snippets, and text/stamps.
- Rehearse a long setlist offline with repeated page turns, half-page transitions, version choices, links, metronome, and supported page-turner input. Observe memory, responsiveness, and battery use on both physical test devices.
- Test repeated orientation changes, background/resume, interrupted input, permission denial, and simultaneous allowed tool use.
- Pair focused unit tests of state transitions and geometry with database/filesystem integration tests and physical-device end-to-end checks. Do not replace recovery tests with superficial UI snapshots.

## Evidence required in the implementation handoff

- Exact supported environment and representative test PDFs, including scans, vector notation, mixed dimensions/orientations, large documents, and problematic files.
- Deterministic commands for automated verification, plus a physical-device test checklist with expected results.
- Defined latency, memory, and durability acceptance criteria once representative document sizes and use cases are confirmed.
- Clear stop conditions: a failed foundation validation must be investigated and resolved before dependent milestones are marked complete.
- A traceable mapping from each reliability requirement to its implementation boundary and verification scenario.

## Protected-content reliability

- Authentication is scoped to the current direct-score or setlist session, never a persistent global unlocked flag. Background/screen-off preserves a live session; leaving its scope revokes it. Process death discards keys and requires fresh authentication.
- Protect all content-bearing storage, including database payloads, write-ahead logs, undo records, snippets, thumbnail caches, temporary exports, saved PDF passwords and backups. Public titles and labels are the approved exception.
- Never persist plaintext secrets to logs or crash reports. No cloud crash reporting is required.
- Authenticate encrypted bytes before supplying them to the PDF engine or object decoder. Failed authentication is corruption/wrong credential, never an empty successful document.
- Exercise recovery-key reset, password changes, protection conversion, backup import and concurrent editing with fault injection. Never discard the last readable copy before the replacement is validated and published.
- Backups cannot depend on an Android Keystore key from the original installation. Test restoration on a clean second installation.
- Shared assets must not let protected content remain accessible through an unintended unprotected storage reference. Explicit user-approved unprotected copies are distinct and outside that score's lock.
