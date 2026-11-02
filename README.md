# Yiu.fm · Aether

> A fresh, native iOS rebuild of Yiu.fm — with a soft, playful, cartoon-inspired visual identity.

## Project principles

- **Aether is a rebuild, not a rename.** Reimplement the product in a clean native iOS codebase.
- **The legacy repository is read-only reference material.** Never push changes to `maguc98/text-display-ios`.
- **Feature parity matters.** Preserve the useful capabilities and distinctive behaviors of the original product while redesigning the presentation.
- **Native-first architecture.** Target iOS 18+ and use Swift, SwiftUI/UIKit, Core Animation, Core Text, AVFoundation, and Metal where each is appropriate.
- **No WebView as the application UI.** Any exception must be explicitly justified and approved; the goal is a genuinely native app, not a WebView shell.
- **Performance is a product requirement.** Measure rendering, audio/lyric synchronization, memory, and frame pacing throughout development.

## Visual direction

A clean, cheerful, cartoon-fresh style: mint, cream, sky blue, and soft lavender; rounded shapes; small friendly illustrations; gentle motion; readable typography; and restrained decorative details. Cute, not childish.

## Rebuild workflow

1. Audit the legacy app's complete feature set, assets, data formats, and interaction rules.
2. Define the native architecture and migration map.
3. Implement the foundation and core playback/lyric timeline.
4. Rebuild the four visual engines: Sonnet, Tempera, Lumiere, and Partita.
5. Rebuild customization, local library/import, persistence, and diagnostics.
6. Validate feature parity, accessibility, performance, and iOS 18+ behavior.

## Current status

- [x] New repository located and verified.
- [x] Confirmed the repository is empty before the first project commit.
- [x] Read-only inspection of the legacy repository started.
- [ ] Complete legacy feature and asset audit.
- [ ] Native architecture decision record.
- [ ] Native app implementation.

See [the legacy audit](docs/LEGACY_AUDIT.md) for verified findings and open questions.
