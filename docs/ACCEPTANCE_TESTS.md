# HighScore — acceptance and release evidence

Status: planned tests, not executed app results. The implementing model must record actual results in `RELEASE_VALIDATION.md`. A checkbox in a plan is not a passing test.

Use generated or appropriately licensed fixtures. User-supplied score screenshots are visual references, not permission to distribute their music in a released test bundle. Keep personal PDFs in ignored local fixtures; ship only authored/licensed fixtures and fonts.

## 1. Test corpus

Create fixture IDs with reproducible generation parameters and hashes:

- F01: 12-page vector score, selectable text, repeated measure labels and internal PDF links.
- F02: 100-page grayscale scans, varying scan resolution and margins.
- F03: 300-page mixed scan/vector document; exercise bounded loading and bookmarks near its end.
- F04: mixed A4/Letter/custom page sizes, portrait/landscape and intrinsic rotations 0/90/180/270, non-zero page boxes.
- F05: ordinary password-encrypted PDFs covering the encryption schemes actually supported by the pinned engine; correct/wrong/empty-password examples.
- F06: truncated/corrupt/unsupported-security PDF plus valid files in the same import batch.
- F07: identical bytes under different names; same filename/different bytes; a provider file whose revision changes while identity stays fixed.
- F08: high-complexity vector page and oversized scan requiring tiling/cancellation; record actual resource limits.
- F09: generated plain and staff-paper templates with text/stamps/highlights at known coordinates.
- F10: a known-value annotation/notes payload containing unique sentinel strings for plaintext-leak tests.
- F11: setlists with duplicated entries, placeholders, different versions, protected/unprotected/trashed/missing scores.
- F12: backup fixtures: valid, truncated, bad hashes/tags, traversal paths, duplicate entries, unsupported mandatory schema, malicious lengths, conflicting IDs and old credentials.
- F13: synthetic known audio tones, noisy tones and silence; metronome sample-clock schedules at 1/92/300 BPM and all subdivisions.

The 500-score scale library may duplicate authored content intentionally under independent score identities. The test must still include distinct originals and varied documents; 500 references to one cached tiny PDF is not representative evidence.

## 2. Product traceability

| Test ID | Behavior / fixture | Required outcome | Milestone |
| --- | --- | --- | --- |
| UI01 | Light/dark/system; phone/tablet | Liberation Sans, #2F7434, readable states, pale #D9F2D7 library plus | M01 |
| UI02 | Viewer toolbars | Icon actions; no Hide; crop contains rotate/straighten; major tools direct | M01/M07 |
| UI03 | First stylus contact | Both toolbars open and the same contact starts drawing without displacement | M08 |
| UI04 | Centre finger taps | Annotation closes/general shows, then general hides; no accidental page turn | M07/M08 |
| UI05 | Phone draw entry/finish | Explicit finger drawing; finish restores navigation | M08 |
| UI06 | Library + | Centred modal with all import actions, dismiss/focus works | M05 |
| UI07 | Font scale/rotation/accessibility | Touch targets stay usable; labels/focus/selection do not rely solely on color | M01/M16 |
| IM01 | Single/folder import F01–F04 | Complete independent app copies; original hashes unchanged | M04 |
| IM02 | Hierarchy options | Preserve, flatten and exclude produce exact expected tree | M04 |
| IM03 | Duplicate F07 | Manual Skip/separate; automatic skip/report; no shared editable state | M04 |
| IM04 | Changed/renamed source | Existing score bytes/title/location unchanged; no repeat import | M04 |
| IM05 | Deleted vs unavailable source | Confirmed deletion badge only after authoritative success; never remove library copy | M04/M15 |
| IM06 | Password/batch F05–F06 | Correct retry/cancel/remember; other valid files proceed; no partial score | M04 |
| IM07 | Auto-import tombstones | Trashing/purging imported item does not recreate it on next scan | M04/M05 |
| IM08 | Drive future child discovery | A new PDF created after linking is found within eligible checks; no paid use or source writes | M00/M15 |
| LI01 | Folder nesting/moves/deletion | No cycles; contents parent-vs-trash choice; stable IDs on restore | M05 |
| LI02 | Search/sorts | Titles/labels only; stable sort tie-break; no label sort | M05 |
| LI03 | Label ANY/ALL/exclusions | Exact expected query results with combined search/folder | M05 |
| LI04 | Recents/library resume | Successful opens ordered correctly; library resumes last-left page/version | M05/M07 |
| LI05 | Trash retention | Manual default never purges; configured expiry uses deletion age | M05/M15 |
| VE01 | Duplicate edited version | Crop/layers/links/snippets/notes independent; original bytes identical | M06/M10 |
| VE02 | Selection behavior | Hold selects; one selected Duplicate/Delete; multiple Delete only; zero exits | M06 |
| VE03 | Delete referenced/final version | Replacement required; final version retained; no dangling entry | M06 |
| SE01 | Repeated score entries F11 | Independent entry identities and version selections | M06 |
| SE02 | Duplicate/merge setlists | Ordered concatenation, duplicates preserved, originals unchanged | M06 |
| SE03 | Boundary navigation | Forward first/next; backward previous last; no wrap; placeholders work | M07 |
| SE04 | Library vs setlist positions | Setlist starts page one and does not overwrite library-selected version/position | M06/M07 |
| SE05 | Missing/trashed entry | Explicit unavailable state, restore/remove, no silent skip | M06/M07 |
| RD01 | Single/two-page strides | Exact expected states, unmatched blank, never mixed scores | M07 |
| RD02 | Half-page F01 | Full N → top N+1/bottom N → full N+1; reversible; no cross-score composite | M07 |
| RD03 | Divider persistence | Same ratio across scores/versions/orientations; no change until user edits | M07 |
| RD04 | Four fits F04 | Correct proportional pan modes and non-proportional Stretch; last fit remembered | M07 |
| RD05 | Render failure/stale result | Current usable image retained; stale version never painted | M07 |
| RD06 | Transform inverse | Round-trip point error within numeric tolerance at corners/centre for all transforms | M07/M10 |
| AN01 | Pen/highlighter | Width/color/pressure/presets persisted; continuous highlight not darkened repeatedly | M08 |
| AN02 | Barrel button/palm/cancel | Temporary eraser and correct restoration; palm never turns page | M08 |
| AN03 | Area/stroke erasing | Correct stroke fragmentation/whole removal; no text/stamp/snippet damage | M08 |
| AN04 | Lasso/rectangle whole/partial | Boundary geometry correct; text/stamps remain whole | M09 |
| AN05 | Transform/copy/cut/recolor | Correct canonical placement and independent copied IDs | M09 |
| AN06 | Layer controls and transfers | Visibility/lock/order/opacity honored; move/copy to existing/new layer | M09 |
| AN07 | Layer merge | Appearance preserved including overlapping translucent groups; objects remain editable | M09 |
| AN08 | History restart/trim | Undo/redo per version, default 200, new-edit redo truncation, required assets pinned | M09 |
| AN10 | Linked cross-owner history | Undo/redo atomic; incompatible partner cursors require intervening edits to be undone, never overwritten | M09/M10 |
| AN09 | Text/Bravura | Editable content, styles, sizing, supported catalogue, embedded export fonts | M09/M13 |
| SN01 | Snippet capture choices | Printed-only vs visible-annotations capture differs correctly | M10 |
| SN02 | Move/Restore snippet | Blank source patch reversible; copied snippet Delete does not restore unrelated source | M10 |
| SN03 | Cross-owner cut failure | Both owners old or both new; cancel leaves source intact | M10 |
| SN04 | Source/destination page deletion | Destination deletion restores patch; source deletion retains surviving moved content with unavailable restore action | M10 |
| CP01 | Crop/rotate/straighten/reset/all | Non-destructive geometry, mixed sizes handled, one-step undo | M10 |
| PA01 | Reorder/add/remove F04/F09 | Stable page occurrences; original/new/existing/template sources; retain one page | M10 |
| RE01 | Bookmark/link reorder | Intended destination survives reorder of duplicated source pages | M11 |
| RE02 | Removed destinations | Retain/repair default or delete per setting; undo restores identity | M11 |
| RE03 | Link placement/jump | Two-step creation/cancel; target flash three times; normal-dot setting | M11 |
| NO01 | Note scopes/formatting | Nested folders, bold/italic/lists, version/shared/global ownership | M11 |
| AU01 | Metronome bounds/options F13 | 1–300, 1–16, accent/mute, subdivisions, sounds, tap, volume, border pulse | M12 |
| AU02 | Timing under load | Sample-clock evidence, no accumulated tempo drift or unbounded underruns | M12 |
| AU03 | Tuner/reference/transposition | Known pitch/cents and concert/written mapping; silence not fake stable pitch | M12 |
| AU04 | Audio interaction | Audible clicks pause for mic, visual continues; eligible resume; tone pauses mic | M12 |
| AU05 | Lifecycle | Popup/page turns continue; viewer exit/background stop; no surprise resume | M12 |
| PE01 | Pedal key mapping | Correct deliberate turns; ignore auto-repeat without dropping rapid distinct presses | M12 |
| PL01 | Performance lock | Default allowed/blocked actions, configurable, toolbar toggle and centre hold work | M07/M15 |
| PL02 | Auth independence | Performance settings never grant encrypted access | M03/M15 |
| EX01 | Export current/unannotated/original | Correct appearance/structure; original hash equal | M13 |
| EX02 | Fidelity and links | Vectors/text preserved where supported; valid links; broken warnings; no silent raster | M13 |
| EX03 | Visibility | Hidden layers/UI dots/warmth/dimming excluded; snippet capture policy honored | M13 |
| BK01 | Full backup portability | Restore on clean second install including all content/settings/history/trash | M14 |
| BK02 | Replacement settings | Restore or keep ordinary settings, with valid incoming protection envelopes | M14 |
| BK03 | Merge conflicts | Exact skips, separately named conflicts, complete remapping, existing settings retained | M14 |
| BK04 | Hostile/corrupt archive F12 | Reject without changing current state; bounded resource usage | M14 |
| BG01 | Default intervals/coalescing | Library-entry/hourly/background requests, no busy polling/duplicate publication | M15 |
| BG02 | Deferral independence | Either toggle defers applicable optional work; never defers saves | M15 |
| BG03 | Network/battery | Drive Wi-Fi default, mobile opt-in, low battery deferral, recoverable auth/errors | M15 |
| SC01 | Awake/warmth | Viewer policy and adjustments; release awake elsewhere; exports unaffected | M15 |

## 3. Protected-session matrix

Test each with real crypto and route-level integration, not only a mocked Boolean.

| ID | Action | Expected result |
| --- | --- | --- |
| PR01 | Open protected score directly | Prompt; cancellation reveals no content |
| PR02 | Switch versions in direct protected score | No repeated prompt; same scoped grant |
| PR03 | Exit direct score to library, reopen | Grant revoked; prompt again; locked thumbnail placeholder |
| PR04 | Open setlist whose first score is unprotected but later score is protected | Prompt at setlist opening, before entry |
| PR05 | Traverse protected/unprotected/repeated entries within that setlist | One grant remains valid; no repeated prompt |
| PR06 | Return from viewer to that setlist's own contents/editor | Same setlist scope remains unlocked |
| PR07 | Exit setlist to library or open another setlist | Old scope revoked; new protected-containing setlist prompts |
| PR08 | Background app and resume, process survives | Same scope remains unlocked; metronome has stopped |
| PR09 | Screen off/on, process survives | Same scope remains unlocked |
| PR10 | Force-stop/process kill/reboot, restore route | Prompt; no persisted plaintext grant/key |
| PR11 | Open global notes while protected score locked | Global notes accessible; no score-scoped note content leaked |
| PR12 | Inspect database/WAL/staging/cache/logs | Sentinel protected content absent in plaintext; only approved public metadata visible |
| PR13 | Wrong password/recovery key, damaged ciphertext | No content access/no silent vault reset |
| PR14 | Recovery reset then reopen all protected scores | New shared credential works without content loss |
| PR15 | Backup before password change, restore afterward on clean install | Backup's original credential/recovery works; current credential not falsely promised |
| PR16 | Export/copy protected to unprotected | Unlock plus explicit disclosure; cancel leaves no copy |
| PR17 | Restore protected backup into existing vault | Source unlocked, keys rewrapped; destination uses its one shared credential |
| PR18 | Credential lost and recovery lost | Honest unrecoverable state; no bogus reset that silently destroys files |
| PR19 | Protect/unprotect same score through injected crash | One complete generation remains readable under its correct policy |
| PR20 | Duplicate-content unprotected score exists | Protected score lock does not imply erasing an explicit separate unprotected copy; no accidental shared-key bypass |
| PR21 | Link/prefetch/clipboard outlives its scope | Access denied and stale rendered/decoded content removed on scope exit |

## 4. Fault-injection matrix

Instrument these named barriers: operation-created; first asset chunk; final chunk-before-flush; validated staging; asset-published-before-DB; transaction-before-commit; committed-before-ack; acknowledged-before-cleanup; cleanup-in-progress.

For each operation below, fail/kill at every applicable barrier, reopen, then kill/reopen recovery again. Record the expected committed revision and surviving asset pins before the test.

| ID | Operation | Invariant after recovery |
| --- | --- | --- |
| FT01 | Single/folder import | Each visible score complete; no duplicate from resume |
| FT02 | Completed stroke/text/notes | Last acknowledged revision survives; unfinished content not falsely marked saved |
| FT03 | Version duplicate/delete | References match either old or new complete version graph |
| FT04 | Layer merge/partial eraser | Geometry and inverse history agree, no half-applied fragments |
| FT05 | Cross-score cut/paste | Both source patch and destination commit or neither |
| FT06 | Page reorder/remove | Stable references correct/broken by explicit policy, never wrong destination |
| FT07 | Trash/purge/GC | Referenced/history/pinned content not reclaimed prematurely |
| FT08 | Protection conversion/password reset | Valid envelope plus complete content generation retained |
| FT09 | Backup/export | Incomplete external output not reported as complete; local data unchanged |
| FT10 | Replacement restore | Old or new complete library generation, safety backup retained |
| FT11 | Merge restore | No partial graph/cross-linked originals; retries idempotent |
| FT12 | Schema migration | Recoverable pre-migration state; no silent database reset |

Also inject full disk, file permission failure, DB lock/busy, revoked SAF grant, Drive 401/403/429, dropped network, native renderer error, cancelled pointer, lost audio focus and low-memory notification. Distinguish retryable failures from corruption/unsupported formats. Never retry indefinitely at full speed.

## 5. Physical release session

Perform on both devices with actual OS versions recorded:

1. Cold launch in airplane mode with 500 scores. Search/filter and open a large scan.
2. Run 100 page turns through all modes and setlist boundaries; insert deliberate rapid presses.
3. Annotate on S Pen device with pressure, palm contact, button erase, half-page state and rotated crop. Repeat equivalent phone finger-drawing flow.
4. Duplicate/select/delete versions and verify empty selection exits. Include a referenced version replacement.
5. Use links/bookmarks and notes; kill/reopen after durable-save indicator.
6. Start metronome, tune, use reference tone, interrupt audio and background/resume. Verify auth grant retention separately.
7. Rehearse protected-containing setlist through duplicate entries and locked-score content; exit/reopen to verify the prompt boundary.
8. Export modes and inspect in an independent reader. Confirm original source hash unchanged.
9. Backup/restore across installations and exercise recovery key.
10. Continue a two-hour mixed rehearsal while logging non-sensitive memory/frame/audio counters. Compare battery against a plain-reader baseline under matching brightness/radio settings. Report observed values, not guessed battery percentages.

A keyboard can validate key mapping but cannot certify an unavailable physical pedal model. Record that gap explicitly.

## 6. Release evidence format

For every test ID record: build/version, device/OS, fixture hashes, automated command or manual steps, actual outcome, artifact/log path, pass/fail/blocked and issue link if applicable. Screenshots alone do not prove durable saving or encryption.

Release blockers include any reproducible committed-data loss, incorrect-version edit, wrong page/link destination, plaintext protected content leakage, access scope bypass, corrupt restore, silent export degradation, required feature stub or unbounded resource growth in the qualified corpus. Performance targets must include actual measurements and exceptions; do not mark them green based on code inspection.
