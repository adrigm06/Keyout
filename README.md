# Keyout

A background remover that runs entirely in the browser. It's a single
`index.html` file (HTML + CSS + JS), no build step and no server: just
double-click to open it, and it works offline.

The name combines two ideas that summarize the app: **chroma *key*** (the
core technique used to separate the subject from the background) and
**cut *out*** (the result — the cropped-out image).

## What it does

### Background removal

- Removes the background from a photo using **chroma key** (green/blue or
  any color picked with the eyedropper), with tolerance and softness
  controls. Multiple key colors can be added at once (e.g. the background
  plus a glow or shadow tint) — each gets its own chip and can be removed
  individually.
- Optional **Auto (AI)** mode: uses `@imgly/background-removal`, lazy-loaded
  from a CDN, for photos without a solid-color background. It only needs
  internet on first use (to download the model); everything else works
  offline.
- **Simple / Advanced mode toggle.** Simple mode applies a tuned set of
  defaults automatically; Advanced mode exposes every control below.
- Edge refinement (Sobel-based): shrink/grow the matte edge, feathering,
  and an edge-detection threshold, plus color despill to remove tinted
  fringes.
- Matte cleanup tools: protect interior (erase only from the border
  inward), keep main subjects only (drop small disconnected pieces),
  solidify (push semi-transparent gray blotches to solid), and
  speck/hole cleanup with adjustable size.
- A **correction brush** (restore / erase) with adjustable size and
  hardness, plus undo and clear.
- Final background: transparent or solid color.
- Exports to PNG (with transparency), JPG, or WEBP, always at the original
  resolution of the uploaded image, with adjustable quality for JPG/WEBP.
- Editable output file name, with an optional "-keyout" suffix.
- Load images via drag & drop, file picker, or paste from clipboard
  (Ctrl+V).

### Batch format converter

- A **"🔁 Convert Format"** button next to the Keyout logo opens a
  standalone tool for converting many images to another format at once —
  independent of the background-removal pipeline.
- Add any number of images via drag & drop or file picker.
- Pick the target format (PNG, JPG, or WEBP) per image, or set one format
  and apply it to the whole batch in one click, with a shared quality
  slider for JPG/WEBP.
- One "Convert & Download All" button converts every image and downloads
  them automatically, one after another.

### General

- Light/dark mode, responsive layout.

## Usage

Open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari
≥16). No installation or dependencies required for the base mode.

## Project structure

- `index.html` — the entire app (markup, styles, and logic).
- `PLAN.md` — design document covering the internal architecture, the
  processing pipeline, and development milestones.

## Architecture (summary)

The pipeline never modifies the original image: it computes an alpha mask
separately and refines it through independent stages.

```
original image ─► base mask ─► edge refinement ─► brush corrections ─► final composite
```

See [`PLAN.md`](./PLAN.md) for more detail.
