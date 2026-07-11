# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

StickerForge Studio is a **single static HTML file** (`index.html`, ~7.3k lines) that converts pixel-art GIFs into messaging-platform sticker formats, and Telegram `.tgs` (gzipped Lottie) stickers back into raster formats. Everything — markup, CSS, and JS — lives in that one file. There is no build step, no server, no bundler, no package manager, and no dependencies to install. It runs entirely client-side; user files are never uploaded.

The only external network calls the page makes: Google Fonts (Faculty Glyphic, Sometype Mono), and — only when a `.tgs` file is loaded — a one-time CDN fetch of `lottie-web` to render Lottie frames to canvas.

## Running / developing

There is no build or install step. Either:

```sh
open index.html                 # or double-click it
```

or serve it (needed if you hit CORS/file:// restrictions while testing):

```sh
python3 -m http.server 4173
# → http://localhost:4173
```

There is no test suite, linter, or CI configured in this repo — validate changes by opening the page in a browser and exercising the affected pipeline (drop a GIF or `.tgs` file and check the relevant output panel/cell).

Because everything is one file, use line-anchored search (`grep -n`) rather than assuming stable line numbers — the file is large and line numbers shift as edits land elsewhere in the file.

## Big-picture architecture

`index.html` contains exactly two `<script>` blocks:

1. **`PixelCore`** (first `<script>`, isomorphic — runs under `window` or `module.exports`) — the core, UI-agnostic encoding/decoding pipeline: GIF LZW decode + disposal-method compositing, integer nearest-neighbor scaling, APNG/PNG encoding (palette + delta + adaptive filter via `CompressionStream` deflate), Lottie/TGS building (pixel → vector rect → gzip), animated WebP muxing, and shared helpers (`quantizeFrames`, `resampleFramesToFps`, `fit3sDelays`, `boundingBox`/`cropRgba`/`diffBox`). Exposed as `global.PixelCore`. Shared canvas constant `CANVAS_SIZE = 512` (aliased as `CS` in the UI script).
2. **UI logic** (second `<script>`) — DOM wiring, panel construction, per-file state, settings, scheduling/debouncing, caching, and export. Consumes `PixelCore` via `const C = window.PixelCore`.

### Two independent pipelines, shared encoders

| | Input | Per-item state array | Panel builders |
|---|---|---|---|
| **GIF pipeline** | pixel-art GIF | `items[]` | `panelHtml` / `addFiles` |
| **TGS pipeline** | Telegram `.tgs` (gzipped Lottie) | `tgsItems[]` | `_tgsPanelHtml` / `_addTgsFiles` |

Both pipelines drive the same underlying encoders (`encodeApng`, `muxAnimatedWebp`, `quantizeFrames`, etc.) but have separate settings objects, caches, schedulers, and reset logic — when changing behavior that should apply to "both directions," check both pipelines rather than assuming shared code covers it.

### Output formats

Six outputs, built per platform target:

| Output | Target platform | Canvas | Notes |
|---|---|---|---|
| `512APNG` | Signal, iMessage | 512×512 | ≤300 KB budget |
| `320APNG` | Discord | 320×320 (configurable) | ≤512 KB budget |
| `LINE・APNG 270` | LINE | 270×270 | ≤1 MB, 5–20 frames, 1–4 loops, ≤4s total (dedicated preset, not a reuse of another canvas) |
| `TGS` | Telegram | 512×512 | ≤64 KB budget (post-gzip); pixels emitted as vector rects |
| `WEBP` | WhatsApp | 512×512 | lossless VP8L by default; ≤100 KB budget |
| `PNG` (frame / pad) | static sticker / thumbnail | 512×512, padded or cropped | no size budget |

### Key pipeline behaviors worth knowing before touching encoding code

- **GIF decode** composites every frame to a full RGBA buffer (disposal methods 2/3 resolved against the previous frame) — downstream code always receives complete, ghost-free frames, never deltas.
- **Scaling** is always integer nearest-neighbor (`maxFitScale` = largest integer scale that fits the target canvas), then centered on a transparent square canvas.
- **TGS encoding**: same-color opaque runs become vector rects (`frameRects`), optionally merged; two stacked copies of the same precomp are rendered with the second nudged by `0.4px` diagonally to fill anti-aliasing hairline seams between adjacent rects.
- **AUTO-FIT** (TGS pipeline, `_encodeUnderBudget`): shrinks output to fit a platform's byte budget by first walking an fps ladder (`[native, 60, 30, 20, 10]`), then a color/quality ladder — fps changes before color changes, always.
- **Debounced rebuild + cache invalidation**: every settings change clears the relevant per-item cache (`it.lastApng`/`lastDc`/`lastWebp`/etc.) synchronously, then a debounced (250–400ms) rebuild runs with a monotonically incrementing token so stale async results are discarded if settings change again mid-build. Exports read from cache first and only rebuild on a cold cache.
- **`quantizeFrames`/`resampleFramesToFps`** never mutate their input and are no-ops when the request wouldn't reduce fidelity (e.g. resampling never upsamples fps).
- Settings persist to `localStorage` (key `stickerforge_studio_v1`) via `saveSettings`/`loadSettings`; this is separate from the user-facing Export/Import JSON feature — the two persistence paths are not guaranteed to carry the same fields, so check both when changing what state needs to survive a reload vs. an explicit export.

### `docs/pipeline-logic.md`

A detailed (Traditional Chinese) deep-dive into the GIF and TGS pipelines, written by cross-checking three parallel sub-agent extractions against the source. It's a useful reference for pipeline internals and historical known-issues, but **line-number references inside it drift** (it says so itself) and some of the issues it lists have since been fixed by later commits (e.g. Export/Import losing TGS settings, APNG duplicate-frame dedup, LINE's dedicated 270 canvas were all called out as gaps there and have since landed) — verify against current code rather than trusting it as ground truth.

### `docs/DESIGN.md`

The canonical design-system reference for this app's own UI — the "BatchFrame Design System" (colors, type scale, components, spacing) reverse-engineered from the CSS custom properties and class rules in `index.html`'s `<style>` block (`--paper`, `--ink`, `--accent`, `.bf-btn`, `.cell`, etc.). Treat it as the default `design.md` for any future design work on this app; cross-check against `index.html` directly for pixel-level detail it doesn't call out.

## Licensing

GPLv3 (see `LICENSE`). Keep the license header in `index.html` intact when editing.
