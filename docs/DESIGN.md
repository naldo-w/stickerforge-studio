---
version: "1.0"
name: BatchFrame-design-system
description: "StickerForge Studio's own design system — a flat, hairline-bordered, monospace-first UI built for a dense batch-processing tool. Every surface is either raw paper (light) or near-black (dark), typography is Sometype Mono set uppercase and letter-spaced throughout, and the single accent (a pale sky-blue) is the only chromatic note in an otherwise ink-on-paper system. Corners stay square everywhere except the one deliberately rounded app icon; borders (never shadows) carry all hierarchy. The one serif display face (Faculty Glyphic) is reserved for the wordmark, empty-state prompt, and modal titles — everything functional is mono."

colors:
  paper: "rgb(244, 244, 244)"
  ink: "#17191A"
  ink-inv: "#E4E6E7"
  accent: "#C3E5F2"
  accent-ink: "#17191A"
  accent-track: "#E7F4FA"
  line: "#17191A"
  field: "rgba(10, 10, 10, .028)"
  scrim-strong: "rgba(10, 10, 10, .93)"
  checker: "rgba(10, 10, 10, .05)"
  ok: "#24a148"
  warn: "#b28600"
  err: "#da1e28"
  fail: "#ff9800"
  danger: "#c0392b"
  dark-paper: "rgb(13, 13, 13)"
  dark-ink: "#E4E6E7"
  dark-ink-inv: "#17191A"
  dark-field: "rgba(244, 244, 244, .05)"
  dark-checker: "rgba(244, 244, 244, .07)"
  dark-ok: "#42be65"
  dark-warn: "#f1c21b"
  dark-err: "#fa4d56"
  dark-danger: "#e74c3c"

typography:
  wordmark:
    fontFamily: Faculty Glyphic
    fontSize: "clamp(28px, 3.4vw, 44px)"
    lineHeight: 0.88
    letterSpacing: -0.01em
    textTransform: none
  display-lg:
    fontFamily: Faculty Glyphic
    fontSize: "clamp(42px, 7vw, 86px)"
    lineHeight: 0.88
    textTransform: none
  display-md:
    fontFamily: Faculty Glyphic
    fontSize: "clamp(42px, 5vw, 72px)"
    lineHeight: 0.88
    letterSpacing: -0.01em
    textTransform: none
  display-sm:
    fontFamily: Faculty Glyphic
    fontSize: "clamp(30px, 3.8vw, 46px)"
  subhead:
    fontFamily: Sometype Mono
    fontSize: 12px
    letterSpacing: 0.16em
    textTransform: uppercase
  body:
    fontFamily: Sometype Mono
    fontSize: 10px
    letterSpacing: 0.16em
    textTransform: uppercase
  note:
    fontFamily: Sometype Mono
    fontSize: 8px
    letterSpacing: 0.18em
    textTransform: uppercase
  label-loose:
    fontFamily: Sometype Mono
    fontSize: 8px
    letterSpacing: 0.22em
    textTransform: uppercase

rounded:
  none: 0px
  pill: 999px
  icon: 16px

spacing:
  page-pad: "clamp(24px, 4vw, 64px)"
  page-max: 1460px
  ctl-h: 36px
  cell-min: 212px
  settings-row-h: 48px
  btn-px: 16px

motion:
  fast: 0.14s
  base: 0.16s

components:
  bf-btn:
    backgroundColor: transparent
    textColor: "{colors.ink}"
    borderColor: "{colors.line}"
    typography: "{typography.body}"
    rounded: "{rounded.none}"
    height: "{spacing.ctl-h}"
    padding: "0 {spacing.btn-px}"
    fontWeight: 700
  bf-btn-accent:
    backgroundColor: "{colors.accent}"
    textColor: "{colors.accent-ink}"
    typography: "{typography.body}"
    rounded: "{rounded.none}"
  bf-btn-danger:
    backgroundColor: transparent
    textColor: "{colors.danger}"
    borderColor: "{colors.danger}"
  bf-seg-btn:
    backgroundColor: transparent
    textColor: "{colors.ink}"
    borderColor: "{colors.line}"
    typography: "{typography.body}"
    rounded: "{rounded.none}"
    height: "{spacing.ctl-h}"
  bf-seg-btn-selected:
    backgroundColor: "{colors.accent}"
    textColor: "{colors.accent-ink}"
  bf-field:
    backgroundColor: "{colors.field}"
    textColor: "{colors.ink}"
    borderColor: "{colors.line}"
    rounded: "{rounded.none}"
    height: 26px
  bf-switch:
    trackBorderColor: "{colors.line}"
    trackRounded: "{rounded.pill}"
    width: 34px
    height: 18px
  cell:
    backgroundColor: transparent
    borderColor: "{colors.line}"
    minWidth: "{spacing.cell-min}"
  cell-header:
    height: 40px
    borderColor: "{colors.line}"
    typography: "{typography.body}"
  item-head:
    borderColor: "{colors.line}"
    padding: "10px 12px 10px 16px"
  warn:
    backgroundColor: "{colors.ink}"
    textColor: "{colors.ink-inv}"
    typography: "{typography.note}"
    padding: "8px 10px"
  ok-tag:
    backgroundColor: "{colors.accent}"
    textColor: "{colors.accent-ink}"
    typography: "{typography.note}"
    padding: "2px 6px"
  info-card:
    backgroundColor: "{colors.paper}"
    borderColor: "{colors.line}"
    rounded: "{rounded.none}"
  info-card-header:
    minHeight: 38px
    backgroundColor: "{colors.field}"
    borderColor: "{colors.line}"
  modal-panel:
    backgroundColor: "{colors.paper}"
    textColor: "{colors.ink}"
    borderColor: "{colors.line}"
    maxWidth: "min(1280px, 96vw)"
    padding: 24px
  spec-table:
    borderColor: "{colors.line}"
    headerBackgroundColor: "{colors.field}"
    typography: "{typography.note}"
  stats-card:
    backgroundColor: "{colors.paper}"
    borderColor: "{colors.line}"
  ws-tab:
    backgroundColor: "{colors.paper}"
    textColor: "{colors.accent-ink}"
    progressFillColor: "{colors.accent}"
  license-icon:
    rounded: "{rounded.icon}"
    borderColor: "{colors.line}"
---

## Overview

StickerForge Studio's UI is the **BatchFrame Design System** — a flat, monospace, hairline-bordered aesthetic built for a dense, panel-heavy batch tool rather than a marketing surface. There are exactly two surfaces (`{colors.paper}` light / near-black dark) and exactly one chromatic accent (`{colors.accent}`, a pale sky-blue used identically in both themes). Everything else is ink, paper, and 1px `{colors.line}` borders.

The defining choice is **uppercase monospace everywhere functional**: body text, labels, buttons, table cells, stat rows all run `{typography.body}` (Sometype Mono, uppercase, tracked). The one serif face, Faculty Glyphic, is reserved for three things only — the wordmark, the empty-state drop prompt, and modal titles — so display type reads as a deliberate accent against an otherwise mechanical, monospaced interface.

**Key characteristics:**
- **Two-color system, one accent.** Paper/ink invert cleanly between light and dark; `{colors.accent}` is the only hue and is identical across both themes (this is why accent-backed text is pinned to `{colors.accent-ink}` rather than the theme's ink variable — see "Theme-invariant accent text" below).
- **Borders carry all hierarchy, never shadows.** Cards, cells, buttons, tables, panels — every container is a 1px `{colors.line}` hairline. The one `box-shadow` in the whole system is the settings/about modal's drop shadow.
- **Flat corners except one icon.** `border-radius: 0` is the default for every button, card, field, and panel. The single exception is the app's own rounded icon mark (`{rounded.icon}` 16px) in the License byline — called out in its own CSS comment as "the one deliberately rounded mark in the system."
- **Progress is a horizontal fill, not a bar.** Cell headers, workspace tabs, and stat rows all encode build progress as a `linear-gradient` sweep across their own background (`--progress` custom property) rather than a separate progress-bar element.
- **The pill is reserved for one control.** `{rounded.pill}` exists solely for the on/off switch track (`.bf-switch`) — the single rounded interactive control in an otherwise square system.

## Colors

### Surface
- **Paper** (`{colors.paper}`): default page/card background, light theme.
- **Ink** (`{colors.ink}`): default text and border color, light theme; becomes the dark theme's paper-equivalent inversion partner.
- **Ink Inverse** (`{colors.ink-inv}`): text-on-ink color — used on the `.warn` banner (solid ink background) and the drag-over scrim.
- **Field** (`{colors.field}`): near-transparent ink wash (2.8% light / 5% dark) — hover states, input backgrounds, table header fill.
- **Checker** (`{colors.checker}`): transparency checkerboard behind preview thumbnails (`.pv`).
- **Scrim Strong** (`{colors.scrim-strong}`): full-screen drag-and-drop overlay background.

### Accent
- **Accent** (`{colors.accent}`): pale sky-blue `#C3E5F2` — the only color note in the system. Used for: primary buttons, selected segmented-control state, switch-on state, focus rings, progress fill, active TOC dot, hairline-underlined links.
- **Accent Ink** (`{colors.accent-ink}`): text color on top of accent fills. Deliberately **theme-invariant** (always `#17191A`, never swapped for the dark-theme ink token) — because `{colors.accent}` itself doesn't change between themes, only a fixed dark ink stays readable against it as fills ramp in via progress gradients. See inline comments at the `.item-status .st` and `.stats-card-h--progress` rules.
- **Accent Track** (`{colors.accent-track}`): the unfilled portion of an accent progress gradient (a lighter tint, not `{colors.field}`) — used in cell headers and stat rows so an unbuilt cell still reads as "in this system" rather than neutral gray.

### Semantic
- **OK** (`{colors.ok}`) / **Warn** (`{colors.warn}`) / **Err** (`{colors.err}`): borrowed directly from IBM Carbon's semantic palette (green-50 / yellow-60-ish amber / red-60) for the collapsed-panel completion summary (`.item-status .st--warn/.st--err`). These are the only borrowed-from-elsewhere colors in the system and exist purely as small status accents, never as fills.

### Auxiliary
Two further tokens cover states that `{colors.warn}`/`{colors.err}` don't quite mean:
- **Fail** (`{colors.fail}`): a harder, more saturated orange than `{colors.warn}` — used specifically for "this build attempt failed" (cell/stat progress fills, over-budget stat values), as distinct from `{colors.warn}`'s softer "this is a soft advisory" meaning on the collapsed-panel summary tag. Theme-invariant like `{colors.accent}` — it only ever appears inside a fill/text pairing, never as a background needing independent per-theme contrast tuning.
- **Danger** (`{colors.danger}`): the destructive-action red for `.bf-btn--danger` and the failed-workspace-tab state (`.ws-tab[data-state="failed"]`). Does invert per theme (`{colors.dark-danger}` `#e74c3c`), unlike `{colors.fail}` — because it sits directly on the paper background as text/border and needs the brightness bump dark grounds require, the same reason `{colors.err}` inverts.

### Dark theme
Dark mode is a straight inversion toggled by adding the `.theme-dark` class to `<body>` (the theme-picker buttons carry a `data-theme` attribute purely as a data hook for the click handler — it is never read by CSS, so don't gate new dark-mode rules on a `[data-theme="dark"]` selector, only on `.theme-dark`). `{colors.paper}` ↔ near-black, `{colors.ink}` ↔ light gray, `{colors.field}`/`{colors.checker}` re-tuned for a dark ground, and `{colors.ok}/{colors.warn}/{colors.err}/{colors.danger}` swap to their brighter dark-theme equivalents. `{colors.accent}`, `{colors.accent-ink}`, and `{colors.fail}` do **not** change — that invariance is the load-bearing detail referenced throughout the CSS comments.

## Typography

### Font families
- **Sometype Mono** (fallback: IBM Plex Mono, ui-monospace, Menlo, monospace) — carries essentially the entire interface: body copy, labels, buttons, tables, stat cards, warnings. Always set uppercase with wide letter-spacing (`0.16em` body, `0.18em` note, `0.22em` loose labels).
- **Faculty Glyphic** (fallback: Georgia, Times New Roman, serif) — reserved for the wordmark, the empty-state "DROP" prompt, the drag-overlay prompt, and modal titles. Always set with `text-transform: none` — it's the one place the UI drops out of all-caps.

### Hierarchy
| Token | Size | Family | Use |
|---|---|---|---|
| `{typography.display-lg}` | clamp(42–86px) | Faculty Glyphic | full-screen drag overlay prompt |
| `{typography.display-md}` | clamp(42–72px) | Faculty Glyphic | empty-state drop prompt |
| `{typography.display-sm}` | clamp(30–46px) | Faculty Glyphic | (reserved display size) |
| `{typography.wordmark}` | clamp(28–44px) | Faculty Glyphic | masthead wordmark (`.dot` in accent) |
| modal-title | clamp(22–30px) | Faculty Glyphic | Settings / About / License modal headers |
| item-no | clamp(20–28px) | Faculty Glyphic | per-file panel index number |
| `{typography.subhead}` | 12px | Sometype Mono | item filename, section headers |
| `{typography.body}` | 10px | Sometype Mono | default UI copy, buttons, table cells |
| `{typography.note}` | 8px | Sometype Mono | meta text, warnings, captions |

### Principles
- **Uppercase + tracked is the default, not an accent.** Nearly every functional string in the app (`text-transform: uppercase`) is deliberate — it's what makes the mono type read as "system," not incidental styling.
- **Display type is the one place caps drop.** Every Faculty Glyphic usage explicitly sets `text-transform: none` — mixing case here is what signals "this is a headline, not a control."
- **The wordmark's accent dot** (`.wordmark .dot { color: var(--accent) }`) is the smallest unit of brand color in the whole system — a single character.

## Layout

- **Base grid unit isn't a fixed 4/8px scale** — spacing is mostly ad hoc per-component (`padding: 10px 12px`, `gap: 8px`, etc.), not a shared spacing token scale like Carbon's. If extending this system, match nearby components' literal values rather than inventing a spacing token.
- **Page width**: `{spacing.page-max}` 1460px soft cap, but the page itself runs at `width: 95%` with no `max-width` — it's built to fill very wide viewports (batch-processing tool, not a marketing page).
- **Panels are horizontally-scrolling cell strips**: `.panel` lays out one `.cell` per output format (`min-width: {spacing.cell-min}` 212px each) in a non-wrapping flex row inside `.panel-scroll { overflow-x: auto }` — the panel is a lane of format cards, not a grid.
- **Controls sit at a fixed 36px height** (`{spacing.ctl-h}`) across buttons, segmented controls, and the settings row — one of the few genuinely fixed layout constants in the system.

## Elevation & Depth

There is effectively **one elevation level**: flat surface + 1px hairline border. The only depth cue beyond that is the Settings/About modal's `box-shadow: 0 12px 28px rgba(10,10,10,.12)` plus a `backdrop-filter: blur` scrim — reserved for true overlays (modal, drag-and-drop full-screen overlay, toolbox modal). Nothing else in the system uses shadow.

## Shapes

- **`{rounded.none}` (0px) is the default for everything**: buttons, segmented controls, fields, cells, cards, tables, modals.
- **`{rounded.pill}` (999px)** is scoped to exactly one component: the `.bf-switch` track/thumb.
- **`{rounded.icon}` (16px)** is scoped to exactly one element: `.license-icon`, explicitly commented in the CSS as "the one deliberately rounded mark in the system."

## Components

### Buttons
- **`bf-btn`** — the default button: transparent fill, 1px `{colors.line}` border, uppercase mono label, `translateY(-1px)` lift on hover. Disabled state switches the border to dashed at 40% opacity rather than graying out the fill — "disabled controls go dashed at 40%, never a filled grey" per the inline comment.
- **`bf-btn--accent`** — solid accent fill for primary actions (e.g. export buttons).
- **`bf-btn--danger`** — border/text only, no fill, in a red (`#c0392b` light / `#e74c3c` dark) that is otherwise unused in the palette.
- **`bf-btn.download-progress`** — an export-in-progress button whose own background is the progress fill (same linear-gradient-sweep pattern as cell headers), including a struck-through failed state.

### Segmented controls & fields
- **`bf-seg` / `bf-seg-btn`** — button-group style toggle (e.g. scale mode, fps mode); selected segment gets the accent fill.
- **`bf-field`** — text/number input: `{colors.field}` wash background, 26px height, right-aligned in most layouts.
- **`bf-switch`** — the sole pill-shaped control; thumb is ink, track becomes accent-filled when checked, thumb becomes accent-ink to stay legible.

### Panels & cells
- **`.item` / `.item-head`** — one collapsible section per uploaded file; the collapsed header shows a compact per-format completion summary (`.item-status .st`) as small accent-progress tiles.
- **`.cell` / `.cell-h` / `.cell-b`** — one output-format card per cell inside a file's panel; cell header is a progress-gradient bar that fills as that format builds, switches to a striped "processing" animation mid-build, and turns `{colors.fail}` orange on failure.
- **`.stats-card` / `.stats-row`** — the Output Stats view's per-format summary card, same progress-gradient language as `.cell-h`.
- **`.ws-tab`** — workspace-view format tabs (Tool Box results), same progress-fill idiom again — this repetition across three unrelated components (`.cell-h`, `.stats-card-h`, `.ws-tab`) is a deliberate, explicitly-commented pattern: progress is always "this element's background fills toward accent," never a separate bar.

### Cards & tables
- **`.info-card`** — used for About/License content blocks: hairline border, `{colors.field}`-filled header bar, same visual language as `.s-card` in Settings (explicitly cross-referenced in the CSS).
- **`.spec-table`** — the platform-spec comparison table: hairline cell borders, `{colors.field}` header row, format names shown as small accent-filled tags (`.fmt`).

### Modals
- **`.modal` / `.modal-panel`** — Settings / About / License live in one tabbed modal (`.tab-page`); the toolbox has its own separate modal (`#toolboxModal`) reusing the same panel styling.

### Navigation
- **`.toc`** (scroll table of contents) — a slim rail of dots pinned to the left edge that expands to show labels on hover/focus; the active section's dot fills accent and, on hover-expansion, morphs from a circle into a short vertical bar via transitioning `border-radius`/`width`/`height` — one of the only non-trivial motion sequences in the system.

## Do's and Don'ts

### Do
- Keep every functional string uppercase mono (`{typography.body}`/`.note`) unless it's one of the three sanctioned Faculty Glyphic spots (wordmark, empty-state prompt, modal titles).
- Use a 1px `{colors.line}` hairline for any new container's hierarchy — never introduce a drop shadow outside the modal/overlay case.
- Pin any text drawn over an accent-filled progress background to `{colors.accent-ink}`, not the theme's ink variable — the accent hue itself is theme-invariant, so the text must be too.
- Keep disabled controls dashed-border-at-40%-opacity, not solid-gray-fill.
- Reuse the progress-gradient-on-own-background idiom for any new "this thing is building" indicator rather than adding a separate progress bar element.

### Don't
- Don't round corners on new buttons, cards, cells, or fields — `{rounded.none}` is the rule; `{rounded.pill}` and `{rounded.icon}` are closed sets of exactly one component each.
- Don't introduce a second chromatic accent. `{colors.accent}` carries every piece of interactive/brand color; `{colors.ok}/{colors.warn}/{colors.err}` exist only as small semantic tags, not fills.
- Don't let `{colors.accent}` or `{colors.accent-ink}` vary between light/dark theme — every other color pair inverts; these two are the fixed point the rest of the palette is built around.
- Don't mix case on mono UI copy — lowercase-with-normal-tracking reads as an unstyled fallback in this system, not a valid alternate state.

## Responsive Behavior

- **768px**: `.wrap` padding tightens; `.panel` drops its computed `min-width` so the format-cell strip scrolls within its own region instead of forcing page-width overflow.
- **900px**: collapsed-panel completion summaries (`.item-status`) hide entirely (lead + actions remain); stats grids collapse from multi-column to single-column; the TOC rail narrows its max-height.
- **720px**: `.info-grid` (About/License two-up cards) collapses to a single column.

## Known Gaps

- There is no formal spacing token scale (no `{spacing.xs}/{spacing.sm}/...` ramp) — component padding/gap values are set ad hoc per rule rather than drawn from a shared scale. Anyone extending the system should match the nearest existing component's literal values rather than inventing a new token tier.
- ~~The failure-state orange and danger red were hardcoded hex literals repeated across 9 call sites, with no dark-theme variant for the danger red.~~ Fixed: both are now `{colors.fail}`/`{colors.danger}` tokens in `:root`/`.theme-dark`. This also fixed a dead `[data-theme="dark"] .bf-btn--danger` selector that never matched anything (dark mode toggles via the `.theme-dark` body class, not a `data-theme` attribute) — the danger button silently kept its light-theme red in dark mode until now.
- This document was reverse-engineered from the CSS in `index.html`'s `<style>` block (lines ~27–1504) rather than authored ahead of the implementation — component names above (`bf-btn`, `s-card`, etc.) are the actual CSS class names, not abstracted design-tool tokens, so cross-check against `index.html` directly for any pixel-level detail not called out here.
