# 2D pixel-remap workflow

Use this workflow when your LED output is driven by a **camera's post-processed
image**, remapped into the LED processor's pixelmap layout. Typical scenario:
your show is running inside Unreal, a CineCamera is framing the shot, and you
need the camera's rendered output to land in the right rectangles on a
`3840 × 2160` (or whatever) pixelmap that feeds the LED processor.

If you're driving physical LED panels via meshes and nDisplay instead, you
want the [3D mesh workflow](3d-workflow.md).

---

## What you'll end up with

- **A UV remap texture** — one per screen. An RGBA32F image the size of your
  LED pixelmap. For each output pixel it stores the source-image UV to
  sample. Baked once per layout change.
- **A post-process material** — a `SceneTexture(PostProcessInput0)` sampled
  through the UV remap texture. Alpha-masked to black where no slice
  covers. Emits to the final image.
- **A per-screen material instance** with the UV remap texture pre-assigned.

You apply the material instance to your camera (or a downstream scene
capture), and the remap happens automatically per frame.

---

## Step-by-step

### 1. Create a 2D layout

Right-click in the Content Browser → *Miscellaneous → Led Layout*. Click
**2D Layout**. Name the asset and open it.

### 2. Set up the screen

In the **Details** panel:

- **Screen Width / Height** — your LED pixelmap resolution (e.g.
  `3840 × 2160`).
- **Source Resolution** — your camera's render resolution (e.g.
  `1920 × 1080`). This is what the slices' input rects are measured in.
- **Source Camera Actor** — drop a `ACineCameraActor` (or
  `ACameraActor` / `ASceneCapture2D`) from your level. The **2D Input**
  canvas will start rendering what the camera sees as its background.

### 3. Add slices and set rectangles

Each **slice** has:

- An **output rect** — where it lives on the LED pixelmap.
- An **input rect** — which part of the camera image feeds it.

Add a slice (Layers → **+ → Add Slice**, or toggle **Auto Add** and type).
Then:

- On the **Output canvas**: position and size the output rect — this is
  your LED layout.
- On the **2D Input canvas**: position and size the input rect on top of
  the camera's live view — this is what gets sampled.

Other per-slice controls (Details panel, 2D section):

- **Input Rotation (deg)** — rotate the sampled region around its center.
  Rotation handle appears above top-center of the selected slice on the
  2D Input canvas. Shift snaps to 15°.
- **Input Flip X / Y** — mirror the sampled region horizontally / vertically.

### 4. Generate

Two buttons at the bottom of the Layers panel in 2D mode:

#### Generate Texture

Bakes **only** the UV remap texture — useful if you've already set up your
own post-process material and just need the texture updated.

Produces, per screen:

- `T_UvRemap_<ScreenName>.uasset`

This is an **RGBA32F** texture the size of `Screen Width × Screen Height`.
Each texel stores:

- `R, G` — source UV to sample (`0..1`)
- `B` — reserved (0)
- `A` — `1` where a slice covers, `0` elsewhere

Compression is `TC_HDR_F32`, filter is `Nearest`, no mips, not streamed.

#### Generate Material

Bakes the texture **and** creates a ready-to-use post-process material.
Produces (in addition to the textures):

- `M_LedRemap_Master` — a shared post-process master material:

  ```
  TextureSampleParameter2D("UvRemap")
       │ RGB                          │ A
       ▼                              ▼
   ComponentMask (RG)            [routes to Lerp.Alpha below]
       │
       ▼ UVs
   SceneTexture(PostProcessInput0)
       │ Color
       ▼                             A = 0
   Lerp ─── A:0, B:SceneTex, Alpha:UvRemap.A
       │
       ▼
   Emissive Color
  ```

- `MI_LedRemap_<ScreenName>` — one material instance per screen, parented
  to the master, with the `UvRemap` texture parameter pre-set to that
  screen's baked texture.

---

## Hooking the material into your scene

The generated material is a **post-process material**. It needs to run
somewhere `PostProcessInput0` exists — i.e. on a camera or scene capture.

### Simplest: on the CineCamera itself

1. Select the CineCamera in your level.
2. *Details → Post Process → Rendering Features → Post Process Materials*.
3. Add `MI_LedRemap_<ScreenName>` as a blendable.

The camera's final rendered image is now remapped per your layout.

> **Caveat**: the camera's render resolution needs to match your LED
> pixelmap resolution for the remap to be pixel-accurate. The UV
> texture's texels correspond 1:1 to output pixels. If the camera is
> `1920 × 1080` and your screen is `3840 × 2160`, use the Scene Capture
> approach instead.

### Recommended for LED output: via a SceneCapture2D

1. Place an `ASceneCapture2D` somewhere near the CineCamera.
2. Follow the camera's transform every frame (set up a tick or parenting).
3. Create a `Texture Render Target 2D` at the LED pixelmap resolution
   (`Screen Width × Screen Height`).
4. Assign that render target as the scene capture's **Texture Target**.
5. On the scene capture: *Post Process Materials* → add
   `MI_LedRemap_<ScreenName>`.
6. Use the render target as your LED output — feed into a Media Output,
   nDisplay node, or a dedicated output viewport.

This gives you a dedicated pixelmap-resolution output without affecting
what the CineCamera sees in-viewport.

### Swapping the source camera

Since the material runs wherever you attach it as a blendable, you can
change which camera drives the remap freely. The layout (UV texture) is
fixed; the source is swappable at runtime.

---

## Live camera preview

The **2D Input canvas** renders the selected `Source Camera Actor`'s live
view as its background. Implementation: a hidden `ASceneCapture2D` is
spawned in the editor world to mirror the camera's transform + FOV each
frame, rendering to a transient render target. When you close the asset
editor, the helper actor is destroyed. You may briefly see a capture actor
appear at world origin before it's positioned — this is cosmetic.

Opacity of the preview is controlled by **Source Preview Opacity** in
Details — dial down to see slice rects clearly against busy content.

---

## Tips

### Overlap priority

Slices later in the Layers panel **win** over earlier ones when output
rects intersect, since each output pixel gets exactly one source UV.
Overlap regions are tinted red on the Output canvas so you catch
accidental double-coverage. Reorder via drag in the Layers panel.

### Flips compose with rotation

Flips are applied **before** rotation (in the UV bake). If you want a
slice that's rotated *and* mirrored, set both — they combine correctly.

### Regenerate after layout changes

Generators are **idempotent** — running Generate Texture or Generate
Material again updates existing assets in place. Textures are re-baked;
the master material is only created once per folder (subsequent runs
re-use it and update per-screen MIC parameters).

If you rename a screen, the existing MIC keeps its old name. Either
delete it and regenerate, or rename the asset manually.

### Hidden slices

Eye icon hides a slice from:
- the Output canvas
- the 2D Input canvas
- the UV remap bake

Hidden screens are skipped entirely by generators.

### Precision

UV remap textures are RGBA32F — full float precision. You'll get
pixel-accurate sampling even on 4K+ source textures. Memory cost is
~132 MB per 4K screen; consider that when planning for many screens.

---

## Next

- [3D mesh workflow](3d-workflow.md) — the other workflow.
- [Troubleshooting](troubleshooting.md) — if something's off in the output.
- [Keyboard & mouse reference](keyboard-reference.md) — all the shortcuts.
