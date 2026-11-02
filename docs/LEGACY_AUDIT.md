# Legacy Yiu.fm audit

**Reference repository:** https://github.com/maguc98/text-display-ios  
**Reference branch inspected:** `main`  
**Audit status:** Initial source-level inventory; not yet a complete behavioral audit.  
**Safety boundary:** This repository is read-only reference material. Do not modify it.

## 1. Repository structure observed

The recursive Git tree returned 81 entries (directories and files), with no truncation.

- App shell: `TextDisplayApp.swift`, `WebFilePicker.swift`
- Main UI: `index.html` (~299 KB in Git tree metadata), `style.css` (~183 KB)
- Main application logic: `APP/` with player, timeline, renderer, storage, menus, import, diagnostics, and engine adapters
- Visual engines and supporting runtimes:
  - Sonnet: `APP/sonnet/sonnet.js` (~3.29 MB), `sonnet-adapter.js`
  - Tempera: `APP/tempera/`, `tempera.js`, `tempera-adapter.js`
  - Lumiere: `APP/lumiere/`, `lumiere.js`, `lumiere-adapter.js`
  - Partita: `APP/P1.js`, `partita-adapter.js`, `partita-vendor.js`, `partita-connect.js`
- Shared rendering/runtime code includes PixiJS assets and `visualizer-manager.js`
- Native asset catalog: `Assets.xcassets/AppIcon.appiconset/`
- Build configuration: `project.yml`, `project.yaml`, and GitHub Actions workflows

## 2. Confirmed application hosting model

The current `TextDisplayApp.swift` creates a `WKWebView`, loads bundled `index.html` using `loadFileURL`, and registers a script-message bridge named `yiuMusic`.

The bridge exposes:
- `list`: list files in the app's Documents/Music directory
- `text`: read text files, trying UTF-8, UTF-16 with BOM, then GB18030
- `read`: read bounded chunks of a file and return Base64 data

The native file picker source uses PHPicker for image selection and UIDocumentPicker for other accepted file types. Its comments note the WKWebView open-panel delegate path is available on iOS 18.4+; lower versions use the system's normal file input behavior.

The inspected `project.yml` sets iOS 15.0 as deployment target. Aether's target is iOS 18+ and its intended app UI is native, not a WKWebView wrapper.

## 3. Feature areas visible in the current UI/source

These are confirmed as present in the inspected source. Detailed behavior and edge cases still need per-module review.

### Playback and lyrics
- Audio import and playback
- LRC import and lyric display
- Word-level timing data (`WT`) alongside line timestamps (`TS`)
- Per-song lyric offset
- Playback modes, shuffle, repeat/no-loop behavior
- Global timeline, seek interaction, current/total time display
- Mini-player and music/text mode controls
- Song library with IndexedDB persistence in the legacy web implementation
- iOS Documents/Music directory integration

### Text and visual presentation
- Multi-line/card lyric editing and adding lines
- Multiple text animation styles, including scatter/reassemble, starfield, fade, typewriter, poster layout, and per-character effects
- Text size, speed, animation speed, shadow, RGB separation, stroke, glow, color, margins, weight, line spacing, alignment, and vertical positioning controls
- Background presets, gradients, solid colors, image backgrounds, blur/dimming, crop/zoom, and transitions
- Splash/launch cover customization
- Theme picker and theme editor
- Custom visual settings for each engine

### Visual engines
- Four named engines: Sonnet, Tempera, Lumiere, Partita
- Engine adapters and a visualizer manager
- Source comments in `visualizer-manager.js` describe serialized switching, teardown, waiting two frames, then mounting the next engine to avoid concurrent WebGL contexts
- Engine-specific quality/resolution/performance and animation controls exist in the inspected UI/source

### Library, files, and persistence
- Local song library and metadata
- Batch local import and duplicate handling
- Import/export project data
- Local storage for UI/settings state
- IndexedDB for audio/song storage
- Storage quota display and local import management
- Native file and music directory integration

### Tools and diagnostics
- Performance optimization toggle
- Real-time FPS display
- Stutter/lag detection
- Beat visualization
- System logs and error/rejection capture
- Developer tools, command console, keyboard/extra commands
- Low-power mode and global timeline controls
- Changelog and splash-cover tools

## 4. Architecture implications for Aether

The existing app's native shell is thin; most product behavior lives in a large HTML/CSS/JavaScript surface. A true native rebuild must explicitly account for:

1. Audio playback and media-session/background behavior.
2. A lyric timeline model that supports both line-level and word-level timestamps, seeking, offsets, and deterministic synchronization.
3. Four distinct rendering engines with a shared lifecycle contract.
4. A native local media library, metadata, import pipeline, persistence, and backup/export format.
5. A customizable theme/background system and a new cartoon-fresh design system.
6. Diagnostics and performance instrumentation that can be tested rather than merely displayed.

Do not assume that translating each JavaScript file one-for-one into Swift is the right architecture. Extract domain behavior and contracts first, then design native modules around them.

## 5. Known migration risks / questions

- Partita's full rendering behavior and source dependencies need deeper review; its adapter alone does not define the complete engine.
- Sonnet, Tempera, and Lumiere include substantial Pixi/WebGL-oriented code. Their visual behavior must be mapped before selecting Core Animation, Core Graphics, Core Text, or Metal implementations.
- The exact song-library schema, project export format, theme format, and compatibility requirements need to be documented from source.
- Existing lyric timing and animation edge cases must be preserved, especially word timing, complete entrance/fall animations, camera-follow behavior, and quality controls.
- The visible UI inventory is not proof that every feature has been fully understood or tested.

## 6. Next audit tasks

- [ ] Inspect every script module and map its public responsibilities.
- [ ] Document every settings panel and control, including persistence keys and defaults.
- [ ] Trace audio import, LRC parsing, word timing, playback clock, seeking, and song switching.
- [ ] Trace each engine from UI selection through adapter lifecycle to rendering implementation.
- [ ] Inspect asset catalog and embedded visual assets.
- [ ] Document data schemas and import/export compatibility.
- [ ] Produce a feature-parity matrix and native migration plan before large-scale implementation.
