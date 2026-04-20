# Troubleshooting

Common issues and their fixes, grouped by symptom. If you hit something
not listed here, open a [bug report](../../../issues/new/choose) with your
Unreal version, plugin version, OS, and reproduction steps.

## Install / loading

### The plugin doesn't appear in *Edit → Plugins*

- Confirm the plugin folder landed in `YourProject/Plugins/LedMapper/`
  with `LedMapper.uplugin` at that exact path.
- If you installed from source, the project must have compiled the
  plugin's modules successfully. Look for a dialog at project startup
  offering to build; say yes.
- Restart the editor after enabling.

### *Miscellaneous → Led Layout* missing from the right-click menu

Plugin isn't loaded. Check *Edit → Plugins → Rendering* and enable
**LED Mapper**. Restart.

### Build error: `FDefaultModuleImpl: base class undefined`

Fixed in v1.2.0. Update to the latest release. (Earlier builds only
compiled on machines where transitive PCH includes brought the symbol in
— Epic's build scanner and clean-build environments failed.)

---

## Canvas / editing

### Arrow keys / typing doesn't do anything

Keyboard focus is somewhere else — the Layers panel, Details, or
Windows itself. Click once in empty space on the canvas to grab focus,
then arrows / typing work.

### Ctrl+Z doesn't undo what I did

All major operations (add, delete, duplicate, move, nudge, reorder,
rotate, property edit) are scoped transactions. If undo isn't reverting:
- Confirm Unreal's transaction history isn't suppressed (some projects
  disable it for specific asset types).
- Some property edits go through the Details panel which commits on field
  change — each change is its own transaction.

### Rubber-band selection doesn't select anything

- Click-drag must start on **empty canvas area**, not on a slice.
- If you want to add/toggle rather than replace, hold Shift / Ctrl.

### Shift-click on a selected slice doesn't start a drag

By design — Shift/Ctrl-click is selection-modifier only, never a drag. To
drag a multi-selection, plain-click (no modifier) on any selected slice
and drag.

---

## Generate Meshes (3D)

### Generated meshes look right in the preview but wrong in world

Check **Pixel Pitch (mm)**: this determines the mesh's real-world size.
If your panel is 2.5 mm pitch, set 2.5. A pitch mismatch leaves the mesh
correctly UV-mapped but at the wrong physical scale.

### Mesh has black / broken material

`MI_Screen_<N>` points at `M_MasterLedScreen`. If the master was deleted
or moved, click **Generate Meshes** again — it re-creates the master in
the output folder.

### Generated assets in the wrong folder

Click the small `...` button next to **Generate Meshes** to pick a new
folder. Existing assets in the old folder aren't moved — delete them
manually if you want a clean tree.

---

## Generate Texture / Material (2D)

### Output is black everywhere

The post-process material outputs black where no slice covers the pixel
— that's the `Lerp(A=0, B=SceneTexture, Alpha=UvRemap.A)` design. If
everything is black:
- Your slices probably aren't where you think on the output rect. Check
  on the Output canvas that they actually cover pixels.
- Confirm the material instance is assigned to the right camera /
  SceneCapture in its Post Process Materials array.

### Remapped output looks blurry or has stair-stepping

Fixed in v1.2.0 — UV remap textures are now RGBA32F. If you still see it
on the latest version:
- Check your **material's UvRemap sample** isn't overriding the texture's
  `Nearest` filter with `Bilinear`. In the material editor, select the
  `TextureSample` node → confirm `Mip Value Mode = None` and
  `Sampler Source = From texture asset`.
- Make sure the source camera's render resolution matches the UV
  texture's resolution. Sampling a `1920 × 1080` camera into a
  `3840 × 2160` layout is intentional scaling, but non-integer ratios
  produce slight softness — switch to the SceneCapture-at-LED-resolution
  setup in the [2D workflow docs](2d-workflow.md).

### Thin lines in the output jitter over time

**TAA jitter** — Temporal Anti-Aliasing on the source camera adds
sub-pixel offsets each frame. Our remap faithfully carries that jitter,
which shows up as shimmer on 1-pixel-wide detail.

Fixes:
- On the source camera's Post Process: *Rendering Features →
  Anti-Aliasing Method* → switch from `TAA` / `TSR` to `FXAA` or `MSAA`.
- Or lower / disable `Screen Percentage` so the source renders at native.

### Material compiles forever on first use

One-time shader compile for the master material. Subsequent uses are
instant. Delete the `M_LedRemap_Master` asset if you need to force a
recompile.

### Sampler-type mismatch warning

Fixed in v1.2.0 — the master's default texture now points at the first
generated UV remap (linear), matching the sampler's `LinearColor` type.
If you regenerate the master and still see the warning, confirm you're
on v1.2.0+ and the default texture on the `UvRemap` parameter is one of
the baked `T_UvRemap_*` textures, not engine `DefaultTexture`.

---

## Live camera preview (2D)

### The 2D Input canvas is black

- Make sure a **Source Camera Actor** is set on the screen (Details →
  Led|Screen|2D section).
- The referenced actor must exist in the current level. `TSoftObjectPtr`
  breaks silently if the actor is in a different map.
- `Source Preview Opacity` might be 0 — check it's non-zero.

### A capture actor appears in my level outliner

The plugin spawns a hidden `ASceneCapture2D` to drive the preview. It
should be filtered out of the Scene Outliner via `bHideFromSceneOutliner`
but may briefly appear before the filter applies. It's transient and
destroys itself when you close the asset editor.

If it's sticking around after closing: report as a bug with steps.

### Camera framing in the preview doesn't exactly match the viewport

The preview mirrors the camera's `FieldOfView` — which for CineCamera is
computed from focal length and filmback. For extreme filmbacks
(anamorphic, film stocks with unusual aspect ratios), there can be a
slight framing mismatch. Workaround: use a plain `ACameraActor` or
`ASceneCapture2D` and set FOV directly.

---

## Performance

### The asset editor stutters with many slices

- The canvas paint is O(n²) in overlap highlighting. With 100+ visible
  output rects, this can become noticeable. Temporarily hide slices you
  aren't actively editing.
- Live camera preview runs a full scene capture every frame. Expect a
  small perf hit vs. regular editor work.

### Generate Texture is slow on large screens

RGBA32F at 4K is 132 MB of pixel data computed on the main thread. On
stopwatch this is a second or two per screen — normal. If generation
stalls for longer, your output folder is probably on a network drive —
move it to local storage.

---

## When to file a bug

Please open a [bug report](../../../issues/new/choose) if:

- You see an editor crash or hang
- Generated assets are incorrect after following the docs
- A shortcut or feature described here doesn't work
- A compile error appears that isn't in this list

Include:
- Unreal Engine version
- LED Mapper plugin version (`.uplugin` VersionName, or *Edit → Plugins*)
- Operating system
- Minimal reproduction steps
- Screenshots / a short video if the bug is visual
