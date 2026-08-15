# MonsterDeleter UI and animation reference

## Contents

- Repository and product intent
- Experience architecture
- Target choreography
- Visual and motion audit
- Implementation rules
- Verification commands
- Manual QA matrix

## Repository and product intent

The repository currently lives at:

`/Users/zhuabao/Documents/Codex/2026-08-09/pip-install-r-requirements-txt/MonsterDeleter-macOS`

If it moves, locate a directory containing:

- `project.yml`
- `Sources/Views/TargetingOverlayView.swift`
- `Sources/Stores/MonsterDeleterStore.swift`
- `Sources/Models/MonsterScene.swift`
- `Tests/MonsterDeleterStoreTests.swift`

MonsterDeleter is a menu-bar-only macOS 14+ app. It turns moving one selected file or folder to Trash into a desktop cat-and-mouse chase: the selected item becomes cheese visually, a mouse identifies it, the user confirms, the mouse carries it away, and a cat chases the mouse offscreen. The UI should feel mischievous and cinematic while keeping the original target, confirmation, cancellation, and result unambiguous.

Exact Tom and Jerry imagery, names, voices, animation frames, and sounds require appropriate rights. Keep user-provided prototype media local and out of commits and releases; use licensed or visually distinct original cat-and-mouse assets for distribution.

## Experience architecture

| Experience layer | Primary files | Owns |
| --- | --- | --- |
| Landing | `Sources/Views/LandingView.swift`, `Sources/Stores/LandingStore.swift` | Product introduction, picker, drop target, extension entry |
| Choreography | `Sources/Stores/MonsterDeleterStore.swift`, `Sources/Models/MonsterScene.swift` | Phase transitions, timing, mouse and cat movement, cancellation, trash cue |
| Overlay composition | `Sources/Views/TargetingOverlayView.swift` | Targeting, cheese and character placement, confirmation and failure UI |
| Targeting cursor | `Sources/Views/TargetingCursorView.swift` | Pointer affordance and hit region |
| Sprite playback | `Sources/Views/SpriteSheetView.swift` | Timeline-driven frame selection and display |
| Sprite preparation | `Sources/Services/SpriteSheetFrameCache.swift` | Image loading, cropping, and cache reuse |
| Audio | `Sources/Services/AudioService.swift` | Music, identification, pickup, chase, and stop behavior |
| Window stage | `Sources/WindowControllers/OverlayWindowController.swift`, `OverlayCoordinator.swift` | Screen selection, coordinate conversion, key handling, lifetime |
| Finder handoff | `FinderSyncExtension/FinderSync.swift`, `FinderInvocationLocationTracker.swift` | Selected target and invocation position |

Keep ownership clear: the store decides what happens and when; views render the current state; services load or play resources; the window layer translates screen geometry.

## Target choreography

Evolve the canonical scene phases toward these user-visible beats. Keep existing enum names temporarily when a rename would add unrelated churn, but do not preserve obsolete behavior merely to match an old phase name.

1. `targeting`: show the stage and determine the target position when Finder did not provide one.
2. `transforming`: visually morph or swap the file icon into a cheese prop while retaining the original file name and type in the confirmation UI.
3. `approaching`: bring the mouse in from the nearest sensible screen edge and move it toward the cheese.
4. `identifying`: let the mouse point at or inspect the cheese, then present “是它吗？”.
5. `awaitingConfirmation`: offer a visible cancel action and an explicit destructive action such as “是，移到废纸篓”.
6. `grabbing`: after confirmation, perform the real Trash operation at one named cue and let the mouse lift the cheese. If Trash fails, do not begin the escape.
7. `escaping`: accelerate the mouse toward an exit edge while it visibly carries the cheese.
8. `chasing`: bring the cat in behind the mouse and preserve readable separation so both silhouettes remain understandable.
9. `exiting`: move both characters fully offscreen, resolve audio, and close the overlay exactly once.
10. `failed`: stop the sequence, restore a stable representation of the target, stop audio, and show recovery UI.

Treat the old walk, kick, explosion, celebration, and flight timings as migration inputs rather than design requirements. Tune the new sequence as a connected rhythm: quick cheese reveal, readable inspection, unhurried confirmation, crisp pickup, then a short accelerating chase.

### Beat contract

| Beat | Visual requirement | Audio/copy | Completion rule |
| --- | --- | --- | --- |
| Cheese reveal | Preserve target position; transform only its stage representation | Short magical or comic cue | Cheese and original file name are both readable |
| Identification | Mouse faces and clearly indicates the cheese | “是它吗？” | Confirmation controls are visible and focused correctly |
| Confirmation | Nothing destructive moves yet | “取消” and “是，移到废纸篓” | Accept exactly one user decision |
| Pickup | Mouse makes contact and cheese becomes attached to its carry anchor | Pickup sting | Trash succeeds; otherwise transition to `failed` |
| Escape | Mouse carries cheese with no sliding or anchor mismatch | Fast footsteps | Mouse establishes a lead toward the chosen edge |
| Pursuit | Cat enters on the same path after a brief readable delay | Chase cue | Cat follows without visually covering the mouse |
| Exit | Mouse, cheese, and cat cross fully beyond the safe frame | Music resolves or fades | Overlay closes exactly once |

## Visual and motion audit

For a broad polish request, inspect the experience in this order:

1. Clarity: can the user identify the target, next action, cancel route, and outcome?
2. Rhythm: do anticipation, arrival, point, confirmation, impact, reaction, and exit each have enough space without dragging?
3. Continuity: do sprite pose, position, audio, overlay copy, and controls change as one beat?
4. Composition: are mouse, cheese, cat, original target name, and confirmation balanced and visible near every screen edge?
5. Responsiveness: does the interface acknowledge click, drop, confirmation, Escape, and failure immediately?
6. Performance: are frames cached, updates bounded, audio prepared, and overlays free of unnecessary invalidation?
7. Adaptation: does the scene survive long names, small screens, multi-display geometry, missing resources, mute, and Reduce Motion?
8. Character: does the result feel like a purposeful cat-and-mouse caper rather than a generic macOS utility or a direct copy of unlicensed production artwork?

Record findings as user-visible symptoms. Tie each proposed change to one symptom and one acceptance check.

## Implementation rules

### State and timing

- Keep transitions in `MonsterDeleterStore` and guard them by current phase.
- Use named timing values or a small choreography configuration when several values must be tuned together.
- Make cancellation idempotent. Cancel all active sequence and character-motion work, stop audio, and finish once.
- Use injectable timing, clock, audio, and trash boundaries when deterministic tests require them.
- Avoid starting choreography from view appearance more than once. Treat repeated SwiftUI task creation as a lifecycle risk.

### Sprite rendering

- Keep `SpriteSheetAsset` metadata consistent with actual grid dimensions and source frame aspect ratio.
- Validate frame indices before display and cover skipped/clipped frames explicitly.
- Cache decoded/cropped frames by all metadata that affects the result.
- Profile memory and main-thread work before changing caching strategy.
- Inspect the chosen interpolation at native, 1×, 2×, and fractional scales where relevant.

### Layout and windows

- Clamp overlays using their measured size plus safe margins, not a single hard-coded center allowance.
- Define whether stored positions represent centers, top-left origins, or target points; convert deliberately.
- Remember AppKit screen coordinates use a bottom-left global origin while SwiftUI overlay coordinates use a top-left local origin in this project.
- Test screens whose `minX` or `minY` is negative and screens above the primary display.
- Preserve panel transparency, all-Spaces behavior, key handling, and deterministic close/release behavior.

### Sound and accessibility

- Name sound cues by scene event rather than view lifecycle: cheese reveal, identification, pickup, mouse run, cat entrance, chase, and exit.
- Prepare players before the cue and define replay/overlap behavior.
- Keep essential meaning visual; never require audio to understand confirmation or outcome.
- Add accessibility labels to meaningful controls and hide decorative sprite frames.
- Read `NSWorkspace.shared.accessibilityDisplayShouldReduceMotion` or an equivalent environment boundary when implementing reduced motion, and make it testable.

### Safety boundary

- Keep `FileManager.trashItem` and existing target validation intact unless the task explicitly addresses them.
- Use mocks for choreography tests. For end-to-end visual QA, create a disposable target in an isolated temporary folder and disclose that it will be moved to Trash.

## Verification commands

Run from the repository root. Regenerate only when `project.yml` changes:

```bash
xcodegen generate
```

Run the full test suite:

```bash
xcodebuild -project MonsterDeleter.xcodeproj -scheme MonsterDeleter -configuration Debug -derivedDataPath DerivedData CODE_SIGNING_ALLOWED=NO test
```

Build the app and embedded extension:

```bash
xcodebuild -project MonsterDeleter.xcodeproj -scheme MonsterDeleter -configuration Debug -derivedDataPath DerivedData CODE_SIGNING_ALLOWED=NO build
```

Use `just check` or `just build` when `just` is available. Do not import upstream assets solely for logic tests. Run the media import script only after the user acknowledges the repository's license warning.

## Manual QA matrix

| Area | Cases |
| --- | --- |
| Landing | picker, valid/invalid drop, hover/drop feedback, long target name, extension link |
| Targeting | background present/missing, pointer visibility, click accuracy, Escape, screen edges |
| Reveal and arrival | cheese replacement, original-name visibility, mouse entry, correct stop point, no repeated start |
| Identification and confirmation | pointing/inspection pose, “是它吗？”, focus, Return, Escape, click, target clarity, edge clamping, repeated input |
| Pickup | Trash cue occurs only after confirmation, cheese attaches to mouse, failure prevents escape |
| Chase and exit | mouse lead, cat delay, readable silhouettes, attached cheese, smooth acceleration, full offscreen completion |
| Failure | immediate stop, readable error, stable return path, no lingering window/audio |
| Displays | primary, left/right/above secondary, negative origins, differing scale factors |
| Accessibility | VoiceOver, Full Keyboard Access, Reduce Motion, increased contrast, muted audio |
| Lifecycle | repeated URLs, reopen from menu bar, cancel then restart, close/release, clean quit |
