# Keyout

A background remover that runs entirely in the browser. It's a single
`index.html` file (HTML + CSS + JS), no build step and no server: just
double-click to open it, and it works offline.

The name combines two ideas that summarize the app: **chroma *key*** (the
core technique used to separate the subject from the background) and
**cut *out*** (the result — the cropped-out image).

## What it does

- Removes the background from a photo using **chroma key** (green/blue or
  any color picked with the eyedropper), with tolerance and softness
  controls.
- Optional **Auto (AI)** mode: uses `@imgly/background-removal`, lazy-loaded
  from a CDN, for photos without a solid-color background. It only needs
  internet on first use (to download the model); everything else works
  offline.
- Edge refinement (Sobel) with feathering, color despill, and a **brush**
  layer to manually restore/erase areas (with undo).
- Final background: transparent or solid color.
- Exports to PNG (with transparency), JPG, or WEBP, always at the original
  resolution of the uploaded image.
- Editable output file name, with an optional "-keyout" suffix.
- Load images via drag & drop, file picker, or paste from clipboard
  (Ctrl+V).
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
