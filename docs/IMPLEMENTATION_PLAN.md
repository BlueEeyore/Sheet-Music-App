# HighScore — detailed implementation plan

Status: ready for implementation handoff, 2026-11-20. This is a plan, not implemented software. No app build, device test or performance qualification has yet passed.

This document is intended for an implementation model that benefits from narrow tasks, explicit constraints and observable completion criteria. Implement in dependency order. Do not interpret a mockup's placeholder behavior as functionality.

## 0. Read this before changing code

Read these files in order:

1. `docs/DESIGN_DECISIONS.md` — confirmed user decisions and superseded proposals.
2. `docs/DESIGN.md` — exact behavior, ownership, state machines and architecture.
3. This plan — build sequence, file boundaries and milestone acceptance.
4. `docs/ACCEPTANCE_TESTS.md` — mandatory checks and release evidence.
5. `docs/RELIABILITY_REQUIREMENTS.md` — cross-cutting invariants.
6. `SPEC.md` — original feature checklist; later decisions override conflicts.

The task is an Android app, not a website. Do not package the HTML mockup in a WebView. Implement with Flutter/Dart, Drift/SQLite, the qualified pdfrx/PDFium adapter, and Android native bridges where required. The eventual iPad port needs clean adapters, not an iPad app in this release.

### 0.1 Non-negotiable implementation rules

- Reliability is the top priority. All retained features still belong in release one.
- No commits or pushes; the user handles git. Never delete unrelated files, the original spec, or editor swap files.
- No account/cloud library, backend hosting, paid dependencies, paid quotas, analytics service or automatic cloud backup.
- Never modify a source PDF/folder, including when app versions change.
- No live updating imported scores from later source edits. Linked folders import new items only.
- No version merging. Setlist merging and layer merging remain required.
- No label sorting, OCR, PDF-text search, or alternative temperaments.
- No fake save success, silent corrupted-library reset, silent raster export fallback or automatic missing-entry skip.
- A component labelled complete must run real logic; a toast saying “not implemented” is not acceptable.
- Do not invent PDF library APIs from memory. Inspect the exact pinned package and write a proof for every needed native capability.
- Do not install a new framework to avoid understanding an existing approved component. Additional dependencies must be free/open source; bring substantive alternatives/new dependency recommendations to the user before adoption, as requested. Use approved platform SDK functionality where practical.
- Internal technical defaults below can be tuned using measured evidence without adding features. Changes to user-visible behavior or scope require asking the user first.
- If blocked by hardware, permissions, a vendor API or an unavailable toolchain, record the exact failed check and continue independent work; do not claim the milestone passed.

### 0.2 Run one milestone at a time

For each milestone:

1. Inspect relevant current code and this milestone's dependencies.
2. List the specific files and invariants affected.
3. Make the smallest coherent implementation, including real error paths.
4. Run targeted tests for the changed behavior, then required milestone integration tests.
5. Record evidence and unresolved failures in `docs/IMPLEMENTATION_STATUS.md`.
6. Mark complete only when its completion criteria pass.
7. Continue to the next milestone. Do not leave all failure handling/testing until the final milestone.

Create `IMPLEMENTATION_STATUS.md` at implementation start with columns: milestone, status, actual files, test commands/results, physical evidence, blockers, next step. Keep dependency/version decisions in `docs/COMPATIBILITY.md`; keep product decisions in the existing decision record. Do not produce a new design on every turn.

## 1. Milestone dependency graph

| ID | Milestone | Depends on |
| --- | --- | --- |
| M00 | Toolchain and capability qualification | Nothing |
| M01 | App shell, themes, domain types and dependency composition | M00 |
| M02 | Durable storage, asset encryption primitives, command journal | M01 |
| M03 | Protected access, recovery and conversion | M02 |
| M04 | Manual import, linked source identity and PDF passwords | M02, M03 |
| M05 | Library, labels, folders, recents and trash | M04 |
| M06 | Version model, duplication/deletion and setlists | M05 |
| M07 | Reader navigation, page rendering and geometry | M06, qualified PDF path |
| M08 | Stylus/finger input, pen/highlighter and erasing | M07 |
| M09 | Selection, layers, text, stamps and persisted history | M08 |
| M10 | PDF snippets, crop and page arrangement | M09 |
| M11 | Links, bookmarks and formatted notes | M10 |
| M12 | Metronome, tuner, reference tone and pedal input | M07; integrate M08–M11 permissions |
| M13 | PDF export | M10, M11, M03 |
| M14 | Backup, replacement restore and merge restore | M03–M13 persisted schemas |
| M15 | Background source imports and full Settings | M04–M14 |
| M16 | Recovery, physical reliability and release packaging | All previous milestones |

M00 must validate difficult capabilities early even though production UI arrives later. A working PDF viewer alone does not prove export or encrypted random access. M12 may be developed independently after M07, but final integration cannot skip later permission/lifecycle tests.

## 2. Planned repository layout

Create an Android-only Flutter scaffold in the repository root without overwriting existing documents. Keep the application label HighScore; initial technical application ID can be `app.highscore.personal` for local builds and must be verified before any distribution. Do not change it after installing real user data without a migration plan.

```text
lib/
  main.dart
  app/
    bootstrap.dart                 # initialization/recovery before normal routing
    composition_root.dart          # constructs services; no hidden service locator
    app_shell.dart
    router.dart                    # explicit direct-score / setlist route scopes
    theme/highscore_theme.dart
    theme/highscore_icons.dart
  core/
    ids.dart                       # typed ScoreId, VersionId, PageOccurrenceId, ...
    errors.dart                    # typed failures, never user text as error code
    clock.dart
    result.dart
    geometry/                      # matrix, clipping, coordinates, region operations
    persistence/                   # commands, revisions, publication coordinator
    security/                      # capabilities, envelope DTOs, access policies
    platform/                      # interfaces and Android adapters
  data/
    database/                      # Drift schema, DAOs, migrations and fixtures
    assets/                        # ciphertext chunks/manifests and dependency GC
    repositories/                  # only sanctioned mutation/read entry points
    serialization/                 # versioned JSON/binary DTO encoders
  features/
    library/                       # controllers, view models, views
    imports/
    sources/
    versions/
    setlists/
    viewer/                        # rendering, navigation FSM, viewport ownership
    annotations/                   # stroke geometry, commands, tools and layers
    snippets/
    crop/
    page_arrangement/
    references/                    # bookmarks and links
    notes/
    protection/
    audio_tools/
    export/
    backup/
    settings/
  services/
    pdf/                           # one PDF-engine owner and serialized queue
    audio/                         # sample-clock bridge
    scheduling/
android/
  app/src/main/kotlin/.../
    HighScorePlatformPlugin.kt
    SafBridge.kt
    CryptoBridge.kt
    KeystoreBridge.kt
    WorkBridge.kt
    StylusBridge.kt                # only if Flutter events lack required data
    ScreenPolicyBridge.kt
  app/src/main/cpp/                # narrowly scoped PDF/audio native adapters
assets/
  fonts/                           # Liberation Sans and Bravura; licences included
  stamps/                          # supported-symbol catalogue, not pirated assets
  audio/                           # authored/generated click samples if used
  templates/                       # plain/staff template definitions
  licenses/
test/
  domain/ geometry/ persistence/ security/ imports/ viewer/ annotations/
  export/ backup/ scheduling/ fixtures/
integration_test/
  offline_rehearsal_test.dart
  protected_sessions_test.dart
  crash_recovery_test.dart
  library_scale_test.dart
tool/
  generate_test_fixtures.dart
  run_fault_matrix.dart
  verify_release.dart
  check_documentation.py
```

These are target module boundaries, not permission to create hundreds of empty files. Split a module when responsibilities are distinct; do not place domain behavior in a single 5,000-line viewer widget. Commands and repository APIs are required even if a feature currently has one screen.

## 3. Core implementation contracts

### 3.1 Typed IDs and commands

Define immutable wrappers around serialized stable IDs. Prevent accidental interchange of `SourcePageIndex` (zero-based), displayed page number (one-based), `PageOccurrenceId`, `VersionId` and `SetlistEntryId`. Store indices only where an ordinal is intended; references always use IDs.

Every mutation implements a command contract equivalent to:

```text
Command {
  commandId
  affectedScoreIds[] / affectedVersionIds[]
  expectedRevisions{}
  accessRequirements
  validate(snapshot) -> typed result
  stageAssets(assetWriter) -> immutable staged references
  apply(transaction) -> new revisions + inverse/history data
}
```

`CommandCoordinator.execute()` checks access at execution time, acquires owner locks in sorted ID order, stages immutable assets, awaits one DB publication transaction, emits a saved revision, then schedules cleanup. It accepts a `FaultInjector` in test builds at named phases. Tests must be able to kill/fail at each phase without subclassing UI widgets.

Queries return immutable domain snapshots. Controllers maintain dirty/pending/saved/error states. A new command with a stale expected revision fails or deliberately rebases using a tested feature-specific rule; it never overwrites a newer edit silently.

### 3.2 Ports

Write interfaces before adapters; implement only the required methods.

| Port | Required behavior |
| --- | --- |
| PdfEnginePort | Open authorized random-access bytes, enumerate stable source-page descriptors, render bounded tiles, manipulate/export a separate document, close/cancel |
| CryptoPort | Derive/wrap/unwrap keys, authenticated records/chunks, random bytes, opaque native key handles, dispose |
| AssetStore | Stage immutable encrypted assets, range-read verified chunks, pin/release dependencies, inspect/recover operations |
| DocumentSourcePort | Pick/list/read with opaque provider identities; paginated traversal, cancellation, per-item errors |
| AudioPort | Start/stop/configure native metronome, microphone pitch stream, reference tone; focus/route lifecycle |
| BackgroundWorkPort | Unique constrained work request, cancel/update, checkpoint/retry notification to domain |
| ScreenPolicyPort | Apply/release keep-awake policy; no global permanent wake lock |
| AccessSessionPort | Open/close direct-score and setlist scopes, authenticate, validate a capability, revoke buffers |
| SecureCredentialPort | Store/retrieve provider tokens and unprotected remembered PDF passwords using platform protection |

Cancellation has real semantics: discard stale rendering results, stop import publication before cancellation, close sources and release native handles. Cancelling after a durable commit returns a committed result rather than pretending it rolled back.

### 3.3 Test invariants that must exist early

- Every published asset reference resolves to a complete validated asset.
- All content changes preserve original PDF bytes.
- Every version has at least one page and every score at least one version.
- A setlist entry's version belongs to its score.
- A link destination is the intended occurrence or explicitly broken.
- A protection domain cannot read another domain's ciphertext through a shared asset reference.
- No completed command is acknowledged before persistence.
- Undo/redo cursors and object revisions agree after restart.
- Imported identity + source tombstone makes automated discovery idempotent.
- `backgrounded` is not `sessionClosed`; audio stop and protected-session retention are separate decisions.

## M00 — qualify the risky foundations

### Work

1. Inspect installed Flutter/Dart/Android SDK/JDK/NDK and connected device versions. Record actual versions and ABI support. If missing, provide exact setup needs; do not claim builds ran.
2. Choose a mutually compatible stable toolchain and exact pdfrx/Drift versions after reading their actual package manifests. Commit no lockfile yet yourself, but create/update it in the working tree for the user. Pin native binaries/checksums and record transitive licences.
3. Use the approved Android SDK for crypto, SAF, scheduling and native audio where practical. Additional Dart packages or native replacements require the dependency review requested by the user; recommend exact package, purpose, licence, maintained source and fallback before adoption. Do not add libsodium/SQLCipher/state-management frameworks simply because they were considered during research.
4. Create disposable qualification code under `tool/qualification/`, isolated from production modules. Keep reusable fixture generators/tests; do not ship qualification screens.
5. Exercise PDF rendering with scans, vector text, mixed page sizes, PDF intrinsic rotation, encrypted PDFs, huge single pages and malformed inputs. Open via a bounded custom read callback over encrypted chunks; no plaintext temporary PDF fallback.
6. Prove export can preserve base-page vectors/text while adding a path, highlight, embedded text, a stamp, a cropped/rotated page, raster snippet and internal link. Reopen in a second independent reader/parser. Confirm native resource ownership and serialized access.
7. Log only non-sensitive pointer metadata in a test fixture on S Pen: pointer type, pressure, button transitions, cancellation, historical/coalesced samples and palm events. Verify first contact can draw without relayout drift.
8. Verify durable DB/file operations and staged replacement on actual Android storage. Demonstrate deterministic fault injection at write/rename/DB commit boundaries.
9. Benchmark platform AES-GCM/PBKDF2 and chunk reader performance; verify KDF input encoding and vectors. Define format bytes before any real user data is written.
10. Prove native sample-clock click timing can run while repeatedly rendering PDF pages. Verify microphone pitch pipeline with generated tones, independently of UI.
11. For Drive, prove selected-folder discovery finds files created after authorization, and distinguishes rename/edit/delete/unavailable. Do not assume `drive.file` grants recursive access to arbitrary future children. If broader read-only consent is necessary, document it for the user's optional connection flow. No scope for source writes and no paid overages.

### Evidence and exit

Create `docs/COMPATIBILITY.md` with versions, licences, commands, device results and capability matrix. PDF adapter public calls must map to verified package/native APIs. Record unsupported encryption types and document fixtures. A failed export, encrypted read or S Pen requirement blocks its dependent milestone; request a concrete alternative rather than weakening behavior.

Engineering target: support the Android versions on both user devices. Select minimum SDK from verified required APIs/toolchain and record it, rather than advertising untested old Android support. Initial planned platform API floor is 26 for native crypto/audio facilities; any higher floor imposed by a package must be explained before restricting distribution.

## M01 — shell and domain foundation

### Files

`lib/app/`, typed IDs/errors/clock, feature controller interfaces, theme/icons, composition root, `assets/fonts/`, basic test harness.

### Steps

1. Scaffold Flutter Android without replacing existing docs. Add analysis options with asynchronous misuse and unawaited persistence work treated as errors.
2. Bundle Liberation Sans regular/bold and Bravura with licence files. No fonts downloaded during app use.
3. Implement light/dark/system theme tokens, `#2F7434` accent, navy, pale library + and matching selected states. Check contrast on both themes; use a lighter green foreground variant where the exact dark accent lacks foreground contrast, keeping the approved accent for filled surfaces.
4. Implement width-responsive shell: Library, Recents, Setlists, Settings. Use real routes and state restoration only for non-sensitive navigation metadata.
5. Add semantic icon buttons, 48dp targets, scalable text, focus and reduced-motion support. No tool-name words on default score toolbars.
6. Build the central dependency graph with injected clock/random/fault/source/PDF/audio ports. No plugin call from domain classes.
7. Define every typed entity and immutable event needed for subsequent storage without making fake repositories look like production completion.

### Verify / complete

Unit tests for ID/type/serialization boundaries; widget tests for phone/tablet navigation and both themes at increased text size. App launches offline, no service/account registration is required, and the Settings route is usable. This milestone does not count as completed library functionality.

## M02 — durable storage and encrypted assets

### Files

`data/database/`, `data/assets/`, `core/persistence/`, `core/security/`, Android crypto/Keystore bridge, serialization codecs.

### Steps

1. Define Drift tables matching DESIGN.md ownership and foreign-key constraints. Separate public catalogue metadata from encrypted content payloads. Export versioned schema snapshots for migration testing.
2. Enable foreign keys on every connection, use one serialized writer, qualify WAL/durability options. Observe committed database streams, not uncommitted internal transaction streams.
3. Implement canonical serialization with explicit format versions and length limits. Reject invalid enums, duplicate IDs, non-finite geometry and impossible owner references.
4. Implement native crypto using platform APIs, with key handles where possible instead of repeatedly copying secrets through Dart strings. Document the byte encoding for KDF inputs; test non-ASCII passwords and whitespace without silently normalizing them differently on restore.
5. Implement encrypted record envelopes and authenticated chunk manifests. The small-record IV must be fresh on every revision. The asset key/generation must be new on every asset rewrite. Never reuse GCM IV/key pairs.
6. Implement AssetWriter staging and bounded range-reader. An authentication failure must not release unauthenticated bytes. Avoid loading an entire PDF or backup into a single Uint8List.
7. Implement publication phases and startup recovery before normal repository access. Include create, update, duplicate and conversion operation states. Recovery is idempotent across repeated crashes.
8. Implement references/pins for versions, history, trash, pending operations and backup snapshots. GC only after complete dependency enumeration; never delete solely because a UI page closed.
9. Implement saves with pending/saved/error states and typed `StorageFull`, `WriteDenied`, `CorruptAsset`, `RevisionConflict`, `Cancelled` failures. Block new editing on write failure while retaining the last saved reader.
10. Add migration runner with a validated pre-migration snapshot. Never `deleteDatabase()` to recover a migration failure.

### Verify / complete

Tests inject failure at every publication boundary and assert either old complete state or new complete state. Test a second restart during recovery. Test disk-full partial chunk writes, stale command revisions and missing asset hashes. Search database/WAL/staging fixtures for distinctive plaintext protected-content strings. Real crypto vectors and chunk substitution/reorder/truncation tests pass. No acknowledged saved command disappears after process kill.

## M03 — score protection, recovery and access scopes

### Files

`features/protection/`, access-session service, encrypted repositories/key envelopes, router scope hooks, settings protection controls.

### Steps

1. Implement one shared PIN/password vault with credential and recovery envelopes. Show the offline recovery key and confirm external retention before enabling protection. Do not add email recovery, biometric-only unlock or a backend.
2. Implement credential validation/retry backoff, password change and recovery reset. Reset rewrites credential envelope atomically; old backups retain their original unlock material.
3. Store source PDF passwords separately from HighScore credentials. Protected score passwords inherit its encryption; never bypass score protection because the PDF password is remembered.
4. Implement direct-score grant lifecycle: same score/version remains open; back to library or another direct score revokes it.
5. Implement setlist grant lifecycle: opening a setlist with any protected entry prompts once; retained until leaving that setlist. Entering/exiting its viewer and editing the same setlist retains scope. New/repeated entries cannot exploit a stale grant outside that scope.
6. Separate session scope from Android foreground state. Background/screen-off keeps live authentication; process kill/reboot requires fresh authentication. Test route restoration cannot treat a persisted route as a persisted key.
7. Implement content-preserving protect/unprotect conversion with fresh key generation, complete dependency copy, validation, atomic publication and old-generation cleanup. On cancellation/failure retain previous state.
8. Replace locked thumbnails with generic lock placeholders while title/labels remain available. Gate notes/history/clipboard/export/custom PDF callbacks at repository level.
9. Implement operation-scoped unlocking for export/copy/restore where a viewer is not open. Unprotected-output confirmation names the destination; dismissing it cancels the operation.
10. Exclude private app content and keys from automatic Android backup/data extraction. Keystore wrappers for unprotected content are local convenience, never a backdoor to protected keys.

### Verify / complete

Run the access-session matrix in ACCEPTANCE_TESTS.md. In particular: opening a protected-containing setlist prompts even if its first entry is unprotected; subsequent entries do not; background/screen-off do not; leaving/reopening does. Cancellation enters no protected route. Wrong PIN/key cannot decrypt. Corrupt envelope does not reset vault. Tests extract files and find no content-bearing plaintext or portable device-wrapped protected key. Conversion and recovery survive injected crashes.

## M04 — manual imports and source identity

### Files

`features/imports/`, `features/sources/`, SAF/Drive source adapters, source-item tables, import coordinator.

### Steps

1. Implement PDF/file-folder selection using SAF. Retain read-only grants only where ongoing access is needed. Never convert `content://` URIs into guessed filesystem paths.
2. Stream into encrypted staging while calculating content digest. Check cancellation, file size/stream limits, storage budget and PDF readability. Validate page count before creating the first version.
3. Implement password prompt/retry/cancel and Remember on this device. An unattended encrypted import becomes Waiting for password, not a repeatedly failing background loop.
4. Manual identical-content duplicate offers Skip or independent score. Use a keyed digest index for equality checks without publishing raw protected-document fingerprints. Do not decrypt arbitrary locked scores to compare contents.
5. Import each file as its own atomic unit. A folder batch can finish partially with a per-file report; never present an incomplete score as imported.
6. Implement subfolder traversal options and library-folder mapping. Detect traversal cycles/multiple provider-parent references, paginate, and checkpoint only committed discovery results.
7. Source record includes provider type, account/root identity, source-item stable identity, imported revision, ignored later revision, digest, availability, completed scan generation and deletion/reimport tombstone.
8. New source identity imports once. Same identity edited/renamed records information but does not change imported bytes or title. New identity with identical content is duplicate-skipped. Reusing a filename is not sufficient to replace a source identity.
9. Only mark removed after a successful complete authoritative scan or explicit provider deletion event. Permission revoked/disconnected/cloud error is Unavailable, not Removed.
10. Implement optional Drive adapter within the qualified authorization/quotas. Request no HighScore account and no source write operations. User cancellation of Google login leaves local app usable.

### Verify / complete

Fixture tests for hierarchy/flatten/exclude, duplicate filename/different content, same bytes/different name, modified original, rename, permanent delete, failed scan, empty folder, nested permission denial, cancellation, wrong PDF password and malformed PDF among valid files. Original source hashes remain unchanged. Restarted jobs do not duplicate scores. Manual imports work in airplane mode from local sources.

## M05 — library, labels, folders, recents and trash

### Steps

1. Replace shell placeholders with paginated/query-backed score collections; never decode every thumbnail at startup.
2. Implement native-feeling centred Add modal and all approved import entry points. Library + uses pale green glyph on approved green surface.
3. Implement per-score menu and multi-select operations. Add folder creation/rename/move, cycle prevention and delete-folder choice. All mutating actions go through commands.
4. Implement label create/edit/color/assignment; query title/label search with Match any/Match all/exclusions. Parameterize queries. No notes/PDF text search.
5. Implement sorts and recents with stable tie-breakers. Successful score opening updates recents; failed authentication/cancelled import does not masquerade as a successful opening.
6. Implement trash with tombstones and retention settings. Default never purges automatically. Restore preserves IDs and setlist references.
7. Permanent purge requires deliberate confirmation and dependency-aware GC. Unavailable setlist references remain explicit after purge. Do not issue source deletes.
8. Handle empty/loading/error states without replacing the entire screen on every query update. Avoid searching by reading encrypted PDFs.

### Verify / complete

Use 500 generated scores, overlapping labels and nested folders. Verify exact membership for combined filters. Repeat bulk operations and kill during commit. Deleted auto-imports stay suppressed. Locked thumbnails reveal no content. Recents does not change the library's selected version unexpectedly.

## M06 — versions and setlists

### Steps

1. Create the initial Original version on import with its own initial layer and page occurrences. Original is an editable version name, not permission to mutate the immutable imported original bytes.
2. Version duplication creates new version/page/layer/object/reference IDs, copies baseline content, remaps internal links and owns an independent history. Preserve immutable assets within the score where safe.
3. Implement tap-to-switch, long-press selection, contextual Duplicate/Delete and automatic exit when selection becomes empty. No always-visible Duplicate and no Merge versions.
4. Version deletion cannot remove the final version. Resolve affected setlist entries to an explicitly chosen surviving version before commit. Apply all reference updates/deletion atomically.
5. Implement nested setlist folders and create/edit lists. Every entry has unique ID independent from score ID. Reorder by entries, not score IDs.
6. Implement placeholders, repeated scores, entry-specific version chooser, duplicate setlist and merge selected setlists in explicitly chosen order.
7. Add protected-setlist session entry gate before showing its contents. Keep setlist session scope alive through list editing/viewing transitions.
8. Keep library last version/position independent from all setlist selections. Direct setlist entry open starts at page one even if the library was at another page.

### Verify / complete

Duplicate a cropped/annotated version with internal links, edit only the duplicate and prove original behavior unchanged. Use the same score/version three times and different versions twice in one setlist; reorder/delete only one occurrence. Merge lists with duplicate entries, placeholders and protected items. Delete a referenced version with replacement and restart. Library position remains unchanged by a setlist rehearsal.

## M07 — reader, geometry and navigation

### Files

`features/viewer/`, geometry service, PDF render scheduler/cache, navigation reducer, view-state persistence.

### Steps

1. Implement canonical page coordinate normalization, forward/inverse matrices and tests before drawing UI overlays. Account for native PDF rotations and non-zero boxes once.
2. Implement a pure `reduceReadingState(state, action, manifest)` for all modes, links, setlist boundaries and placeholders. Return requested state plus effects; do not call native render functions from the reducer.
3. Single page, two-page stride one/two, and half-page modes must match DESIGN.md. For half state, use two clipped page instances with independent occurrence IDs and identical viewport scaling policy.
4. Add global persisted divider ratio, draggable without losing current content mapping. A divider drag is not a page turn; disable it under default performance lock.
5. Implement Whole/Fit width/Fit height/Stretch. Persist last fit. Zoom/pan is transient view state unless explicitly part of remembered library position; never edit the crop when zooming.
6. Add central gesture dispatcher: controls/panels first, active placement/annotation next, valid link target next, then page zones. Ignore synthetic click after a drag or consumed long press.
7. Implement request generations for rendering. Keep old usable image until new render is ready. Drop stale results, free discarded images and never paint an old version after switching.
8. Implement a byte-budgeted LRU for rendered tiles/documents. Initial working target: 96 MiB decoded-render budget on phone, 192 MiB on tablet, reduced on memory pressure. These are engineering budgets to profile, not promises about total process RSS.
9. Prefetch adjacent states and likely visible link targets within budget. Do not preload an entire PDF or setlist. During a protected setlist session, only that scope can decrypt protected prefetch targets.
10. Implement independent toolbar states and overlay-stable viewport. Do not copy exclusive-toolbar state from the HTML prototype. Add performance-lock capability checks and long-press unlock.
11. Persist library last-left occurrence/half phase after successful navigation, without treating a failed render as a successful page change. If a saved occurrence was removed, show the nearest surviving ordinal from that version and persist the repaired location deliberately.

### Verify / complete

Property tests: next then previous is inverse away from explicit jumps/boundaries; no invalid indices; no cross-score half/spread; target links survive reorder. Golden geometry fixtures for all transforms. Physical tests at portrait/landscape with differently sized PDFs and 100 repeated turns. Toolbars do not move the first stroke coordinate. Render failure never blanks a usable current page or selects a different version.

## M08 — input, pen, highlighter and erasers

### Steps

1. Build input state machine with Idle, Drawing, Erasing, Selecting, Transforming, Panning, Pinching, PlacingLink, DraggingDivider. Each gesture has one owner until ended/cancelled.
2. S Pen down reveals both toolbars and begins ink on the same sample. Pressure support is optional per tool; constant-width mode ignores pressure. Finger input on tablet navigates/pans rather than draws.
3. Add phone annotation entry/checkmark. While finger drawing is enabled, finger taps are not page turns. Switching tool or route cancels/commits a gesture according to explicit command boundaries.
4. Verify barrel-button hold temporarily chooses eraser, including release mid-contact and hover-before-contact. No permanent switch caused by a lost button-up; reset transient state on cancellation/device loss.
5. Collect actual samples, construct smooth visual strokes, render live feedback without blocking on DB writes, and persist completed samples/outlines. Use bounded buffers and resampling that does not remove pressure discontinuities.
6. Pen/highlighter options include width/color/pressure mode and 0–7 favorites for each. Saving a preset does not mutate existing objects.
7. Area eraser computes geometric subtraction and creates surviving stroke fragments in one command. Stroke eraser deletes whole touched strokes. Both operate only on the active unlocked visible ink layer and ignore text/stamps/snippets.
8. Retain dirty completed strokes on save error; block additional edits and surface retry. Do not lose them by rebuilding the widget tree.

### Verify / complete

Real S Pen tests for first-contact stroke, fast strokes, loops, slow pressure curves, barrel hold, palm contact, orientation change/cancel and long-session latency. Geometry tests erase a stroke into multiple fragments with preserved interpolation. Undo restores exact original. Phone drawing cannot accidentally change pages. Writing under performance lock cannot mutate content unless the corresponding permission was enabled.

## M09 — selection, layers, text, stamps and history

### Steps

1. Implement lasso/rectangle intersections and whole-stroke/partial-area choice on canonical geometry. Text/stamp objects always stay whole.
2. Transform selected objects through one group matrix; commit at gesture end. Preview state never rewrites source object geometry repeatedly during each frame.
3. Implement copy/cut/paste/recolor/delete. Ensure paste generates fresh IDs and owns destination bytes. Cross-scope protected copies use the required confirmation.
4. Implement layer add/rename/order/show/hide/lock/duplicate/delete/opacity; select active layer explicitly. Move/copy selection to existing/new layer without accidental appearance change.
5. Implement appearance-preserving merge with internal opacity groups, not a bitmap flatten. Test overlapping translucent strokes/layers. Layer merging is required even though version merging is excluded.
6. Implement editable text boxes with size/style/color and deterministic layout metrics. Store content and style separately. Render/export use bundled font metrics.
7. Build categorized searchable supported Bravura stamps, whole-object transforms and sizing. Include attribution/licence. Search here concerns the stamp catalogue, not the excluded PDF-text search feature.
8. Complete persisted command history: default 200 per version, configurable, redo truncation after a new committed command, retained asset pins, version-duplication baseline. History is encrypted under the score domain. Cross-owner commands have linked transaction IDs and compatible-cursor checks; undo cannot overwrite later edits in a partner version. Explain dependencies or request partner authentication when necessary.
9. Every tool obeys performance-lock permissions at invocation and commit. UI disabled state alone is insufficient.

### Verify / complete

Boundary selection tests (touching edges, holes, crossing curves), whole text/stamp selection, layer-opacity merge equivalence, restart undo/redo and history trimming. Hidden/locked content remains unchanged. Duplicate version tests include layers/groups/history isolation. No memory growth from retaining selected raster previews after disposal.

## M10 — snippets, crop and page arrangement

### Steps

1. Implement explicit PDF-content selection, separate from ink selection. Rasterize at a bounded high resolution based on target size; use tile assembly for large captures rather than unbounded allocation. Store capture resolution and provenance.
2. Offer printed-only or printed+visible-annotations. Respect layer opacity and visibility. Lasso raster has transparency outside selection.
3. Copy inserts a snippet without a patch. Move/cut produces a source patch plus destination object only once paste commits. Cancel/failure leaves original arrangement intact.
4. Implement Restore snippet for moved objects; Delete for copies. Include cross-page and cross-version/score transfers with two-owner transaction and asset ownership remapping.
5. Crop tool contains crop edges, rotate, straighten, reset, undo/redo, Apply to all. All transforms are non-destructive; crop does not delete hidden annotations. Apply-to-all creates one reversible command with explicit per-page results.
6. Page arrangement loads virtualized thumbnails, supports reorder/remove and adds pages from new PDFs, existing library versions or blank templates. Preserve at least one page. Copy source-version edits independently and rekey protected/unprotected transfers appropriately.
7. Generate plain, single-staff and grand-staff vector pages, current dimensions default and A4/Letter choices. Keep template IDs/versioned parameters in storage for repeatable export.
8. For page removal, show affected references; retain disabled broken references by default or delete per setting. Deleting the destination page of a moved snippet restores its source patch in the same command. Deleting its source page keeps any surviving moved destination and provenance; Restore snippet becomes unavailable with Source page removed until the source occurrence is restored. Preserve tombstone/history dependencies instead of silently discarding surviving music.

### Verify / complete

Compare compositing to known fixtures, including a moved snippet restored after reopen. Kill during cross-score cut and assert both sides old or both committed. Export tests later must prove snippet patches/crops match preview. Reorder duplicate occurrences and ensure links follow correct IDs. Mixed page sizes plus Apply-to-all must not generate out-of-bounds crop geometry.

## M11 — bookmarks, links and rich notes

### Steps

1. Build bookmark list/add/rename/jump/delete tied to page occurrences.
2. Implement link creation as explicit source-placement then destination-placement states, with cancel. Persist neither half-created link as valid. Dot placement uses inverse geometry and current half-page occurrence.
3. Link activation shows full target page, flashes destination three times and returns to the configured normal visibility. Respect reduced-motion accessibility without losing the target cue. Default destination hidden except while link controls are open; setting can keep it visible.
4. Add reference repair choosing a new destination, or delete. Reorder never rewrites IDs to numeric indices. Crop-hidden targets are reported/located rather than pointing to another page.
5. Implement scoped nested notes and versioned rich-text schema with bold/italic/lists. Autosave with visible pending/error state; no executable HTML or remote embedded resources.
6. Enforce encryption for version/shared notes. Moving/copying to Global requires explicit unprotected-copy confirmation.

### Verify / complete

Link source/destination through crop, rotation, zoom/stretch and both half-page phases. Delete/reorder/restore pages and verify references. Repeat jump cancels previous flash sequence cleanly. Notes survive restart, scopes do not leak, and large nested note folders prevent cycles.

## M12 — metronome, tuner, tone and pedals

### Steps

1. Build native audio service with one output owner and bounded realtime buffers. Realtime callbacks must not allocate, lock on DB work, read files or invoke Flutter methods synchronously.
2. Generate/bundle three free click sounds; no copied commercial assets. Schedule by sample position with fractional carry for tempo/subdivision, accent/mute pattern and volume. Apply configuration at safe beat boundaries.
3. Implement 1–300 BPM, tap tempo, 1–16 beats, subdivision 1/2/3/4 and beat cycle Normal → Accented → Muted. Persist per-version musical pattern; global sound/volume.
4. Drive the border pulse from native timestamps; do not use it as the audio clock. Closing popup keeps playback; leaving viewer/background stops it. A setlist score boundary remains inside the viewer: keep a running metronome running and apply the next version's stored musical pattern at the next beat boundary. Leaving the whole viewer stops it.
5. Implement microphone permission and pitch estimator off UI thread, confidence gating, equal-tempered note/cents display, reference A4 400–480 and instrument transposition. Validate with generated tones and instrument samples, not only sine waves.
6. Reference tone pauses listening; tuner pauses audible clicks while visual pulses continue. Closing tuner restores only eligible previously running clicks. A manual stop, background transition or audio-focus loss cancels automatic resumption.
7. Keep audio and auth session lifecycle separate: a protected setlist stays unlocked in background while its metronome is stopped.
8. Keyboard-style Bluetooth pedal input maps configurable key-down events to the navigation reducer. Default left/right arrow mappings, ignore OS auto-repeat by repeat flag, do not use an arbitrary long debounce that swallows intentional taps. Provide remapping without requiring a specific brand SDK.
9. Dispose microphones/streams and release screen policy on lifecycle exit. Do not add network permissions for tuner functionality.

### Verify / complete

Record native sample timestamps under PDF import/render load; verify no cumulative drift over a ten-minute fixture. BPM endpoints, rapid tap/config changes, muted subdivisions, popup closure, backgrounding, microphone refusal and headset disconnect. Compare pitch against known A4/transposition frequencies. Hardware pedal certification remains explicitly unverified until an actual pedal is available; a keyboard test is not evidence for every Bluetooth pedal model.

## M13 — PDF export

### Steps

1. Freeze a committed version snapshot and pin referenced assets/history as needed during export. If dirty edits exist, save successfully or explicitly resolve them first.
2. Export original bytes separately from composed version. Original hash must match imported source bytes after storage decryption.
3. Assemble output pages using original PDF objects where possible; preserve native page text/vector content. Apply shared geometry crop/rotation/straightening, inserted templates/pages, source cover patches and snippet images.
4. Flatten visible app overlay content into PDF page content, with correct text fonts, Bravura, opacity/groups and stroke widths. Unannotated version removes ordinary annotation layers but retains structural snippet edits. Included annotations already flattened inside an explicitly captured snippet remain part of that snippet.
5. Map source occurrence IDs to output page indices before creating links. Omit broken links with a user-visible warning; valid links remain clickable without visible navigation dots.
6. Handle password/source permissions using the qualified engine. Report inability to preserve fidelity as an error with affected pages. Never silently replace all pages with screenshots.
7. Protected export requires scoped authorization plus explicit unprotected-output confirmation. Write via staged private stream to the user-chosen destination; never export into the linked source by default or overwrite it implicitly.
8. Reopen the completed output and validate page count/references before reporting completion; handle interrupted external provider writes without claiming success.

### Verify / complete

Independent-reader renders and structural inspection for vector/text preservation, original hash, all three export modes, visibility/opacity, rotations, mixed sizes, snippets and links. Include password-encrypted original and protected HighScore score as separate cases. No warmth, dimming or tool UI in output. Export failure leaves current score/version untouched.

## M14 — backup and restore

### Steps

1. Define an explicit versioned archive schema. Use streaming ZIP support through the platform or an approved library; no whole-library memory buffer. Include manifest hashes, logical record IDs and encryption envelopes.
2. Pin a consistent committed snapshot; copy all required originals/additional assets, versions, notes/history, folders/labels, setlists/placeholders, trash and settings. Exclude caches/session grants/tokens.
3. Preserve protected ciphertext/envelopes; make unprotected content portable without the original Keystore. Do not accidentally put the vault key alongside ciphertext in the archive.
4. Verify archive structure/hashes after writing. Reject traversal, duplicate entries, unknown mandatory schema features, absurd lengths and truncated data. Separate a structurally valid locked payload from a cryptographically verified unlocked payload in internal results.
5. Restore validation obtains backup credential/recovery as required. Incorrect credentials change nothing. A validated safety backup of current state must exist before replacement or merge publication.
6. Replacement builds a new staged library generation and allows restore settings or retain current settings. Protection envelopes are not ordinary settings; restoration must preserve decryptability. Reauthorize external providers independently.
7. Merge computes logical identity/fingerprint, skips exact matches and retains conflicts as independent named copies. Use a complete mapping table for scores, versions, occurrences, layers, objects, notes, references, history and entries. Preserve duplicate setlist entries.
8. Rewrap imported protected keys into the destination shared vault after authenticating source and destination. If no destination vault exists, run setup; never silently drop protection or retain multiple app-level PINs.
9. Validate referential integrity and actual protected reads in staging. Publish one active-library generation switch. Startup recovery chooses one complete generation; cleanup is resumable.
10. Show restore results: added, skipped, conflict copies, warnings and required source reconnections. Do not expose private content in diagnostic logs.

### Verify / complete

Backup from device A, restore on clean device B with no A Keystore. Verify both replacement settings choices, merge with same IDs/different edits, old-password backup after live password change, recovery-key restore and missing credentials. Crash every stage of restore, including safety-backup publication and active-generation switch. Full storage never destroys the only working library.

## M15 — background work and complete Settings

### Steps

1. Connect source discovery to unique WorkManager requests with persisted checkpoints and no overlapping full scans. Maintain one process-independent source lease so foreground/background workers cannot publish duplicates.
2. Enforce default 15-minute library-entry, hourly active-library and six-hour requested-background checks; adjustable durations and off state. Honor OS minimum periodic duration without busy-loop emulation.
3. Independent Defer while viewing and Defer while performance locked settings default on. Either applicable setting defers optional work. Re-evaluate at safe job boundaries; never cancel a durability-critical publication halfway through.
4. Wi-Fi-only Drive default, mobile-data opt-in, low-battery background deferral, retry/backoff and credential-needed state. Do not poll every second or keep wake locks during idle waiting.
5. Complete every Settings item listed below. Store typed values and migrate them explicitly. Settings changes cannot rewrite existing annotation geometry, change source originals or erase history without a clear action.
6. Add provider reconnection/manual refresh UI and clear last-success/error states. No source marked Removed on a failed scan.
7. Finish toolbar customization and restore-to-default layout. Required escape controls (performance toggle/centre unlock/phone finish-annotation) cannot all be removed in a way that traps the user.
8. Add bundled licence notices and local non-sensitive diagnostic information. No telemetry/hosted reporting feature.

### Settings completeness checklist

- Theme Light/Dark/System; approved palette/font fixed as the selected design.
- Default/last fit and page layout/stride; global half divider ratio.
- Toolbar order/visibility; pen and highlighter favorite counts 0–7 and presets.
- Eraser/selector defaults; persisted undo retention (200 default).
- Performance-lock permitted actions; automatic-work deferral toggles independently.
- Source-check intervals/off, Wi-Fi/mobile policy and low-battery deferral.
- Broken-reference retain-for-repair vs delete; normal return-dot visibility.
- Trash retention manual default or positive days.
- Warmth/dimming controls; viewer Never sleep/System/custom inactivity duration.
- Pedal key mappings.
- Global metronome sound/volume; tuner reference pitch/transposition.
- Protection credential/recovery management without an auto-relock timer.
- Local backup, replacement/merge restore; replacement settings choice.

### Verify / complete

Fake-clock/scheduler tests prove coalescing and gates. Physical observation verifies no idle busy polling and no stalled saves while optional work defers. Changing one setting does not accidentally reset unrelated settings. Starting foreground viewing while a background scan runs respects safe boundaries and rendering priority.

## M16 — release hardening and handoff

### Steps

1. Run the complete acceptance suite. Resolve every failing data-loss/corruption/access-boundary/navigation check before marking release ready.
2. Exercise a 500-score corpus; include vector, scanned, mixed, encrypted, malformed and oversized fixtures. Include 100-page and 300-page scores and at least one large scan file. These are test loads, not an advertised universal maximum.
3. Run 100 consecutive repetitions of each critical workflow class: import, completed-stroke save/reopen, version switch, setlist navigation, protected session entry/exit, backup/restore. Fault-injection tests are separate from successful repetition tests.
4. Conduct at least a two-hour offline rehearsal on both physical devices with half-page mode, links, frequent version changes, annotation, metronome, background/resume and orientation changes. Measure memory and battery relative to a viewer-only baseline.
5. Verify release-mode behavior, not just debug. Profile pen latency with slow-motion recording if needed; ensure logging is not distorting measurements.
6. Audit native handle disposal and asset garbage collection after repeated open/close, restore, undo trim and protection conversion. Never run destructive GC against real user data before fixture tests pass.
7. Build a signed personal APK only after signing material is provided through the user's chosen local setup. Do not invent signing passwords, commit keys or publish to a store. Keep debug installation usable for testing meanwhile.
8. Write `docs/RELEASE_VALIDATION.md` with exact versions, devices/OS/build types, commands, fixture corpus, measurements, unsupported cases and remaining hardware gaps. No certification of an untested pedal or iPad.
9. Hand the user the build and instructions, with no git commit/push. Flag any remaining release blockers honestly; do not silently move a first-release feature to a later release.

### Initial engineering budgets

These are qualification targets, to be measured and refined with explicit evidence; they are not guarantees for every PDF.

| Measure | Initial target |
| --- | --- |
| UI response | Controls respond within 100 ms in normal qualified fixtures |
| Cached page turn | Correct next state visible within 150 ms, excluding optional short animation |
| Ink feedback | p95 input-to-visible feedback under 50 ms on S Pen test device |
| Completed small edit save | p95 durable acknowledgement within 250 ms with healthy local storage |
| Frame work | No routine main-thread task above 16 ms at 60 Hz; no synchronous PDF/I/O/crypto work on UI thread |
| Render cache | Enforced byte budget; shrinks on memory pressure |
| Sustained rehearsal | No crash, data loss, stuck input or steadily unbounded memory growth |
| Audio | No cumulative sample-clock drift; actual audio jitter/underruns measured under load |
| Recovery | Every injected boundary returns an old/new complete committed state or an actionable recoverable error |

If a target fails, profile and fix the bottleneck first. Do not degrade fidelity or drop features to meet a number. Document cold-render outliers separately; keeping the current image visible is mandatory even when a pathological page takes longer.

## 4. Commands and verification workflow

The following commands are planned commands after the Flutter scaffold exists; they have not been run against an app in this planning task.

```sh
flutter --version
flutter doctor -v
flutter pub get
# Only after adding the qualified code-generation dependencies:
dart run build_runner build --delete-conflicting-outputs
dart format --output=none --set-exit-if-changed lib test integration_test
flutter analyze
flutter test
flutter test integration_test/offline_rehearsal_test.dart -d <device-id>
flutter test integration_test/protected_sessions_test.dart -d <device-id>
flutter test integration_test/crash_recovery_test.dart -d <device-id>
flutter test integration_test/library_scale_test.dart -d <device-id>
flutter build apk --debug
flutter build apk --release
```

Do not pass placeholder `<device-id>` literally. Obtain devices with `flutter devices`/`adb devices`. A successful APK build is not evidence that physical interaction tests passed. The release command requires the user's signing setup for a distributable personal release; do not label a debug-signed build as a finished signed release.

Use targeted test files during feature work; run the full suite at milestone joins and before release. Test generated database migrations against stored old schema fixtures. Test native code with platform unit/instrumentation tests as well as Dart integration tests. Use adb force-stop/process-kill injection on disposable test libraries, not the user's real data.

## 5. Failure handling implementation map

| Failure | Required result |
| --- | --- |
| PDF password wrong | Retry/cancel specific import; no incomplete score |
| Source permission revoked | Source unavailable/reconnect; imported library untouched |
| Drive offline/quota/auth error | Bounded retry or needs-auth state; local reader unaffected |
| Disk full during edit | Dirty state retained; saved state readable; further edits blocked |
| Native render failure | Keep usable previous image, report/retry; no wrong-page fallback |
| Asset tag/hash mismatch | Corruption error, no unauthenticated decode/no empty-success substitution |
| Wrong protection credential | No grant, no data mutation |
| Process killed in session | Grants gone, committed data intact, authenticate on next protected entry |
| App merely backgrounded | Existing auth grant retained; audio stopped; optional work follows settings |
| Broken link | Disabled, repair/delete; never target by recycled numeric page index |
| Interrupted backup/export | Partial output not reported as complete; original library intact |
| Interrupted restore | Recover one complete generation; retain safety snapshot |
| Unavailable native capability | Stop dependent milestone with evidence; ask before changing behavior |

## 6. Completion definition for the implementing model

The app is complete only when all M00–M16 criteria and acceptance-test IDs are satisfied or an explicitly approved scope change supersedes them. “Compiles,” “looks like the mockup,” “unit tests pass,” and “works on emulator” are insufficient alone.

The final implementation handoff must contain:

- Built application artifact and repeatable build instructions.
- Exact dependency/toolchain/licence ledger and user-owned signing instructions.
- Completed feature-to-test traceability with observed physical results.
- Backup/restore and recovery-key instructions that match the actual app.
- Known limitations stated precisely, including any untested pedal models and lack of iPad delivery.
- No commits performed by the agent, no source PDFs modified and no paid service activated.
