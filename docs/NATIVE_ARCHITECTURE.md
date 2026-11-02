# Aether native architecture — proposal v0.1

**Status:** Proposed direction, based on the initial source audit. This is an architecture plan, not implemented code.  
**Target:** iOS 18+  
**Legacy reference:** `maguc98/text-display-ios` (read-only; never modify it).

## 1. Product boundary

Aether is a full rebuild of Yiu.fm, not a wrapper or a visual reskin. The legacy project is used to discover behavior, formats, and edge cases. All production code in this repository must be newly implemented here.

The application UI should be native. Do not embed the legacy HTML/CSS/JavaScript app in WKWebView as the primary interface. A WebView exception requires an explicit technical justification and approval.

## 2. Proposed module map

| Module | Responsibility | Candidate Apple technologies |
|---|---|---|
| AppShell | App lifecycle, navigation, scene state, dependency composition | SwiftUI, UIKit |
| DesignSystem | Cartoon-fresh palette, typography, shapes, reusable cards, motion tokens | SwiftUI, Core Animation |
| Playback | Audio source, transport, seek, queue, repeat/shuffle, playback state | AVFoundation / AVPlayer, MediaPlayer |
| LyricDomain | LRC metadata, line timestamps, word timestamps, offsets, validation | Swift value types, Foundation |
| LyricClock | Audio-time-driven synchronization; deterministic seek/pause/resume behavior | AVPlayer time observers, a small clock abstraction |
| LyricPresentation | Line/word layout, typography, transitions, accessibility | SwiftUI, Core Text, Core Animation |
| VisualizerCore | Engine protocol, lifecycle, renderer selection, resource budget | Swift protocols, UIKit lifecycle |
| SonnetEngine | Reimplementation of Sonnet visual behavior | Metal first where GPU simulation is required; Core Animation/Core Graphics for simpler layers |
| TemperaEngine | Reimplementation of Tempera visual behavior | Metal / Core Image / Core Animation, selected after effect inventory |
| LumiereEngine | Reimplementation of Lumiere visual behavior | Metal / Core Image / Core Animation, selected after effect inventory |
| PartitaEngine | Native recreation of the React + Framer Motion lyric composition | SwiftUI / Core Animation / Core Text |
| MediaLibrary | Track metadata, ordering, artwork, import state | SwiftData or SQLite metadata store |
| MediaFileStore | Original audio and imported lyric files | FileManager in Application Support; security-scoped import handling |
| ImportExport | Audio/LRC/image import, project backup and restore | UniformTypeIdentifiers, FileImporter, Codable |
| ThemeStudio | Theme presets, custom colors, background images, crop/blur/dim | SwiftData/Codable, PhotosUI, Core Image |
| Diagnostics | FPS/frame pacing, hitch detection, logs, engine lifecycle metrics | MetricKit, os.Logger, Instruments signposts |
| Settings | User preferences, per-engine tuning, accessibility and performance settings | SwiftData or typed Codable preferences |

This is a candidate decomposition. Exact persistence technology and rendering backend should be finalized after the data-schema and visual-effect audits.

## 3. Critical behavior contracts to preserve

### Audio and lyric clock

- The audio player's media time is the source of truth while a track is playing.
- LRC line timestamps and optional word timestamps must be represented separately.
- Preserve metadata fields: title, artist, album, by, and offset where present.
- Preserve positive/negative lyric offset semantics and per-track offset.
- Seeking must immediately recompute the active line and visible word state from media time; do not replay stale timer state.
- Pause/resume, app background/foreground, interruptions, route changes, and track switching must not cause timeline drift.
- Shuffle, repeat-one, repeat-all/no-loop behavior, and current-track restoration need explicit tests.
- The legacy implementation limits its main playback tick to about 30 Hz and throttles some UI paints to 100 ms. Aether should not blindly copy those limits; use event/time observers and display-paced rendering where appropriate.

### Word-level lyrics

The legacy parser supports ordinary multi-timestamp LRC and word-level tags. In word-level lines, each tag's timestamp applies to the following character sequence; consecutive characters after one tag share a timestamp. Whitespace at line edges is trimmed.

The legacy renderer also tries to ensure a character's entrance animation completes before the next character appears by shortening the entrance duration when necessary. The final character's completed entrance affects when the line's exit/fall begins.

Aether must model this as explicit timing/presentation policy rather than burying it in view code. Required regression cases:
- No next character appears before its own timestamp.
- A character entrance completes before the next character's entrance where timing allows.
- The last character is fully visible before line exit begins.
- Seeking backward/forward correctly resets/replays character visibility.
- Lines without word timestamps remain normal line-level lyrics.

### Visualizer lifecycle

The legacy VisualizerManager intentionally serializes engines: tear down active WebGL resources, wait two display frames, then mount the next engine. It forwards lyric data and media time to adapters.

Aether should preserve the product-level invariant that engines do not compete for unbounded GPU resources, but replace the WebGL-specific workaround with a native renderer lifecycle and explicit memory/render budgets. Engine switching, interruption, backgrounding, and failure fallback must be deterministic.

## 4. What the source audit confirms about the four engines

- **Sonnet:** large Pixi-based visual engine; adapter has tuning/theme/extra/effect configuration and dynamic performance-governor hooks.
- **Tempera:** Pixi-based visual engine; supports camera/glyph motion, resolution, post-processing, custom colors, depth controls, and performance presets.
- **Lumiere:** Pixi-based visual engine; supports light/fog/motes/bloom/trails/echo/line-art and render quality/resolution settings.
- **Partita:** distinct implementation built around React 19 and Framer Motion, not the same Pixi adapter pattern. Its adapter exposes layout/timing/theme/tuning configuration and uses a React root.

These are not interchangeable renderers. Each needs a visual behavior inventory and representative screenshots/video captures before native implementation begins. Do not assume every effect needs Metal; use the simplest native renderer that meets visual and performance requirements.

## 5. Persistence and migration considerations

The legacy app currently uses several storage mechanisms:
- IndexedDB database `yiufm-songs` with separate `songs` and `files` object stores.
- Local storage keys for current song, manual song order, global settings/autosave, themes, and per-engine configuration.
- A separate IndexedDB store `themeImgsV1` for theme background images.
- Project settings export/import and embedded image data in the autosave payload.

Aether should define versioned, Codable domain schemas and an explicit import path for legacy backup files. Prefer storing large audio/image assets as files and keeping metadata/references in a database; do not store large media blobs inside one settings document.

No automatic destructive migration should run. Keep originals intact, validate imported files, and make backup/restore testable.

## 6. Cartoon-fresh design system

Visual direction:
- Primary surfaces: warm cream and soft white.
- Accent palette: mint green, sky blue, butter yellow, lavender, and soft coral.
- Rounded cards and friendly iconography; restrained doodle-like illustrations.
- Motion should feel light and responsive, not bouncy everywhere.
- Maintain strong text contrast, clear hierarchy, Dynamic Type support, and reduced-motion support.
- The player and lyrics remain the visual focus; decoration must never compete with readability.

## 7. Performance and quality gates

Before optimizing, establish repeatable baselines on representative devices:
- Frame pacing and hitch count during idle, lyric transitions, word-by-word lyrics, and each visualizer.
- CPU/GPU utilization and memory high-water mark during engine changes.
- Audio/lyric synchronization error during steady playback, seeking, backgrounding, and interruptions.
- Import time and memory use for large audio files and large theme images.
- Cold launch, warm launch, and restore-library time.

Initial acceptance targets should be set after baseline measurement. Do not invent FPS or performance claims.

## 8. Open questions before implementation

- Exact legacy song metadata schema and ID3/FLAC/MP4 artwork extraction behavior.
- Full LRC variants supported, including malformed/duplicate/out-of-order timestamps.
- Exact theme/background image schema and backup compatibility.
- Per-engine settings inventory and which effects are essential versus optional.
- Full Partita visual behavior and font-fitting/layout rules.
- Whether any original engine assets can be legally/technically reused, or must be recreated from behavior descriptions.

## 9. Next work

1. Finish a per-module and per-control source audit.
2. Extract the legacy data schemas and backup format.
3. Produce a feature-parity checklist with verified source locations.
4. Capture reference behavior for the four engines and lyric timing.
5. Freeze the architecture and create the native Xcode project only after the contracts are clear.
