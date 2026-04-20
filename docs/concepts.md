# Concepts

A short tour of the data model and UI so the rest of the docs make sense.

## Layout asset

The core object you create is a **Led Layout** asset (`ULedLayoutAsset`) —
a file that lives in your project's *Content/* folder. Each layout:

- Is fixed to either **2D** or **3D** mode (you pick at creation time).
- Contains one or more **screens**.
- Has an **Output Directory**: the content folder where generated meshes /
  textures / materials land.

To create one: *right-click in Content Browser → Miscellaneous → Led Layout*.
A modal appears — pick **2D Layout** or **3D Layout**.

## Screen

A **screen** (`FLedOutputScreen`) represents one LED surface. Properties:

- **Name** — your label; affects generated asset names.
- **Screen Width / Height** — the target LED pixelmap resolution in pixels
  (e.g. `3840 × 2160`).
- **Screen Texture** — optional preview texture for the output canvas
  background. Not required for generation.
- **Visibility** — hidden screens are skipped by all generators.
- **Source Resolution** *(2D only)* — the source image resolution the
  slices are sampling from (e.g. your CineCamera's render size).
- **Source Camera Actor** *(2D only)* — optional reference to a
  `ACameraActor`, `ACineCameraActor`, or `ASceneCapture2D` in your level.
  Drop one in and the 2D input canvas will render the camera's live view
  as its background.

A single layout can contain multiple screens — one per LED wall / surface.

## Slice

A **slice** (`FLedSlice`) is a rectangular region on a screen's pixelmap.
Every slice has two pieces of geometry depending on workflow:

### Output rect (always)

Position + size in pixels **on the LED pixelmap**:
- `X`, `Y` — top-left corner
- `Width`, `Height` — size in pixels

This is edited on the **Output canvas**.

### Input rect (2D only)

Position + size in pixels **on the camera image**:
- `InputX`, `InputY` — top-left corner
- `InputWidth`, `InputHeight` — size in pixels
- `InputRotationDegrees` — rotate the sampled region around its center
- `bInputFlipX` / `bInputFlipY` — mirror the sampled region

This is edited on the **2D Input canvas**.

A slice with a `300 × 300` output rect and `1000 × 1000` input rect
essentially shrinks that 1000×1000 camera area down to 300×300 on the
pixelmap. Rotation and flip let you mount LED strips at any orientation.

### 3D data

Only used in the 3D workflow:
- **Pixel Pitch (mm)** — real-world millimeters per pixel. Drives the
  physical size of the generated static mesh (`pixel_count × pitch` =
  mesh size in mm).
- **World Transform** — preview transform used when building the in-editor
  3D preview. Override once you drop the mesh in your level.
- **Material Slot**, **Generate Individual Mesh** — advanced/rarely-used.

### Draw order

Slices later in the list **overlap / win over** slices earlier in the list
when their output rects intersect — this matters most in the 2D UV bake,
where every output pixel gets exactly one source UV.

Reorder slices in the **Layers** panel by dragging rows around.

## Asset editor tour

Opening a layout opens a docked editor with four panels:

```
┌──────────────┬────────────────────────────┬──────────────┐
│              │                            │              │
│   Layers     │     Input canvas (2D)      │   Details    │
│              │     or 3D preview (3D)     │              │
│  Screens     │                            │  Selected    │
│  Slices      ├────────────────────────────┤  screen /    │
│  Visibility  │                            │  slice       │
│  Drag order  │     Output canvas          │  properties  │
│              │     (LED pixelmap)         │              │
│  Generate    │                            │              │
│              │                            │              │
└──────────────┴────────────────────────────┴──────────────┘
```

- **Layers** — tree of screens and slices. Visibility toggles, rename,
  drag-to-reorder, add / delete / duplicate, **Generate** buttons, output
  folder picker.
- **Input canvas** *(2D layouts)* — edit input rects against the camera's
  live view. Rotation handle appears on selected slices.
- **3D preview** *(3D layouts)* — updating 3D viewport of what the
  generated meshes will look like.
- **Output canvas** — edit output rects on the LED pixelmap space. Overlap
  between slices is highlighted red.
- **Details** — properties of the current selection. Fields that don't
  apply to the current mode are hidden automatically.

## Two complete workflows

- [**3D mesh workflow**](3d-workflow.md) for nDisplay / in-camera VFX
- [**2D pixel-remap workflow**](2d-workflow.md) for post-process
  camera compositing

Both use the same editor — just pick the mode at asset creation.
