# Legacy feature parity checklist — source-backed inventory

**Scope:** second-pass inventory of the legacy `main` branch.  
**Status:** Partial; each checkbox is a work item for Aether, not a claim that Aether already implements it.  
**Reference:** https://github.com/maguc98/text-display-ios

## Playback and lyric synchronization

Sources: `APP/player.js`, `APP/timeline.js`, `APP/song-library.js`, `APP/mini-player.js`, `APP/ios-music.js`

- [ ] Audio file import, source lifecycle, play/pause, seeking, duration/current-time display.
- [ ] Line-level LRC parsing, including multiple timestamps on one line.
- [ ] LRC metadata: `ti`, `ar`, `al`, `by`, and `offset`.
- [ ] Word/character timestamps, with timestamps applying to following text segments.
- [ ] Per-song lyric offset and seek alignment.
- [ ] Deterministic line selection from media time, including backward seeks.
- [ ] Word reveal state and animation-duration fitting.
- [ ] Line exit/fall timing after the final word's entrance completes.
- [ ] Playback modes: ordered, shuffle, repeat-one; no-loop/end-of-list behavior.
- [ ] Song queue/list order, current-song restoration, previous/next track.
- [ ] Lyrics-only and music-player presentation modes.
- [ ] Mini-player synchronization and artwork/title presentation.
- [ ] Global lyric timeline, scrub/seek, long-press magnifier.
- [ ] Playback interruption, audio route changes, background playback, lock-screen controls (native implementation needs separate validation; do not infer full support from the current source).

## Song library and media import

Sources: `APP/song-library.js`, `APP/local-import.js`, `APP/ios-music.js`, `TextDisplayApp.swift`, `WebFilePicker.swift`

- [ ] Local song metadata list, add/remove, manual ordering, current item.
- [ ] Audio bytes stored separately from song metadata in the legacy IndexedDB model.
- [ ] Duplicate detection and batch import progress/results.
- [ ] Audio metadata/artwork extraction:
  - [ ] MP3 ID3v2.2/v2.3/v2.4 APIC/PIC artwork
  - [ ] FLAC PICTURE blocks
  - [ ] MP4/M4A `covr` artwork
- [ ] Artwork normalization: bounded artwork image and small list thumbnail.
- [ ] External cover image import and per-song cover selection/reset.
- [ ] LRC file import and association with audio.
- [ ] Native Documents/Music directory scan/read bridge in the old wrapper.
- [ ] Filename normalization and matching for paired audio/LRC/metadata imports.
- [ ] Permission-safe document import and copy semantics.

## Lyrics and visual presentation

Sources: `APP/renderer.js`, `APP/poster.js`, `APP/cards.js`, `APP/textbar.js`, `APP/stage.js`, `style.css`

- [ ] Multiple lyric cards/lines, text editing, add/remove/reorder.
- [ ] Per-line size, color, animation, and duration overrides.
- [ ] Global typography: size, weight, family/style, alignment, line height, margins, spacing, vertical position.
- [ ] Animation speed and global UI animation speed.
- [ ] Text shadow, stroke, glow, RGB edge effects, and color controls.
- [ ] Animation families identified in persistence validation: scatter, star, pan, fade, type, bounce, flip, focus, wave, glitch, poster.
- [ ] Poster mode and its per-line word fitting/exit behavior.
- [ ] Idle/song-title display and no-lyrics state.
- [ ] Layout modes, stage overlays, status indicators, and compact player.
- [ ] Reduced-motion/accessibility equivalents in Aether.

## Four visual engines

Sources: `APP/sonnet-adapter.js`, `APP/tempera-adapter.js`, `APP/lumiere-adapter.js`, `APP/partita-adapter.js`, `APP/visualizer-manager.js`, engine source folders.

- [ ] Sonnet: scene behavior, lyric-driven animation, tuning, theme, extra options, RGB edge effects, dynamic performance governor.
- [ ] Tempera: camera intensity, glyph motion, whole-line mode, settle stretch, color mode, depth controls, texture resolution, post-processing, custom colors, optional blocks/decor/corner marks.
- [ ] Lumiere: lighting, audio response, fog, dark field, motes, bloom/text bloom, trails, echo/decay, line art, bokeh, theme color mix, render quality, resolution.
- [ ] Partita: React/Framer Motion visual behavior recreated natively; semantic layout, stagger timing, guide lines, font style/weight, animation intensity, text visibility, lyric font scale, per-line fitting.
- [ ] Engine switching lifecycle, async readiness, failure fallback, suspend/resume, resource teardown.
- [ ] Global and per-engine render quality/resolution settings.
- [ ] Reference capture pack for each engine at representative lyrics and settings.

## Themes, backgrounds, and customization

Sources: `APP/theme-picker.js`, `APP/theme-editor.js`, `APP/background.js`, `APP/storage.js`

- [ ] Built-in theme list and custom theme naming/editing/deletion.
- [ ] Per-theme background mode: gradient, solid, image.
- [ ] Four gradient colors, angle, animated flow.
- [ ] Image crop/aspect mode, pan/zoom, blur, dim.
- [ ] Background transition behavior and no-flash handoff.
- [ ] Persist theme image assets separately from theme metadata.
- [ ] Splash/cover customization.
- [ ] Theme-specific engine colors and tuning.
- [ ] Theme backup/restore and orphan-image cleanup.

## Data, tools, and diagnostics

Sources: `APP/storage.js`, `APP/devtools.js`, `APP/sys-log.js`, `APP/cmd-console.js`, `APP/beat.js`, `APP/bravais.js`, `APP/changelog.js`

- [ ] Versioned project settings backup/export and validated import.
- [ ] Autosave and restore.
- [ ] Performance mode and render resolution controls.
- [ ] FPS/frame pacing display and hitch detection.
- [ ] Error and unhandled-rejection logging equivalent.
- [ ] System log viewer and export/copy diagnostics.
- [ ] Beat-reactive visual effects.
- [ ] Bravais full-screen wall/alternate presentation mode.
- [ ] Developer command console and related tools (decide which are end-user features versus internal diagnostics).
- [ ] Changelog/release notes.

## Known legacy data stores and keys observed

- IndexedDB `yiufm-songs`: object stores `songs` (key path `id`, auto-increment) and `files` (key path `id`).
- IndexedDB `themeImgsV1`: object store `img`; image assets are referenced from theme metadata using `imgId`.
- Local storage `song_cur`: current song ID.
- Local storage `song_order`: manual song ID order.
- Local storage `td_autosave`: app state/backup snapshot.
- Local storage `themeCfgV1`: theme editor state.
- Local storage `themesV1`: theme names/preset list.
- Local storage `sonnetCfg.v1`, `temperaCfg.v1`, `folia.lumiere.cfg`, `partitaCfgV3`: engine configuration.
- Local storage `qt_res_mul`: global render resolution multiplier.
- Local storage `performanceMode`: performance mode preference.
- Local storage `uiSpd`: UI animation speed.

This is a discovered subset, not a complete key inventory. Continue scanning the remaining modules before designing a legacy backup importer.

## Explicit non-goals for the first native milestone

- Reproducing the old HTML/CSS DOM structure.
- Copying large bundled JS/Pixi/React/Framer runtime blobs into the Aether app.
- Keeping every developer-only console feature in the consumer-facing interface.
- Claiming visual parity based only on matching engine names; parity needs behavior and reference captures.
