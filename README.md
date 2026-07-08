# StickerForge Studio

A single-file, fully client-side web tool that converts **pixel-art GIFs** into the
sticker formats each messaging platform demands — and converts **Telegram TGS
stickers** back into raster formats for use on every other platform.

No build step, no server, no upload. Open `index.html` and work.

---

## Why

Pixel-art animations are usually drawn as GIFs, but no major messaging platform
accepts GIF stickers directly — each demands its own format, canvas and size
limit. Converting by hand means juggling several command-line tools and losing
crisp pixels along the way.

StickerForge Studio does it in the browser with nearest-neighbour scaling, so
pixels stay square: no ghosting, no resampling blur, original frame timing
preserved.

## What it does

**GIF converter** — drop pixel-art GIFs, get one independent processing panel per
file, with live animated previews for every output format:

| Output | Target | Canvas |
| --- | --- | --- |
| `APNG` | Signal, LINE, iMessage | 512 × 512 |
| `320APNG` | Discord | 320 × 320 (configurable) |
| `TGS` | Telegram | 512 × 512 |
| `WEBP` | WhatsApp | 512 × 512 |
| `PNG` | static sticker / thumbnail | 512 × 512, padded or cropped |

**TGS converter** — drop Telegram `.tgs` stickers, get raster outputs for every
other platform: 512APNG, 320APNG, WEBP, and single-frame PNG (first / last / mid
/ custom, padded or cropped).

**Tool Box** — one-shot mini converters for single-format jobs, without opening a
full panel.

## Platform specs

| Platform | Animated format | Canvas | Size limit |
| --- | --- | --- | --- |
| [Telegram](https://core.telegram.org/animated_stickers) | TGS (Lottie) | 512 × 512 | ≤ 64 KB |
| [Signal](https://support.signal.org/hc/en-us/articles/360031836512-Stickers) | APNG | 512 × 512 | ≤ 300 KB (max 3 s) |
| [Discord](https://support.discord.com/hc/en-us/articles/4402687377815-Tips-for-Sticker-Creators-FAQ) | APNG | 320 × 320 | ≤ 512 KB |
| [WhatsApp](https://github.com/WhatsApp/stickers/wiki/Animation) | Animated WebP | 512 × 512 | ≈ ≤ 100 KB |
| [LINE](https://creator.line.me/en/guideline/animationsticker/) | APNG | ≤ 320 × 270 | see official guideline |
| [iMessage](https://developer.apple.com/documentation/messages) | APNG | 300–618 px | Xcode sticker pack |

## Usage

1. **Add** — drop GIFs or `.tgs` files anywhere on the page, or use the Add
   buttons. One panel per file.
2. **Review** — each panel shows source info and live animated previews for every
   output format.
3. **Tune** — set defaults in Settings; override scale, FPS, frame range, loops or
   quality inside any panel.
4. **Export** — per format, per file, or all at once as a ZIP.

## Privacy

All conversion runs locally in the browser. **Your files are never uploaded.**
The only network requests the page makes are:

- Google Fonts (Faculty Glyphic, Sometype Mono)
- for TGS *input* only: a one-time `lottie-web` fetch from a CDN

## Running locally

It is one static HTML file. Either open `index.html` directly, or serve it:

```sh
python3 -m http.server 4173
# → http://localhost:4173
```

## Under the hood

- **APNG** — full-frame reconstruction (GIF disposal methods resolved, so no
  ghosting), timing preserved, transparent centered canvas, palette + delta +
  adaptive-filter compression.
- **TGS** — pixels emitted as vector rects, one precomp reused across 2 layers.
  The second copy is nudged by 0.4 px so the overlap fills the hairline seams
  that anti-aliased Lottie renderers show between adjacent rectangles.
- **WEBP** — lossless VP8L delta frames with duplicate-frame dedup (lossy VP8
  optional).
- **PNG** — palette + adaptive filter, transparent background.
- **TGS input** — `lottie-web` renders each frame to an off-screen canvas at
  target size; frames are then assembled into the raster formats above.

## Tools & acknowledgements

Built on the methods of:

- [pixelart2tgs](https://github.com/sliva0/pixelart2tgs) by **sliva0** — the TGS
  vector-rect approach
- [apngasm](https://github.com/apngasm/apngasm) — APNG delta frames
- [oxipng](https://github.com/shssoichiro/oxipng) — PNG adaptive filtering
- [libwebp](https://github.com/webmproject/libwebp) — lossless VP8L, via the
  browser's own encoder
- [Lottie](https://airbnb.io/lottie/) — the animation format behind TGS

Type: [Faculty Glyphic](https://fonts.google.com/specimen/Faculty+Glyphic) and
[Sometype Mono](https://fonts.google.com/specimen/Sometype+Mono) via Google
Fonts. UI: BatchFrame Design System.

Thanks to the open-source community for making local, serverless tools like this
possible.

## Author

**Naldo Wong**
[naldo.design](https://naldo.design) · [@n_do.w](https://www.instagram.com/n_do.w) · [@naldo.design](https://www.instagram.com/naldo.design)

## License

Copyright © 2026 Naldo Wong.

This program is free software: you can redistribute it and/or modify it under
the terms of the **GNU General Public License v3.0** as published by the Free
Software Foundation, either version 3 of the License, or (at your option) any
later version.

You may study, share and adapt this tool — derivative works must remain under
the same license. It is distributed in the hope that it will be useful, but
**without any warranty**; without even the implied warranty of merchantability
or fitness for a particular purpose.

See [`LICENSE`](LICENSE) for the full text, or
<https://www.gnu.org/licenses/gpl-3.0.html>.
