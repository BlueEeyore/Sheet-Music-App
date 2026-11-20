# HighScore design decisions

Status: product decisions consolidated after the final clarification round. See DESIGN.md for the behavior and architecture specification, IMPLEMENTATION_PLAN.md for implementation order, and ACCEPTANCE_TESTS.md for release checks. These documents plan the app; implementation and device validation have not been performed.

The user's later decisions below supersede conflicting statements in SPEC.md. Unresolved proposals must not be treated as approved requirements.

## Confirmed scope

- Working name: HighScore; may change later.
- Android first, with architecture prepared for future iPad support.
- Phones and tablets receive equal design attention.
- Physical test devices: Samsung Galaxy Tab S8+ 5G with S Pen, and Google Pixel 6a.
- Initial distribution is personal use, with architecture prepared for broader distribution.
- All retained specified features belong in the first release. Implementation milestones are not permission to omit features from that release.
- App is free. Open-source app libraries plus the free Android SDK are approved; no required paid services or paid API quota upgrades.
- All imported content and local features must work offline. Network-dependent imports require connectivity.
- No HighScore accounts, hosted library storage, or account-based device synchronization. Earlier account/sync proposals are withdrawn.
- Recoverable trash, full-library backup/restore, duplicate detection, and performance lock are approved additions.
- Annotation layers are approved.
- Ask before adding other features beyond SPEC.md and explicit user decisions.
- User handles git commits.
- Reliability is the highest priority, above feature polish and development speed. The user's expectation is consistent operation every time. This must become concrete implementation and release criteria; do not claim a literal guarantee of zero failures.

## Import and storage behavior

- Import creates an independent app-owned copy of the PDF.
- Subsequent source PDF changes do not modify the imported copy.
- App edits save immediately in the app and do not alter the source file.
- Library names and folders remain independent from source names and folders after import.
- Earlier automatic replacement of imported PDFs when source files change is superseded.
- Previously approved folder traversal choices: include subfolders preserving hierarchy, include subfolders without preserving hierarchy, or exclude subfolders.
- Google Drive is retained as an optional import source within its free limits, without HighScore accounts or hosted storage.
- Linked folders automatically import newly discovered PDFs, ignore edits to already imported PDFs, and retain an informational removed-from-source badge when a source is deleted.
- Google Drive downloads default to Wi-Fi only, with an option to permit mobile data.
- Exact duplicate detection uses PDF contents, regardless of filename. Manual imports offer Skip or Import as a separate score. Automatic imports skip duplicates and report them in import results.
- Deleting an automatically imported score prevents immediate automatic re-import of that source file.
- Approved refresh defaults: check when opening/returning to the library if at least 15 minutes elapsed since the last check; hourly while remaining in the library; request background checks every 6 hours subject to Android scheduling; defer automatic checks/imports while reading or performing; defer background work on low battery. Manual refresh remains available. Intervals are configurable in Settings.
- Source identity tracking and interval validation are specified in DESIGN.md; use provider identities rather than filenames as persistent identity.
- Deferring automatic work has two independent Settings controls, both enabled by default: Defer automatic imports while viewing scores, and Defer automatic imports while performance lock is active. This policy covers optional checks/imports, never durable saves or required recovery work.
- Support ordinary password-encrypted PDFs with a password prompt and optional Remember on this device. Certificate-/DRM-protected PDFs are reported as unsupported; test supported encryption formats explicitly in implementation. Unreadable files receive clear per-file errors while the batch continues; never publish partial imports as successful scores.

## Score access protection

- The user requests an optional PIN/password lock on a library score so that accessing it requires authentication.
- This is distinct from performance lock, which prevents accidental interaction rather than protecting access.
- Protected score content must be encrypted, with a recovery method.
- All protected scores use one shared app-level PIN/password.
- Direct library viewing: relock on leaving the score or changing to another score. Switching versions of that score retains the grant.
- Setlist viewing: authenticate when opening a setlist that contains a protected score; retain its unlock session until exiting the setlist, including while moving between its scores. Cancellation does not enter that setlist.
- Backgrounding the app or switching off the screen does not revoke a live session. Do not implement a timeout that contradicts this. A process restart cannot retain memory-only keys and requires authentication again.
- Recover a forgotten shared credential using an offline recovery key saved outside the app. No hosted recovery. Losing both credential and recovery key makes protected content unrecoverable.
- Encrypt originals, all versions, annotations, snippets, score-scoped notes, and editing history. Titles/labels remain visible; locked thumbnails use a lock placeholder. Global notes are outside score protection.
- Protected content remains encrypted in backups. Restoring it requires that backup's credential or recovery key. Exporting/copying into an unprotected destination requires unlocking and explicit confirmation that the resulting copy is unprotected.
- Platform crypto and key-storage implementation requirements are specified in DESIGN.md, with validation gates in the plan; no cryptographic correctness claim is made before implementation testing.

## Reliability, backup, and performance lock

- Plan for hundreds of scores and diverse PDF types (scanned, vector, mixed). No largest representative document size was supplied; do not invent a universal PDF compatibility or size guarantee.
- On save failure, preserve the last saved state, identify unsaved work, temporarily prevent further editing, allow reading saved content, and offer retry/recovery export where possible.
- Manual local backups contain originals, all versions, layers, snippets, notes, labels, folders, setlists, and settings.
- Restore supports both replacing and merging into the existing library, with archive validation and a safety backup of the current library.
- Merge restoration skips identical items and retains conflicting items as separately named copies without overwriting current work. Imported setlists reference the appropriate imported scores/versions; existing setlists remain unchanged. Existing app settings win during a merge.
- Replacement restoration offers a choice to restore backed-up app settings or keep current settings.
- Tools available in performance lock are configurable. Default allowed tools: page turns, valid navigation links, bookmarks, and metronome controls. Default blocked tools: version switching, drawing, cropping, page arrangement, divider dragging, and zoom/pan.
- Performance lock is manually enabled/disabled using the viewer toolbar padlock. The toolbar stays accessible while locked. Long-pressing the centre of the score for one second also disables the lock, with visible confirmation that performance lock is disabled. Do not show a small persistent lock indicator; the user explicitly rejected it.
- Bluetooth keyboard-style pedals have configurable next/previous mappings. No physical pedal model is currently available for testing; actual hardware compatibility remains unverified until tested.

## Musical tools

- Metronome range is 1–300 BPM, with tap tempo and 1–16 beats per measure.
- Users can choose which beats are accented. The free Metronome Beats app by Stonekick is the reference for this beat-accent interaction; do not copy unrelated features automatically.
- Tapping individual beat indicators cycles Normal → Accented → Muted. Initially beat one is accented and the others are normal. Subdivision choices are 1, 2, 3, or 4 clicks per beat. Three sounds, volume, and audio/visual/both modes are approved.
- Save tempo and meter per score version; sound and volume are shared app settings.
- The metronome continues when its popup closes and through page turns; it stops when leaving the viewer or backgrounding the app.
- Microphone tuning pauses audible metronome clicks while visual pulses continue. Closing the tuner resumes previously running clicks unless the user stopped the metronome. Reference-tone playback pauses microphone listening to avoid self-detection.
- Chromatic microphone tuner shows nearest note and cents sharp/flat; include a selectable reference tone, A4 = 440 Hz default, adjustable over 400–480 Hz.
- Closing the tuner stops microphone listening and any reference tone.
- Instrument transposition is included. Alternative temperaments are excluded; use equal temperament.

## Library organization and trash

- Library and setlist folders support nested subfolders.
- Each score belongs to one library folder and also appears in All scores.
- Deleting a folder offers moving its contents to its parent or moving the contents to Trash.
- Labels filter rather than sort. Multiple labels per score are supported. Included-label filters offer Match any and Match all; excluded labels always take precedence. Do not implement label grouping/sorting from the original SPEC wording.
- Search covers score titles and label names only. PDF full-text search, OCR, and searching written notes are out of scope.
- Trash retention is configurable by number of days; default is until manual removal.
- Setlist entries referring to a trashed score remain visibly unavailable, with actions to restore the score or remove the entry.

## Setlists and reading position

- Merging creates a new setlist in a user-selected source-setlist order, retains repeated entries, and leaves originals intact.
- Duplicating a setlist copies its entries and selected versions, retaining references to the same scores and versions.
- Reopening from the library resumes the page the user left, not the final page of the document.
- Selecting an entry from a setlist starts it at page one. Turning backward across a score boundary enters the previous entry at its last page. Navigation stops at the final entry without looping.
- Two-page mode never mixes different scores; an unmatched page appears beside empty space.
- Half-page mode finishes the current score before showing the next score.
- Bookmarks and links follow their target page identities after reordering. Removing referenced pages warns and identifies affected references. By default, retain broken references with Destination removed status, disable their jumps, and offer Choose destination or Delete. A setting switches this policy to deleting affected references when the destination page is removed.
- Every score version must retain at least one page.

## Score versions

- Named/selectable score versions replace the earlier setlist-local edit proposal.
- Users can duplicate an existing version and edit the duplicate independently (for example, crop an annotated version while preserving the previous version).
- Users can easily switch versions from the viewer toolbar.
- A setlist remembers its selected score version; different setlists can select different versions of the same score.
- Each version independently owns annotations/layers, crops/rotation, page arrangement, bookmarks, and links.
- Score title, library folder, and labels are shared across versions. This division was explained to the user.
- Notes can be shared across a score's versions or specific to a version. Global note folders from SPEC.md remain in scope.
- Notes support formatting including bold, italics, and lists, and nested folders. The three locations are This version, All versions of this score, and Global.
- Editing a version updates every setlist entry that references that version; duplicating creates an independent version.
- Every individual setlist entry remembers its selected version. Repeated scores are allowed with either identical or different versions.
- Library opening uses the last version selected from the library, independently of selections in setlists.
- Deleting a version referenced by setlists requires choosing a replacement first.
- Long-press a version in the version chooser to enter selection mode. Duplication is contextual to selecting one version, rather than a permanently visible button. Selection supports deleting one or multiple versions. Deselecting every version automatically exits selection mode.
- Version merging is excluded: the user withdrew that proposal and requested deletion instead. Layer merging remains approved and separate from this decision.
- The half-page split is a single percentage shared across every score, version, and orientation on this installation. It persists until the user changes it, superseding the earlier per-page/per-version proposal.

## Annotation layers and input

- Named layers span the whole version and support show/hide, lock/unlock, reorder, rename, duplicate, delete, opacity, and merging.
- Drawing, erasing, and selection operate on the active unlocked layer.
- Duplicating a version copies all its layers.
- Selection offers both Move to new layer and Copy to new layer.
- Text and stamps remain whole objects when selected; no partial cutting of those objects.
- S Pen draws; finger input turns pages and zooms/pans on the tablet.
- Initial pen contact both opens the annotation toolbar and begins drawing.
- On tablets, stylus contact opens the annotation toolbar AND the general toolbar if it is hidden; it never closes the general toolbar. A centre finger tap closes annotation controls and shows the general toolbar; another centre tap hides the general toolbar. No dedicated Hide action.
- On phones an explicit annotation button enables finger drawing; leaving annotation mode restores finger page turns.
- Holding the S Pen button temporarily activates the eraser.
- Annotation selection supports move, resize, rotate, copy, cut, delete, recolor, and move/copy to another layer. Text and stamps always remain whole objects.
- Both area and stroke erasers affect pen/highlighter strokes only. Text, stamps, and snippets are removed using selection actions.
- Undo/redo survives closing/reopening, is separate per version, and covers annotations, layers, snippets, cropping, and page arrangement. Default retention: 200 editing actions, configurable in Settings.
- Printed-music snippet editing is approved using high-resolution image captures, not arbitrary native PDF object editing. Select an explicit PDF content layer, then rectangle or lasso select; copy, cut/move, resize, rotate, and delete are supported.
- Moving/cutting printed content covers the original region with a reversible blank patch. Changes are version-specific and never alter the imported original.
- Snippets can be copied/pasted across pages, versions, and scores. Capture offers printed music only or printed music plus visible annotations; included annotations become part of the flattened snippet image.
- The user was informed that captured snippets can pixelate at high magnification and approved this approach.
- Removing a complete moved snippet restores its source region; label this action Restore snippet rather than Delete/Remove to communicate its effect. Removing a copied snippet removes only the copy.
- Notation stamps use the free/open-source Bravura music-symbol font with a searchable, categorized catalogue of supported symbols. No additional unusual symbol requirements were specified.
- Inserted pages offer plain paper and staff-paper templates. Default dimensions match the current page, with A4 and US Letter alternatives.

## PDF export

- Export the selected version's page order, crops, rotations, and currently visible annotation layers.
- Exclude hidden layers, interface warmth/dimming, and navigation dots from the exported appearance.
- Also offer an unannotated version and the untouched imported original.
- Unannotated version export retains printed-music rearrangements/snippets but omits ordinary annotation layers. Original PDF export remains completely untouched.
- Navigation links should remain clickable in exported PDFs.
- Preserve underlying vector music and selectable text where supported; flatten visible app annotations into page appearance. Preserve valid clickable navigation links; omit broken links with a warning. Never silently fall back to a lower-quality raster export.

## Viewer references

- Functionality reference: Lectern (com.lecternapp.lectern), subject to the explicit feature scope.
- Aesthetic reference: forScore, adapted to HighScore's approved palette and responsive layout.
- Half-page reference sequence: full page 27; top of page 28 with bottom of page 27; full page 28.
- Links jump from the visible source dot to the destination page and flash its destination dot three times.
- A setting controls whether destination dots are normally visible.
- The half-page divider is draggable; its percentage is globally shared and persists until changed.
- Include Whole page (proportional, fully visible), Fit height (proportional, horizontal panning if needed), Fit width (proportional, vertical panning if needed), and Stretch to screen (non-proportional).
- Whole page is the initial installation default. Thereafter use the last selected fit mode.
- Dark mode changes the interface while the PDF keeps its original colours. Eye comfort separately applies adjustable warmth/dimming.
- The viewer initially keeps the screen awake. Settings offer Never sleep, System timeout, and a custom inactivity duration. Outside the viewer, normal system behavior applies.
- Deterministic gesture, page-turn and layout rules are specified in DESIGN.md.
- Score and annotation toolbar actions use icons instead of word labels, with accessible action names. Most tools are directly available on the toolbar rather than behind More. Remove the dedicated Hide-toolbar action.
- Rotation and straightening belong inside Crop, without separate general toolbar buttons.

## Approved visual direction

- Accent: #2F7434, selected by the user. This replaces the earlier magenta #E94A64 throughout the current design.
- Navy: #18243A.
- Library add button: green #2F7434 background with a pale-green #D9F2D7 plus glyph, replacing the dark glyph.
- The user requested no further mockups; use the existing review file and complete the written design/plan after necessary remaining clarifications.
- Interface typeface: Liberation Sans, selected by the user.
- Restrained monochrome controls and score thumbnails.
- Light, dark, and system-following interface themes.
- Phone bottom-tab navigation and tablet sidebar navigation.
- Full-screen score viewing on both form factors.
- Tablet side panels and phone expandable sheets.
- Deliverables include screen mockups, written behavior/architecture, and a Markdown implementation plan suitable for a less capable implementation model.
- The user viewed the standalone library preview and explicitly approved its visual direction. The approved preview is docs/mockups/highscore-library.html. Further preview deliverables should be supplied as files because the inline preview was not visible to the user.
- The earlier standalone viewer at docs/mockups/highscore-viewer.html is a historical proposal, superseded by the revised review and final written behavior. It is a design prototype, not an implemented PDF editor. Static checks were performed; no browser-based visual verification is claimed.
- The library + action opens a centred modal, replacing the earlier dropdown menu.
- The current review file is docs/mockups/highscore-review-v2.html. It incorporates icon toolbars, input-triggered visibility, version selection/duplication/deletion, and the library import modal. The earlier viewer is a historical proposal rather than the latest design.
- Liberation Sans is approved and applied by default in the current review. Earlier font alternatives remain as reference samples. Embedded fonts retain licence notices in the review file.
- The user's custom green #2F7434 is approved and applied to the current mockup, including matching light/dark selection backgrounds. Earlier green candidates remain as reference swatches; they are not the selected palette. Earlier mockup files retain their historical appearance.

## Approved technical direction

- Flutter/Dart for shared application logic and responsive interfaces.
- SQLite through Drift for local structured data, with PDFs stored in app storage.
- pdfrx/PDFium as the leading PDF rendering/manipulation candidate.
- Custom editable annotation layer; do not assume the PDF package supplies the required eraser, selection, or export behavior.
- Early implementation validation should cover stylus latency, partial-stroke editing, and PDF export.
- Supabase and all proposed cloud synchronization infrastructure are removed from consideration for this release.

## Research references

Checked 2026-11-20; external terms and package capabilities must be rechecked when relevant implementation begins.

- Samsung documents that PDF import creates a copy without altering the original: https://www.samsung.com/us/support/answer/ANS10004548/
- forScore documents importing copies: https://forscore.co/documentation/adding-files/
- Android SDK download and license (royalty-free use, distinct from an entirely open-source license): https://developer.android.com/studio
- Google Drive API standard usage is currently available at no additional cost; quotas and announced future overage billing must be respected: https://developers.google.com/workspace/drive/api/guides/limits
- Lectern viewer guide: https://lecternmusic.com/guide/viewer
- Flutter platform support: https://docs.flutter.dev/reference/supported-platforms
- pdfrx: https://pub.dev/packages/pdfrx
- Drift: https://pub.dev/packages/drift
- Metronome Beats accent/mute reference: https://stonekick.com/metronome_changelog.html
- Bravura symbol catalogue/font: https://github.com/steinbergmedia/bravura

## Final handoff precedence

- User decisions in this record and the final clarification round override historical mockup interactions. In particular, the historical review file may show mutually exclusive toolbars; the final implementation must open both on stylus contact. No additional mockup is requested.
- Detailed engineering defaults and qualification gates in the plan do not authorize feature reductions, paid services, account sync, or new product features.
