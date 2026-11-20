# HighScore — behavior and architecture specification

Status: implementation handoff, 2026-11-20. Product decisions incorporate the user's final answers. Engineering choices below are implementation prescriptions to validate, not claims that packages or hardware have passed testing.

Read with [decision record](DESIGN_DECISIONS.md), [implementation plan](IMPLEMENTATION_PLAN.md), [acceptance tests](ACCEPTANCE_TESTS.md), and [reliability requirements](RELIABILITY_REQUIREMENTS.md). Later explicit user decisions take precedence over all files. This specification supersedes conflicting wording in the original SPEC.md and historical mockups.

## 1. Product boundaries

HighScore is an offline-first Android PDF sheet-music library and performance reader, equally usable on phones and tablets. Flutter/Dart holds the portable domain and interface. Platform adapters isolate Android functionality so an iPad implementation can follow later. iPad delivery is not part of this release.

Every retained feature is required in release one. Milestones establish build order, not a reduced MVP. No HighScore account, hosted storage, automatic cross-device library sync, OCR, PDF full-text search, alternative temperaments, version merging, paid dependencies or required paid services. Source files are never modified. Google authorization is only for optional Drive import, not a HighScore account.

Reliability wins over animation polish and coding speed. Never advertise an absolute no-failure guarantee. All supported workflows require automated recovery tests and physical validation on Galaxy Tab S8+ 5G/S Pen and Pixel 6a.

## 2. Interface contract

### 2.1 Visual system

| Token | Value / behavior |
| --- | --- |
| Typeface | Bundled Liberation Sans, regular and bold; no runtime font download |
| Accent | `#2F7434` |
| Navy | `#18243A` |
| Library add | Accent background; `#D9F2D7` plus glyph |
| Light surfaces | White panels, very light neutral library background |
| Dark surfaces | Navy panels and deeper neutral background |
| Selection | Pale green in light mode, deep green in dark mode; shape/pressed state as well as color |
| Themes | Light, Dark, System |
| Score colors | Original PDF colors in either theme; warmth/dimming is a separate viewing effect |
| Tool buttons | Icons with semantic labels, tooltips and focus indicators; 48dp minimum touch regions |

Use the current [review file](mockups/highscore-review-v2.html) for visual proportions, not its demonstration logic. It is not app code. Its superseded mutually exclusive toolbar behavior must not be copied. The user requested no further mockups.

Use available width, not device model, for layout: compact widths below 600 logical pixels use bottom navigation and sheets; wider layouts use sidebar navigation and side panels. Stylus behavior is capability-based, not inferred from the width breakpoint. Keep major tools directly visible; compact layouts wrap toolbar rows without shrinking touch targets. The user can customize tool visibility/order; do not move nearly all default tools into an overflow menu.

Default general toolbar actions: library/back, versions, performance lock, display, bookmarks, metronome, tuner/reference tone, links, notes, crop, page arrangement, eye comfort, export, viewer settings. Crop includes rotate and straighten. No separate rotate action and no Hide action. Compact/finger-only devices also offer annotation entry.

Default annotation tools: pen, highlighter, eraser, selection, text, stamps, layers, PDF-content selection, undo, redo and favorite presets. Re-selecting pen/highlighter opens width/color/pressure options; re-selecting eraser opens area/stroke and size; re-selecting selection opens lasso/rectangle and whole/partial stroke choices. The prototype's separate subtype icons do not override the original specified tool-option behavior. Toolbar customization may expose subtype shortcuts, but do not introduce additional editing semantics.

### 2.2 Toolbar visibility and input

Maintain separate `generalVisible`, `annotationVisible`, `fingerAnnotationEnabled` states. Initially both toolbars are hidden.

| Input | Effect |
| --- | --- |
| Stylus contact on score | Show both toolbars, start the actual stroke on that same contact if editing is allowed |
| Centre finger tap while annotation toolbar is visible | Hide annotation toolbar, show general toolbar |
| Centre finger tap with only general toolbar visible | Hide general toolbar |
| Centre finger tap with both hidden | Show general toolbar |
| Phone annotation icon | Show both toolbars and enable finger drawing |
| Phone annotation checkmark | Disable finger drawing, hide annotation toolbar, retain general toolbar |
| Left/right finger tap | Previous/next reading state, unless finger drawing or a placement tool consumes it |
| Finger drag/pinch | Pan/zoom when permitted; never reinterpret a completed drag as a page tap |
| S Pen barrel button held | Temporarily erase; restore previous drawing tool on release |

Toolbars overlay a stable score viewport; opening them during the first pen stroke must not move the page underneath the nib. Do not include hidden controls in hit-testing. Panels may reduce the viewport only outside an active stroke; defer relayout until the gesture ends. Finger palm events during a stylus stroke must not turn pages. Use actual pointer kind, button state and cancellation, not timing guesses alone.

Use equal left/centre/right horizontal tap thirds as the initial implementation geometry; exclude toolbar/panel regions and give actionable links priority over page taps. Tap thresholds follow platform gesture tolerances. Long-press centre for one second disables performance lock, consumes that gesture, and shows a temporary confirmation. No persistent small lock badge. A press cancelled by movement does nothing.

### 2.3 Performance lock

Performance lock prevents accidental actions; it is unrelated to encrypted score access. Enable/disable using the general-toolbar padlock. Default allowed: page turns, valid links, bookmarks and metronome. Default blocked: version changes, drawing, crop/arrangement, divider changes, pan/zoom and other editing. Settings allow individual action groups to be enabled. Authentication can never be bypassed through this configuration.

Toolbar reveal, padlock toggle and centre hold-to-unlock always remain accessible. Tool visibility is independent of permission: stylus contact can reveal toolbars while editing controls remain disabled. Cancel any active mutation before enabling the lock. Re-check permissions at command execution, not only when rendering buttons.

## 3. Library, import and setlists

### 3.1 Library

A score has an immutable imported original, public title/folder/labels, one or more versions, and independent library reading position. All scores is a virtual collection; a score has one real library folder. Folders can nest. Prevent folder cycles.

Support title A–Z/Z–A, last opened, and last imported sorting. Use stable ID as a tie-breaker. Labels are filters, not a sort. Include labels using Match any/Match all, then subtract excluded labels. Search only score titles and label names. Combine search, folder and label conditions. Recents is the same score presentation ordered by last opened.

Per-score menu: rename, labels, move, add to setlist, protect/unprotect, trash. Long-press multi-selection: move, add to setlist, trash. Library + opens a centred modal for PDF import, folder import, local linked folder, Drive PDF import and Drive linked folder. Native file picking follows the modal selection.

A folder import creates a library folder. Subfolder choices are preserve hierarchy, flatten, or exclude. Library names and locations are independent after import; source rename/move never renames library items.

### 3.2 Import semantics

An import streams bytes into app-owned staging, validates the PDF, records its digest and only then publishes a complete score. Original PDF byte identity is preserved even if its storage container is encrypted. A PDF password is separate from the shared HighScore protection credential. Support ordinary password-encrypted PDFs, optional Remember on this device, and per-file errors for wrong password, unreadable PDF and unsupported certificate/DRM protection. Prompted imports can be cancelled independently inside a batch; unattended imports needing a password wait for user input without opening a background dialog.

Content-identical PDFs are duplicates even under different filenames. Manual import offers Skip or separate score; automated import skips and reports. Separate scores must not share editable state. Do not merge annotations based on matching bytes.

Linked source discovery imports new provider identities only. Subsequent edits to an already imported source do not change its score. Record ignored revisions so changes do not repeatedly appear as new. Use local provider authority/tree/document ID or Drive account/file ID, not pathname. Rename does not reimport. Source deletion retains a removed-from-source badge on the library copy. Losing permission, going offline or receiving an incomplete directory listing is not proof of deletion.

Maintain a source-to-library-folder mapping for future new imports. If the mapped library folder is deleted, new imports go to the surviving nearest mapped ancestor (root if none); never recreate a user-deleted folder or move existing scores. A trashed/permanently deleted automatically imported source identity is tombstoned against immediate reimport; explicit manual import is still possible.

### 3.3 Scheduling

Defaults: check on library entry if 15 minutes have elapsed; hourly while library stays open; request background work every six hours; low-battery background deferral; Drive Wi-Fi only. Each automatic interval can be disabled or configured as a positive duration. Validate background intervals against platform limits instead of promising exact execution times.

Two independent default-on settings defer automatic source checks/imports while viewing scores and while performance lock is active. Either applicable setting can defer optional work. Saves, visible-page rendering and essential recovery never defer. Manual refresh is available. Network policy still applies to Drive. Coalesce duplicate jobs, checkpoint traversal, back off on transient errors, and cap concurrency. A full-library scan must not run every time a widget rebuilds.

### 3.4 Setlists

Each entry has its own stable ID and is either a score/version reference or a text placeholder. Duplicate entries are allowed, including identical score/version pairs. Create/edit/reorder/remove entries, choose versions, nest setlist folders, duplicate a setlist and merge selected setlists. Merge creates a new setlist by concatenating in user-chosen source order; originals and duplicates are retained. Removing an entry never deletes its score.

A placeholder is a blank viewing page with its assigned title. Directly selecting an entry starts at its first page. Forward beyond the last page enters the next entry; backward from the first page enters the previous entry's last page. No wrap at either end. Unavailable entries are shown explicitly; no automatic skipping or substitution.

A protected-containing setlist requires authentication on entry to the setlist, before presenting its contents/viewing session. One shared credential unlocks its protected scores until exiting that setlist. Repeated entries and version changes do not reprompt. Editing that same setlist or opening its viewer panels does not end the session. Leaving for the library, another setlist or an unrelated direct score does end it. If protection is introduced into a previously unprotected open setlist, acquire authentication before granting protected access.

### 3.5 Trash and references

Trash retention defaults to indefinite/manual; Settings accepts a positive number of days for automatic purge. Trash preserves versions, notes and references. Setlist entries remain unavailable with Restore/Remove actions. Restore reinstates identity and references. Folder deletion offers move contents to parent or trash the subtree; make the descendant count clear.

Permanent purge removes owned content only after reference/undo/staging/backup dependency checks. Retained setlist references become explicit missing-score entries. Deleting a version requires a surviving replacement for each affected setlist reference; never delete the final version. The version picker enters selection on long-press, exposes Duplicate for one selected version and Delete for one or more, and exits selection when none remain. No version merge.

## 4. Versions, reading and geometry

### 4.1 Ownership

| Shared across a score | Independent per version | Independent per entry/session |
| --- | --- | --- |
| Title, folder, labels, protection policy, imported original, shared notes | Page occurrences/order, crop/rotation, annotation layers/objects, snippets, bookmarks, links, version notes, tempo/meter, undo history | Setlist entry's version selection; viewer session position |

Duplicating a version copies editable state and remaps internal IDs/references; immutable bytes may be shared within that score. A duplicated version starts its own empty undo timeline over its copied baseline. Editing one version affects all entries using it. Library last-selected version/position is separate from setlist entry choices. Keep a position per library version so switching versions does not overwrite another version's reading position; only direct library viewing updates that position.

### 4.2 Page identities and display state

A `PageOccurrenceId` identifies an occurrence in a version, not a PDF page number. Repeated copies of one source page have distinct identities. A link/bookmark attaches to an occurrence and canonical point. Reordering changes ordinal position only. Removing a destination warns about affected references. Default retains a disabled Destination removed reference with repair/delete; the approved setting instead deletes affected references. Undo restores exact identities. Each version retains at least one page.

Define `ReadingState(entryId?, versionId, mode, anchorOccurrenceId, halfPhase, fit, zoom, pan)`. Never implement navigation by incrementing a PDF widget page number in several places.

| Mode | Forward/back behavior |
| --- | --- |
| Single | One occurrence at a time |
| Two-page, stride 2 | `[0,1] → [2,3]`; backward reverses spread anchors |
| Two-page, stride 1 | `[0,1] → [1,2]`; backward moves one anchor |
| Half | Full page N → next page top/current page bottom → full page N+1; reverse exactly |

Two-page spreads never contain different scores. Pad unmatched final spreads with blank space. For backward entry across a boundary, choose the last spread containing the last page. Half mode at the final full page advances to the next score's first full page; never compose two scores into a half state. Links/bookmarks reset half phase to a full target page and establish the target as the spread anchor, so it is visibly present. Subsequent previous/next follows the resulting reading state.

The half divider is one installation-wide percentage, initially 50%, saved until changed. Clamp only enough to retain usable touch/render regions (initial engineering bounds 5–95%); persist the ratio, not pixels. Keep the page content at the same scale in both halves; clip by the divider. Never scale each half separately. Link hit-testing and annotations route to the occurrence visible in the touched half.

### 4.3 Transform contract

Store canonical page points in PDF-point units after normalizing MediaBox/CropBox origin and intrinsic PDF rotation once on import. Retain the conversion back to native PDF coordinates. Use doubles, not device pixels. Persist user crop as a canonical polygon/rect and rotation/straighten as transforms.

One shared geometry service supplies both forward and inverse matrices:

`canonical page → user rotation/straighten → crop clipping/output bounds → fit → zoom/pan → viewport`

Apply half-view clipping in viewport space. Retain the underlying page-to-viewport transform per visible occurrence. Hit testing inverts the correct transform; export uses the same page-space geometry without device fit/zoom. Reject non-finite values and singular transforms.

Fit modes: Whole page (initial default, proportional), Fit width (vertical pan), Fit height (horizontal pan), Stretch (independent x/y scale). Remember the last selected fit globally. Two-page fit operates on the spread bounds while preserving each page's intrinsic proportions except in Stretch. Rotation/straightening/crop do not rewrite source bytes or delete off-crop annotations. Crop reset resets the selected page's user adjustments; Apply to all applies equivalent normalized crop and angles to pages with different sizes, with a preview and one undoable command.

Keep the current rendered page visible until its replacement is ready. A render error retains the usable image and provides Retry; do not show a different score or silently advance the committed reading position. Use a brief page transition only when both images exist; reduced-motion mode removes animation. Rapid commands are serialized and preserve the requested final state.

## 5. Editing model

### 5.1 Layers and objects

Layers span the version and have name, order, visible, locked and opacity fields. Only the active unlocked visible layer receives drawing/erasing/selection. Selecting a hidden/locked layer must not silently unhide/unlock it. PDF content is a separate explicit selection target, not an ink layer.

Editable object types: pressure/constant-width stroke, translucent highlight stroke, rich text object, Bravura stamp, raster snippet. Text/stamps remain whole under partial selection. Store stable object IDs, local-to-page transform and bounding boxes. Layer opacity multiplies object opacity once; avoid repeatedly darkening a continuous highlighter stroke at self-overlaps.

Pen/highlighter each support 0–7 favorite presets, size/color, variable or constant width. Stylus samples retain position, pressure and order. Do not save predicted samples as actual ink. Area erasing clips stroke geometry, producing editable remaining fragments; stroke erasing removes touched strokes. Both leave text/stamps/snippets untouched. Do not erase the PDF bitmap.

Selection supports lasso/rectangle and whole-stroke/partial-area semantics; move, scale, rotate, copy, cut, delete, recolor and move/copy to existing/new layer. For partial strokes, split at geometric boundaries and preserve width/color/pressure interpolation. Hidden/locked layers are not selected. Text and stamps are selected as whole objects when intersected.

Layer merge must preserve the pre-merge appearance and editable objects. Preserve constituent group opacity as nested groups when simple per-object multiplication would change overlaps. No destructive rasterization of merged ink layers. Require selected layers to be unlocked; merge at their stacking position and make one undo command. Group structure is an internal implementation detail, not another product feature.

### 5.2 Printed snippets

Capture a high-resolution raster from the selected canonical PDF region (rectangle or lasso), with transparent outside-lasso pixels. Include either printed content only or printed content plus visible annotations as selected by the user. Included annotations are flattened in the snippet; their originals are not silently deleted.

A copied snippet has an independent raster/transform. A moved snippet also has a source patch and stable source occurrence/region. Rendering order is base PDF, source cover patches, moved/copied printed snippets, then ordinary annotation layers. A white/appropriate blank patch is a layout aid, not secure redaction; original bytes remain. Preserve source provenance even after a snippet is moved to another page. Cross-score transfer owns destination bytes and never depends on another score remaining unlocked/existing.

The action for a moved snippet is Restore snippet: atomically remove its moved instance and source patch. Copied snippets use Delete. A cut is not committed until paste succeeds; cancelling or failing paste leaves the source intact. For cross-score cut, commit both sides together or neither, using the operation coordinator. Protect-to-unprotected copying requires authentication and the approved unprotected-copy confirmation. Keep clipboard data in an app-owned memory service with source access scope; do not leak score images into the system clipboard implicitly.

If deleting a page removes a moved snippet's destination, remove that moved instance and restore its source patch in the same page-removal command. If deleting the source occurrence while its moved destination survives elsewhere, keep the destination and its provenance; Restore snippet is unavailable with “Source page removed” until that occurrence is restored. Never silently delete the surviving music. Tombstone source occurrence identity until no live snippet/history reference needs it.

### 5.3 Text, notes and templates

Text boxes offer size, style and color; persist text content and layout parameters, not just a raster. Bundle the interface font and supported Bravura symbols with licence notices. The stamp catalogue groups accidentals, dynamics, articulations, navigation, fingering, notes/rests and other supported Bravura notation; no promise that every conceivable notation exists.

Notes have nested folders and explicit scope: version, all versions of score, global. Store a versioned structured rich-text schema for bold/italic/lists, not unrestricted HTML. Score scopes inherit protection. Copy/move to Global is an unprotected destination and uses the disclosure confirmation.

Page arrangement supports reorder/delete, import from a new PDF, copy pages from an existing library version, and generated plain/staff-paper pages. Default dimensions match current page, alternatives A4/Letter. Build staff templates as vector lines with fixed template IDs; template variants are plain, single-staff and paired grand-staff layouts. Copying from a library version copies its selected rendered structure and editable owned content with remapped IDs; source remains independent. If its selected pages omit a link destination, retain a broken reference according to the configured policy. Imported external PDFs are stored as additional immutable assets, not merged into the original imported file.

### 5.4 Undo and durability

Every editing command includes ID, affected version(s), expected revision, forward/inverse data and immutable asset references. History is separate per version, survives restarts and defaults to 200 actions, configurable. Cross-version/score operations use linked history records with one transaction ID. Undo/redo applies the linked operation atomically only when its dependent records are at compatible history cursors; otherwise explain which later edit must be undone first. Never overwrite subsequent edits in another version to force an undo. A protected partner requires an appropriate authenticated capability before a linked undo can proceed. New edits after undo discard redo history only after successful commit. No app-global Ctrl-Z that accidentally edits another score.

Commit a completed stroke on pen-up, a transform on gesture completion, and a text/notes change after a short idle debounce and before deliberate navigation. UI can render optimistic feedback, but a saved indicator acknowledges only durable commit. Flush queued completed commands before leaving. Process kill may discard an unfinished stroke/uncommitted keystrokes; it must not discard previously acknowledged edits. When a commit fails, retain dirty buffers, block new mutations and offer Retry or recovery export while saved reading remains possible.

## 6. Audio and display tools

Metronome: 1–300 BPM, 1–16 beats, tap tempo, individually Normal/Accented/Muted beats, subdivisions 1/2/3/4, three bundled/generated sounds, volume and audio/visual/both. Initial first beat accented. Visual pulse is around the screen border, not only in its popup. Persist tempo, meter, accent pattern/subdivision per version; sound and volume global. Closing popup/page turns does not stop it. On a setlist score/version transition, a running metronome remains running and adopts the newly selected version's stored pattern at the next beat boundary. Leaving viewer/backgrounding stops it; returning does not unexpectedly restart it. Protected session retention on background is independent from stopping audio.

Use a native audio sample clock, not a Dart timer to trigger each click. Accumulate fractional samples to prevent tempo drift. Tempo changes occur at the next beat boundary; rebuild future schedule, not an already played beat. Muted beats and their subdivisions remain silent. Subdivision clicks are quieter than main beats and do not invent extra accents. Tap tempo averages recent taps, clears stale intervals and clamps to supported range.

Tuner: microphone permission only on use, nearest chromatic note/cents/confidence, no misleading stable reading in silence/noise; equal temperament, A4 440 default, 400–480 selectable. Instrument transposition changes displayed/written note naming relative to concert pitch, not the PDF or captured signal. A selectable reference tone uses the corresponding concert frequency. Use signed semitone transposition with concert C = 0; B-flat instrument written C sounds -2 semitones. Store the instrument/reference setting globally.

Microphone listening pauses audible metronome output; visual pulses continue. Closing tuner resumes clicks only if they were running and the user has not stopped them, the viewer remains foreground, and audio focus permits. Reference tone pauses microphone listening; stopping it can resume listening while tuner stays open. Close releases microphone/tone. Handle interruption, permission revocation and route disconnection.

Warmth/dimming is a non-destructive viewer overlay, excluded from export. Screen-awake Settings: Never sleep (viewer default), System timeout, or custom inactivity duration; elsewhere follow system. Release awake requests on background and viewer exit.

## 7. Encryption and access sessions

### 7.1 Scope

Public catalogue fields are score title, folder and labels, plus minimal opaque reference/status fields needed to display lists. Content payloads include original PDF, version names/details, page content, annotations, snippets, notes, history, thumbnails and remembered PDF passwords. Protected payloads must be encrypted on disk, including staging and database logs. Locked thumbnails display only a lock placeholder.

A direct-score grant covers that score and its versions until direct viewing ends. A setlist grant covers the protected scores referenced by that setlist until that setlist closes. Store grants in memory, tied to route/session IDs; never persist `isUnlocked=true` or plaintext session keys. Background and screen-off preserve a live grant, with no automatic relock timer. Process death/reboot destroys in-memory grants; the next entry authenticates again. This is a technical consequence of key lifetime, not a background timeout.

Code that accesses protected bytes requires a capability checked by the repository and asset reader. Hiding a UI route is insufficient. Library metadata remains browseable without a grant. Short protected operations outside viewing (backup verification, protection changes, exports) use an explicitly authenticated operation-scoped capability, not a global bypass.

### 7.2 Native crypto design

Use the already approved Android SDK facilities through a `CryptoPort`, with a future iOS adapter. Do not implement cipher/KDF primitives in Dart. Planned primitives are AES-256-GCM authenticated encryption, PBKDF2-HMAC-SHA256 credential derivation, SecureRandom, and Android Keystore wrapping. This avoids requiring an additional unapproved crypto library. Validate with known vectors and independent test tooling before committing the file format. Android documents the recommended primitive families in [Cryptography](https://developer.android.com/privacy-and-security/cryptography).

Generate a random 256-bit vault master key. Wrap it independently with (a) the credential-derived key and (b) a randomly generated 256-bit recovery key. Store salts, KDF version/parameters and encrypted envelopes, never the credential/recovery key in plaintext. Use at least 600,000 PBKDF2-HMAC-SHA256 iterations as the starting floor, benchmark off the UI thread on both devices, and record the actual parameters. This floor follows [OWASP's PBKDF2 guidance](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html); it does not make a short numeric PIN equivalent to a strong password. Support PIN or password without silently altering/trimming the input; confirm it twice and give clear strength guidance. Use local retry backoff without wiping data.

Each score has a random data key; each large immutable asset has a fresh random content key wrapped under its score key. Protected score keys are wrapped under the vault key only. Device-accessible keys for unprotected content are wrapped by Android Keystore, which is not a substitute for the user's protected-score credential. Keep this distinction in tests. See [Keystore](https://developer.android.com/privacy-and-security/keystore).

Encrypt small records with fresh IVs and authenticated context `(schema, ownerScoreId, objectId, recordRevision, recordType)`. Bind bytes to their owner to prevent ciphertext swapping. For random-access PDF assets, use independently authenticated fixed-size chunks (initial 1 MiB) with a new content key for every new asset generation. Use a unique 96-bit IV per chunk: random 64-bit file prefix + unsigned 32-bit chunk index. Never edit a chunk in place using the same key/IV; rewrite to a new asset generation. Authenticate asset ID, generation, total length, chunk index and length as additional data. Authenticate the manifest itself; verify every chunk before exposing plaintext. Bound indices and lengths before allocation. Keep decoder callbacks bounded and cancelable.

Recovery setup displays an exportable/printable encoded random key with checksum; require confirmation that it was saved outside the app before enabling protection. A recovery reset unwraps the vault key and creates a new credential envelope atomically; it does not decrypt/re-encrypt every score. Changing a password does not invalidate old offline backups: their original credential/recovery envelope remains necessary. A new recovery key replaces only the live envelope, with explicit explanation about old backups.

### 7.3 Leakage, conversion and portability

Store score content in encrypted payloads from the first import even while unprotected; unprotected keys are device-accessible. Protecting a score rotates its data key and copies/re-encrypts its complete dependency closure into a new generation before publication, removing obsolete device-accessible wrappers. Do not share assets across protection domains. Public metadata remains the explicit exception.

Purge live old thumbnails, plaintext working buffers and obsolete access wrappers after successful conversion; recover interrupted conversions safely. Encrypt undo payloads and operation journal content. Never use decrypted temporary PDFs on shared storage. Use PDF custom-read callbacks/in-memory bounded data; if the engine cannot support this reliably, stop the protection milestone rather than silently introducing plaintext temp files. Do not claim forensic erasure of old flash blocks, source PDFs or previously exported copies.

No automatic cloud backup of app-private content/keys; configure Android backup/data extraction rules accordingly. User-requested local backups are the supported portability path. Logs contain operation IDs/error classes, not note text, passwords, recovery keys, plaintext PDF bytes or OAuth tokens. Normal plaintext rendering in memory during an authenticated session is expected; purge session-owned decoded buffers and render caches on scope exit. Do not claim guaranteed zeroization of every VM/GPU copy.

## 8. Backup, restore and export

### 8.1 Backup format

Use a versioned archive with a manifest, stable logical IDs, record versions, hashes and immutable payload entries. Bound archive sizes, entry counts and decoded lengths; reject traversal paths, duplicates, symlinks and invalid owner references. Include originals/additional PDFs, all versions/history/layers/snippets/notes, labels/folders, setlists/placeholders, trash and settings. Exclude derived caches, transient grants, raw device keys and Drive tokens.

Protected records remain encrypted with portable key envelopes. A normal backup may copy locked ciphertext without unlocking; verifying protected semantic content or restoring it needs its credential/recovery key. Unprotected records are portable and must not depend on the original Keystore; wrap their portable backup content keys in the archive or store those records plainly. Do not describe the entire backup as password-protected: only protected score payloads have that guarantee.

Take a logical snapshot at a committed library generation and pin referenced assets until archive finalization. Verify complete archive structure/hashes before reporting success. A half-written external file is never reported as a usable backup.

### 8.2 Restore

Validate before changing the library. Create a validated local safety backup of current state; if storage is insufficient, stop with an actionable error. Restore into a staged generation, rebuild/check relationships, then atomically switch the active generation. Recover after a crash to the old complete generation or the new complete generation, never a mixture.

Replacement offers restore app settings or keep current app settings. Credentials are key material, not ordinary settings: validate incoming envelopes explicitly. A replacement restores the backup's protection domain; retaining theme/tool settings must not accidentally retain incompatible key envelopes. Reauthorize Drive and local folder permissions when unavailable on the destination device.

Merge keeps existing settings. Match identity plus canonical content fingerprint, not PDF digest alone. Identical records skip; same identity with different content becomes a separately named independent copy with remapped IDs. Remap entries, versions, page occurrences, links, layers, history and snippet source pairs coherently. Unlock incoming protected content with the backup credential/recovery key, then wrap imported keys under the destination's shared vault. If no destination vault exists, initialize one through the approved setup flow. Do not retain two user-facing protection passwords after merge.

### 8.3 PDF export

Export modes: current version with visible annotations; current version without ordinary annotation layers; untouched imported original. Version exports retain page order/crop/rotation/straightening, inserted pages and printed-content snippet edits. Hidden layers, UI dots, warmth/dimming and tool chrome are excluded. Original export is byte-identical after storage decryption and retains any original PDF encryption.

Build into a separate PDF. Preserve imported PDF objects/vector music/selectable text where supported; flatten app overlays into page content, not necessarily into one page bitmap. Embed/subset supported text fonts and Bravura. Snippets remain their approved raster form. Preserve valid app navigation as invisible clickable link annotations; warn and omit broken destinations. Never make a broken destination point at the same numeric index on a different page.

Protected export requires authentication and confirmation that the exported copy is outside HighScore protection. An untouched original may still retain its own original PDF password; do not claim that HighScore removes it. Remembered source PDF passwords must never be emitted in metadata. A fidelity limitation is a visible failure/report, not permission for silent full-page rasterization. PDFium capability qualification must include page import, clipping/transforms, object insertion and link creation; add a narrowly scoped native adapter only where exposed bindings are insufficient, using the same serialized native engine boundary.

## 9. Architecture

### 9.1 Layers and ownership

```text
Flutter views + semantic controls
        ↓ actions / immutable view models
Application controllers + command coordinator + access policy
        ↓ ports
Pure Dart domain: versions, navigation FSM, geometry, layers, manifests
        ↓ repositories/adapters
Drift catalogue & encrypted records | immutable encrypted asset store
PDF service | Android platform services | native audio worker
```

Widgets never issue database writes, decrypt raw files or mutate PDF documents directly. One command coordinator serializes mutations per affected aggregate, acquires multiple score locks in ID order, and performs a single catalogue publication transaction. One PDF service owns document handles and serializes native calls, including foreground/background engines. One native audio worker owns sample-clock streams. A source scheduler owns coalesced discovery queues. AccessSession owns protected capabilities and decrypted buffer lifetime.

Use Flutter SDK state primitives and constructor injection initially; a third-party state-management framework is unnecessary for the specified behavior. Controllers expose immutable states/streams, implement disposal, and depend on interfaces. Tests use explicit fake clocks, random providers, sources and fault injectors. Real crypto tests must use real implementations; a fake cipher is not proof of confidentiality.

### 9.2 Data and storage contract

The following are logical records, not a mandate to make every object a huge mutable JSON blob. Sensitive payload columns are authenticated ciphertext even in SQLite WAL.

| Record | Minimum fields/invariants |
| --- | --- |
| Score | ID, public title, folder ID, timestamps, protection state, active generation, original asset reference, trash state |
| ScoreKeyEnvelope | score/generation, wrapping domain/key ID, versioned authenticated envelope |
| Version | ID, score ID, revision, encrypted name/settings, stable original/duplicate lineage |
| PageOccurrence | ID, version ID, ordinal, asset/source page or template, encrypted geometry; unique `(version, ordinal)` |
| Layer | ID, version ID, order, encrypted name/visibility/lock/opacity/group structure |
| Annotation | ID, layer/page IDs, object kind, encrypted payload/bounds/transform, revision |
| SnippetMove | ID, source occurrence/region, destination object, patch asset, cross-score operation ID |
| Bookmark/Link | ID, version ID, encrypted source/destination points, stable occurrence references, broken status |
| Note/NoteFolder | ID, explicit scope/owner, parent, encrypted rich text for score scopes; prevent cycles |
| Label/ScoreLabel | name/color, relation unique per pair; public searchable metadata |
| LibraryFolder/SetlistFolder | ID, parent, name/order; no cycles |
| Setlist/Entry | ID, ordered entry IDs, score/version or placeholder; no uniqueness on score reference |
| LibraryPosition | score/version, page occurrence, mode/half phase; encrypted for protected content |
| Source/SourceItem | provider/root identity, config, cursor, last complete scan, imported identity/digest, tombstones/status |
| History | version, monotonic command index, undo cursor, encrypted forward/inverse, referenced asset IDs |
| Asset | random ID, owner/domain, generation, ciphertext location, digest/length, reference pins |
| Operation | ID/type/state, expected revisions, staged asset IDs, encrypted sensitive recovery payload |
| Settings | typed/schema-versioned global values, never keys mixed into ordinary settings |
| VaultEnvelope | KDF/version/salt, credential envelope, recovery envelope, no plaintext secrets |

Use foreign keys, checks and transactional uniqueness. Titles/labels normalize for matching without destroying display text. UUID/random stable IDs are generated through an injected secure ID service. Keep raw original digests private/keyed where they would disclose protected content equality; duplicate checks may compare accessible originals during import, never bypass a locked scope.

Disk layout: `catalog/` (SQLite), `assets/<random-id>/` (immutable ciphertext), `staging/<operation-id>/`, `recovery/` (validated local safety snapshots), `cache/` (bounded disposable derived data). Keep all beneath private app storage. Archive/export destinations are selected by the user, not source folders by default.

### 9.3 Durable publication protocol

1. Validate command, access capability, referenced entities and expected revision.
2. Allocate operation ID; persist a recovery record with no plaintext sensitive payload.
3. Write new immutable encrypted assets to staging; flush, close, validate hashes/tags and expected shape.
4. Move complete assets into final app-private locations on the same filesystem and durably record their availability. Existing asset generations remain untouched.
5. In one awaited database transaction, install new references/payloads, update revisions/history, and mark the operation published.
6. Acknowledge saved state only after durable transaction success.
7. Cleanup obsolete staging and unreferenced assets later under dependency-aware garbage collection.

A crash before step 5 leaves no published change; recovery reclaims or resumes staged work by operation type. A crash after step 5 sees complete referenced assets; cleanup is idempotent. Never assume a database transaction atomically commits arbitrary filesystem writes. Use SQLite foreign keys, WAL and durability settings verified on Android, with one writer and explicit checkpoint handling for backups/migrations. Await every Drift transaction call; see [Drift transactions](https://drift.simonbinder.eu/dart_api/transactions/).

### 9.4 Native boundary and future iPad

Ports: `DocumentSourcePort`, `PrivateStoragePort`, `CryptoPort`, `PdfEnginePort`, `AudioPort`, `StylusPort`, `BackgroundWorkPort`, `ScreenPolicyPort`, `CredentialStorePort`. DTOs contain domain IDs, geometry and byte streams, never Android Context/Uri types in domain classes. Use typed wrappers for provider IDs. Platform URI serialization belongs to the adapter.

Android: SAF for local file/tree grants, AndroidX WorkManager for scheduled requests, platform crypto/Keystore, native audio output/input and keyboard-style pedal events. Future iPad adapters replace those services while keeping schema, backup format, geometry and domain tests. Do not assume background intervals, stylus buttons or folder grants behave identically on iPad.

## 10. Qualification constraints and sources

Package versions must be pinned together after a clean-build spike. Current package documentation is not proof that a specific combination works on the user's devices. Record exact Flutter/Dart/Gradle/NDK/PDFium versions and licences in an implementation compatibility ledger.

- [pdfrx](https://pub.dev/packages/pdfrx): candidate PDF wrapper; current documentation describes native engine packaging and SDK requirements. Do not blindly adopt a release published just before implementation.
- [PdfDocument API](https://pub.dev/documentation/pdfrx_engine/latest/pdfrx_engine/PdfDocument-class.html): custom reads and PDF encoding are exposed; test exact pinned APIs rather than inventing methods.
- [PDFium public API](https://pdfium.googlesource.com/pdfium/+/refs/heads/main/public/fpdfview.h): verify global serialization/ownership requirements, including direct native export calls.
- [Android SAF](https://developer.android.com/guide/topics/providers/document-provider): provider IDs/grants, not ordinary absolute file paths, underpin linked-folder access.
- [WorkManager requests](https://developer.android.com/develop/background-work/background-tasks/persistent/getting-started/define-work): scheduled work is constrained/deferred by the OS; it is not an exact timer.
- [Drive scopes](https://developers.google.com/workspace/drive/api/guides/api-specific-auth): selected-file access does not automatically prove access to future children of a linked folder. Qualify discovery with an actual test account; read-only full-Drive access may be needed for linking and carries verification implications for future public distribution.
- [Drive limits](https://developers.google.com/workspace/drive/api/guides/limits): check current free quotas before integrating; never enable paid overages automatically.
- [Android stylus](https://developer.android.com/develop/ui/views/touch-and-input/stylus-input/advanced-stylus-features): verify pressure/buttons/cancel/history on actual S Pen hardware.
- [AAudio](https://developer.android.com/ndk/guides/audio/aaudio/aaudio): native sample-clock audio path; do not schedule musical clicks from widget rebuilds.

No product clarification is outstanding from the final round. Capability failures discovered during implementation must be reported with evidence and a concrete alternative before altering the approved behavior. A qualification gate is a required engineering experiment, not permission to ship a placeholder.
