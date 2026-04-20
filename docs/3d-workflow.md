# 3D mesh workflow

Use this workflow when you're driving **real LED panels** (via nDisplay,
ICVFX, or a scene capture), and you want the virtual camera to see what's
on the LEDs. You design where the pixelmap slices sit on physical panels
in the world, and the plugin generates pixel-pitch-accurate static meshes
with correct UVs.

If you're trying to remap a camera's output onto an LED processor pixelmap
instead, you want the [2D pixel-remap workflow](2d-workflow.md).

---

## Step-by-step

### 1. Create a 3D layout

Right-click in the Content Browser → *Miscellaneous → Led Layout*. A modal
appears — click **3D Layout**. Name the asset. It opens in the LED Mapper
editor with one empty screen.

### 2. Set up the screen

In the **Details** panel:

- **Screen Width / Height** — the total pixel dimensions of your LED
  pixelmap. If your processor takes a `3840 × 2160` input, those are the
  numbers.
- **Screen Texture** — drop a reference texture (pixelmap image, test
  pattern, screenshot of a render target) so you can see what you're
  mapping on the Output canvas. Preview-only; not baked into the mesh.
- **Screen Texture Opacity** — how much of the background texture to show
  on the canvas.

### 3. Add slices

Each **slice** is one LED panel (or one logical region that corresponds
to a mesh piece).

- Click **+ → Add Slice** in the Layers panel, or
- Toggle **Auto Add** and type `X Y W H Pitch` on the canvas.

For each slice, set on the Output canvas:

- **Position** and **size** of its rectangle on the pixelmap.
- In Details: **Pixel Pitch (mm)** — the panel's real-world pitch. This
  directly determines the generated mesh's physical size
  (`pixel_count × pitch` in mm).

Example: a slice `1920 × 1080` pixels at `2.5 mm` pitch generates a mesh
**480 × 270 cm** (just under 5m × 2.7m) with UVs sampling that region of
the pixelmap.

### 4. Tune with the layers panel

- Drag slices to reorder — mostly cosmetic in 3D (no overlap semantics
  like in the UV bake), but keeps the tree readable.
- Toggle the eye icon to hide slices you don't want to generate yet.
- F2 or type-to-rename.

### 5. Generate meshes

Click **Generate Meshes** at the bottom of the Layers panel.

- First time: a content-folder picker appears. Pick somewhere under
  `/Game/` — e.g. `/Game/LedWalls/StageA`. The choice is saved on the
  asset.
- Subsequent clicks use the saved path. Click the small `...` button next
  to *Generate Meshes* any time to re-pick.

What you get, per screen:

| File | What it is |
|---|---|
| `M_MasterLedScreen` | Unlit two-sided master material with a `ScreenTexture` parameter. Created once per output folder. |
| `MI_Screen_<N>` | One material instance per screen, with the screen's `Screen Texture` plugged into the parameter. |
| `SM_<SliceName>_<WxH>_P<pitch>` | One static mesh per slice. Vertical quad facing +X with UVs mapping its region of the LED pixelmap. |

Generated assets are **marked dirty but not saved**. Hit Ctrl+S or
Save All to commit them to disk.

### 6. Place in your level

Drop each `SM_*` mesh into your world, position and rotate it to match
where the real LED panel sits. The **World Transform** field in Details
lets you preset positions on the asset, which the generated mesh respects
when first dropped.

Typical setups:

- **nDisplay** — include the meshes in your nDisplay cluster's
  `Scene Component` hierarchy; configure each node to render to its
  corresponding output.
- **Scene Capture** — point an `ASceneCapture2D` at the meshes; its
  render target becomes your pixelmap output.
- **ICVFX / Film capture** — the meshes render as part of the virtual set;
  the camera sees them naturally in the final image.

---

## Tips

### Pixel pitch matters for scale, not rendering

Pitch doesn't affect the texture sampling — just the physical size of the
mesh. Use it to reflect the real panel pitch (2.5 mm, 5 mm, etc.) and the
mesh lines up correctly at 1:1 world scale.

### Regenerate when the layout changes

Generators are **idempotent** — running Generate Meshes again updates the
existing meshes in place (geometry and UVs) without creating duplicates.
Material instances stay linked.

If you rename a slice or change the output folder, existing assets with
the old names aren't renamed automatically. Either delete and regenerate,
or rename manually and update the slice name on the asset.

### The in-editor 3D preview

The 3D preview tab shows the meshes as they'd render, using a **transient**
preview material — no files are written to disk when you just open the
asset editor. (Earlier builds accidentally wrote `/Game/LedLayouts/*.uasset`
on every preview refresh — this is fixed.)

### Hidden slices and screens

The eye icon in the Layers panel hides a slice/screen from:
- the Output canvas
- the 3D preview
- **all generators** — hidden items are skipped when you click Generate

Use this to preview subsets of your wall, or to exclude a draft slice
from a release build.

---

## Next

- [2D pixel-remap workflow](2d-workflow.md) — the other workflow, if your
  setup is camera → post-process → LED processor instead.
- [Keyboard & mouse reference](keyboard-reference.md) — all the shortcuts.
- [Troubleshooting](troubleshooting.md) — if something's off.
