# Keyout — Implementation Plan

Local background-removal web app. Single `index.html` (HTML+CSS+JS, no build, no server). English UI, dark mode. Opens with double click.

## 1. Architecture

- **Core:** vanilla JS + HTML5 Canvas API with typed arrays (`Uint8ClampedArray`). No dependencies for the base tool → fully offline.
- **Optional AI mode:** lazy-loaded on demand (only when the user clicks "Auto (AI)") using `@imgly/background-removal` from CDN (ONNX/WASM under the hood). If offline or load fails → graceful fallback message; chroma mode unaffected.
- **Separation of concerns inside the file:** `state` object → `pipeline` (pure functions over pixel buffers) → `ui` (event wiring, rendering). Makes it easy to split into modules later.

## 2. Processing pipeline

The key design decision: the mask is computed as a **separate grayscale alpha buffer** (`Float32Array`, 0–1), never destructively applied to the image. Every stage reads the previous buffer:

```
image (original res, immutable)
  └─► base mask ──► edge refine (Sobel + feather) ──► brush correction layer ──► composite
```

1. **Base mask — Chroma Key:** per-pixel distance to the key color computed in **YCbCr space using only Cb/Cr** (chroma plane). Ignoring luminance makes it robust to shadows/lighting on a green/blue screen — much better than raw RGB distance. Two thresholds derived from the Tolerance slider: below `t0` → alpha 0, above `t1 = t0 + softness` → alpha 1, linear ramp in between (soft matte instead of hard cut).
2. **Base mask — AI (optional):** model returns an alpha matte at original resolution; it replaces the chroma mask and flows through the exact same refine/brush/composite stages.
3. **Edge refinement:** Sobel operator over the *mask* to find transition zones; Edge Threshold slider gates which gradients count as edges. Feathering slider applies a separable box blur (radius = feather px) **only where edges were detected** — smooths the sawtooth without eroding the whole matte.
4. **Despill (checkbox):** desaturate the key color on pixels near edges (removes green fringe). Cheap and high-impact for chroma work.
5. **Brush correction layer:** independent buffer with three states per pixel: neutral / force-restore (alpha 1) / force-erase (alpha 0), painted with a soft round brush. Because it's a separate layer, **corrections survive slider changes** and are individually undoable. Controls: Restore/Erase toggle, brush size, hardness, Undo (stroke stack), Clear corrections.
6. **Composite:** checkerboard (transparency) or solid color (second color picker) drawn first, cut-out drawn on top with the final mask as alpha.

## 3. Performance strategy

- Live preview processes a **downscaled copy** (longest side ≤ ~1400 px); slider input is coalesced with `requestAnimationFrame` so dragging stays at 60 fps even on large photos.
- **Export re-runs the full pipeline at original resolution** — output always matches the uploaded image's dimensions.
- Buffers reused between runs (no per-frame allocation). If profiling shows jank on huge images, move the pipeline into a Web Worker (code already structured as pure functions to allow this).

## 4. UI layout

- **Header:** app name, dark/light toggle (follows `prefers-color-scheme`, manual override).
- **Sidebar (top bar on mobile, responsive via CSS grid):**
  - Key color: eyedropper (click on the *original* canvas picks the color, 3×3 average) + native color input.
  - Sliders: Tolerance, Softness, Feathering, Edge Threshold — each with live numeric value.
  - Background mode: Transparent / Solid Color (+ color picker).
  - Brush tools: Restore | Erase, size, Undo, Clear.
  - "Auto (AI)" button with loading state.
- **Preview:** side-by-side Original / Result canvases (stacked on narrow screens), checkerboard behind result, shared **zoom & pan** (wheel + drag) so the brush can work precisely; cursor shows brush outline.
- **Upload:** full-window drag & drop + file button + **paste from clipboard** (Ctrl+V). Accepts PNG/JPG/WEBP.

## 5. Export

- Format dropdown: **PNG** (transparency), **JPG**, **WEBP**; quality slider for JPG/WEBP.
- JPG has no alpha → auto-flattened onto the chosen background color (white if transparent mode).
- Works as a pure **format converter** too: upload → export without keying anything.
- `canvas.toBlob()` → download named `originalname-nobg.png` etc.

## 6. Milestones

| # | Deliverable |
|---|-------------|
| M1 | Load (drop/button/paste), chroma key + tolerance/softness, live preview, PNG export at full res |
| M2 | Sobel edge refine + feathering, despill, background modes, JPG/WEBP + quality |
| M3 | Brush restore/erase with undo, zoom/pan, brush cursor |
| M4 | AI mode (lazy CDN load), dark mode polish, responsive pass, edge-case hardening (huge images, no-alpha formats) |

## 7. Risks / notes

- AI mode needs internet on first use (model download); everything else is offline.
- Very large images (>30 MP) may be slow at export — acceptable (one-time), preview stays fast via downscale.
- WEBP export support is universal in modern browsers; Safari ≥16 OK.
