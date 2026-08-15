---
name: monsterdeleter-macos
description: Design, implement, diagnose, and polish the UI and animation experience of the native MonsterDeleter macOS app built with SwiftUI, AppKit, sprite sheets, audio, and a full-screen overlay. Use when requests involve MonsterDeleter visual redesigns, its cat-and-mouse cheese chase, landing or targeting screens, character choreography, motion timing, sprite playback, transitions, confirmation and failure states, sound synchronization, multi-display positioning, animation performance, reduced motion, keyboard interaction, or visual QA.
---

# MonsterDeleter Cat-and-Mouse UI and Animation

Create a playful desktop cartoon chase that feels intentional from target selection through the cat and mouse's exit. Transform the target into cheese visually, let the mouse identify it, ask for confirmation, then have the mouse carry the cheese away while the cat chases it offscreen. Treat motion, sound, copy, and interaction as one sequence rather than isolated effects.

## Begin with the experience

1. Locate the repository using the markers in [ui-animation-reference.md](references/ui-animation-reference.md).
2. Read the files for scene state, overlay rendering, sprite playback, audio, and window positioning before proposing changes. Preserve existing implementation names such as `MonsterScene` when renaming would create unrelated churn.
3. Define the user-visible goal in one sentence and list the affected beats. When the request is broad, audit the current experience and select the highest-impact coherent improvement rather than scattering cosmetic edits.
4. Inspect existing screenshots, recordings, assets, or reference media when available. If none exist, derive the current behavior from code and state what still requires visual confirmation.
5. Preserve unrelated changes and avoid editing generated Xcode output or imported media unless the task explicitly requires it.

## Design the sequence

Write or update a compact motion plan before changing timing-heavy code. For each beat, specify:

- trigger and end condition;
- sprite or visual state;
- position and movement path;
- duration, easing, and interruption behavior;
- audio cue and relative start time;
- overlay copy and available input;
- reduced-motion alternative.

Keep the canonical phase order in `MonsterDeleterStore`. Add a phase only when it represents a real interaction or choreography boundary. Avoid duplicating phase logic or timing constants in SwiftUI views.

Use [ui-animation-reference.md](references/ui-animation-reference.md) for the current phase map, component ownership, coordinate rules, and verification matrix.

## Shape the visual language

- Favor expressive chase-comedy energy: readable silhouettes, anticipation, suspicious inspection, a brief comic pause, and accelerating pursuit.
- Keep the target recognizable and the destructive action unmistakably confirmable. The cheese is a visual stand-in only: show the original file name and do not move the real item to Trash before confirmation.
- Treat the full-screen overlay as a stage. Place copy and confirmation controls relative to the mouse and cheese while clamping them inside safe screen margins.
- Use depth deliberately through dimming, material, scale, shadow, and contrast. Avoid adding generic cards, gradients, particles, or blur without a narrative purpose.
- Keep Chinese UI copy short, characterful, and consistent. Use “是它吗？” for identification and label the destructive confirmation with the actual outcome, such as “是，移到废纸篓”; distinguish cancel, confirm, retry, and return.
- Make targeting, confirmation, failure, and cancellation visually distinct. Provide immediate feedback for hover, drop targeting, focus, and accepted input.
- Maintain legibility across display sizes, dark backgrounds, high contrast settings, and long file names.

## Build motion correctly

- Drive choreography from explicit state and cancellable tasks. Model identification, confirmation, cheese pickup, mouse escape, cat pursuit, and exit as coherent beats. Ensure every sequence ends through completion, cancellation, or failure exactly once.
- Derive sprite duration from frame count and frame rate when the animation owns the timing. Do not stack unrelated magic delays around the same beat.
- Keep sprite decoding and frame caching outside per-frame rendering. Reuse cropped frames and avoid rebuilding providers during view updates.
- Keep frame selection deterministic and safe for empty, missing, clipped, or non-looping sheets.
- Choose interpolation intentionally for the source art and output scale; verify at actual display scale instead of assuming one global setting.
- Use monotonic time for movement progress. Keep animation state on `@MainActor`, but minimize observable writes and view invalidation at high refresh rates.
- Synchronize the identification cue, confirmation sting, cheese pickup, mouse footsteps, cat entrance, chase, and exit with named sequence cues. Stop or fade audio on cancel, failure, and normal completion.
- Convert global AppKit coordinates to local overlay coordinates once at the window boundary. Test displays with negative origins and different scale factors.
- Respect Reduce Motion. Replace long traversal or aggressive scale/flash with a shorter readable transition while preserving confirmation and outcome.

## Protect interaction quality

- Keep Escape available throughout the overlay. Provide a visible cancel action at confirmation and a reliable keyboard default only for deliberate confirmation.
- Keep focus usable in a non-activating/full-screen panel. Verify keyboard, VoiceOver labels, and Full Keyboard Access after window changes.
- Disable repeated confirmation and overlapping invocations once the mouse starts picking up the cheese.
- Announce failures in plain language and return the user to a stable state.
- Continue moving files only to Trash. UI polish must never weaken validation, protected-path checks, or recoverability.

## Verify the experience

1. Run focused state, sprite, audio, or coordinate tests for the changed layer.
2. Run the full suite and build both the app and Finder extension after cross-layer UI changes.
3. Launch the app only when visual QA is authorized. Test with a disposable target; never use a valuable user file.
4. Capture or inspect the complete sequence, not just a static frame. Check cheese transformation, mouse entrance and identification, “是它吗？” confirmation, cheese pickup, mouse escape, cat pursuit, offscreen exit, cancellation, and failure as applicable.
5. Test long names, screen edges, multiple display arrangements, repeated launch, missing media, muted audio, reduced motion, keyboard-only use, and window reopening.
6. Report the visible change, measured or observed performance effect, automated coverage, and any manual checks still required.

## Handle character assets responsibly

- Treat exact Tom and Jerry likenesses, names, animation frames, voices, and sounds as third-party copyrighted material. Do not download, extract, commit, redistribute, or ship them unless the user has the necessary rights.
- For local prototypes, use only user-provided or already-local reference media and keep it out of commits and releases. For distributable builds, create or request licensed assets or original cat-and-mouse characters with a distinct visual identity.
- Keep choreography and state logic independent of specific media so licensed or original sprite sheets can replace local prototypes without rewriting the sequence.

Use the exact commands and detailed checklists in [ui-animation-reference.md](references/ui-animation-reference.md).
