# Changelog

All notable changes to the **LED Mapper** plugin are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.3.0] — 2026-04-21

Makes the 3D preview viewport interactive — click-to-select + a transform
gizmo on the selected slice — and adds a single combined mesh per screen
to the 3D generation path.

### Added
- **Combined per-screen mesh on Generate Meshes.** Alongside the
  per-slice `SM_*` assets, each screen now also produces a single
  `<ScreenName>_Combined` static mesh with every visible slice's quad
  baked into one geometry using that slice's `WorldTransform`. Drop the
  combined mesh at origin and the whole wall appears assembled. Shares
  the per-screen material instance.
- **`bGenerateIndividualMesh` honored.** Slices with the flag cleared
  are skipped for individual asset generation but still appear in the
  combined mesh. Uncheck it on every slice to get a single combined
  asset as the sole generated mesh.
- **Click-to-select in the 3D preview viewport.** Left-click any slice's
  mesh → primary selection updates; click empty space → deselect. Layers,
  Details, Output canvas all follow. Implemented via `AStaticMeshActor`
  + standard `HActor` hit proxies.
- **Transform gizmo on the selected slice.** Standard UE translate (W)
  and rotate (E) gizmos operate on the slice's `WorldTransform`. Drag
  is wrapped in one scoped transaction so Ctrl+Z reverts the whole
  manipulation. World / local coord-space toggle via the usual shortcut.
  Scale is ignored by design (pixel pitch drives panel size).
- **Viewport toolbar overlay.** An `SSegmentedControl` in the top-left
  of the 3D preview toggles between translate and rotate modes without
  needing the keyboard. Pressing W / E on the keyboard also updates
  the control.

### Changed
- **3D preview now uses `AStaticMeshActor`s** instead of bare
  `UStaticMeshComponent`s. Actors give us native hit proxies for
  click-to-select with no extra scaffolding.
- **`OnLayoutChanged` resyncs preview-actor transforms** instead of
  forcing a full mesh rebuild. Editing `WorldTransform` in the Details
  panel now moves the preview mesh live; only `OnGeometryChanged`
  (size / pitch / add / remove) still triggers the full rebuild.

### Fixed
- 3D-viewport click-selection was silently broken: `ProcessClick`
  guarded on `Event == IE_Pressed`, but UE routes click events as
  `IE_Released` in most paths. The cursor would turn into a cross on
  hover (hit proxy detected) but clicks never fired selection. Now
  triggers on any left-click regardless of event.
- `FEditorViewportClient::SetWidgetMode` was being called in the
  subclass constructor, before `Viewport` is assigned by
  `SEditorViewport::Construct` — caused a null-deref inside
  `IsFlightCameraActive`. Deferred to the first tick.
- Migrated the viewport toolbar from the now-`final`
  `SEditorViewport::MakeViewportToolbar` to `PopulateViewportOverlays`
  (the UE 5.x successor API) so the plugin builds on 5.7.

### Internal
- New `FLedPreviewViewportClient` overrides `GetWidgetLocation`,
  `GetWidgetCoordSystem`, `InputWidgetDelta`, `TrackingStarted`, and
  `TrackingStopped` to drive gizmo manipulation against
  `ULedMapperState`.
- `SliceActors[]` parallel array on `SLedPreviewViewport` keyed by
  slice index so the client can look up actors for live transform
  updates during drag without a map lookup.
- New `FLedMeshBuilder::BuildCombinedMesh` — one polygon group, all
  slices pre-transformed by their `WorldTransform`, normals rotated via
  `TransformVectorNoScale` so scale on the transform doesn't corrupt them.

---

## [1.2.0] — 2026-04-20

First Fab / Unreal Marketplace release. Substantial additions to editing
ergonomics and output fidelity since the initial internal 1.1.x builds.

### Added
- **Multi-selection.** Shift- / Ctrl-click a slice to toggle it in the set.
  Click-drag on the empty canvas for a rubber-band selection (Shift adds,
  Ctrl toggles, no modifier replaces).
- **Batch operations.** Move-drag, delete, duplicate, and arrow-key nudge
  operate on the whole selection. Edge/corner resize and rotation still
  primary-only by design.
- **Arrow-key nudge.** Arrow = 1 px, Shift+Arrow = 10 px, Ctrl+Arrow = 5 px.
  Wrapped in a single scoped transaction so Ctrl+Z reverts the whole nudge.
- **Snap guides while dragging.** Dashed cyan lines when the dragged rect's
  edges or center line up with another visible slice's edges/center or the
  canvas edges/center. Threshold is constant in on-screen pixels regardless
  of zoom. Hold Alt to disable snapping.
- **Overlap highlighting.** Pairwise output-rect overlaps tinted red on the
  UV canvas to catch accidental double-coverage before baking.
- **Per-slice rotation (2D input workflow).** `InputRotationDegrees` now
  honored by both the 2D input canvas and the UV bake. Dedicated rotation
  handle above top-center of the selected slice; Shift snaps to 15°
  increments. Edge/corner resize is rotation-aware (delta is inverse-rotated
  into slice-local space).
- **Per-slice visibility toggles.** Eye icon in the layers panel. Hidden
  slices and screens are skipped on the canvas, in the 3D preview, and in
  all generators.
- **Per-screen visibility toggles.** Hide an entire screen to skip it during
  generation.
- **Drag-to-reorder layers.** Drag slices between rows to change draw order
  (overlap priority in the UV bake), or onto another screen row to move them.
- **Live CineCamera preview in the 2D input canvas.** Pick a `ACameraActor`,
  `ACineCameraActor`, or `ASceneCapture2D` as a screen's Source Camera and
  the input canvas's background renders what the camera sees, updating every
  frame.
- **2D pixel-remap workflow — Generate Material button.** Alongside the
  existing UV texture bake, a one-click post-process material generator:
  shared `M_LedRemap_Master` + per-screen `MI_LedRemap_<Screen>` with the
  UV remap texture assigned. Graph is a `SceneTexture(PostProcessInput0)`
  sampled via a ComponentMask-RG of the remap texture, with alpha-driven
  Lerp to black, wired to Emissive.
- **Auto-add mode on the 2D input canvas.** Was UV-canvas-only; now works
  on the input canvas too — typed X/Y/W/H values land on the input rect.
- **Output folder picker** on first Generate. Never writes to `/Game/`
  without an explicit Generate click; picks a content folder via a modal
  content-browser dialog and remembers it on the asset.
- **2D / 3D asset-creation dialog** — replaces the Yes/No prompt with a
  two-card modal (3D Layout / 2D Layout) with icons and descriptions.
- **Cancelling asset creation** now aborts cleanly instead of creating a
  default asset.
- **Mode-aware details panel.** Fields that don't apply (2D-only vs 3D-only)
  are hidden via `IDetailCustomization`; clean single view regardless of
  workflow.
- **Layers panel overhaul.** Compact icon toolbar (+ dropdown, duplicate,
  delete, auto-add toggle), aligned columns for name / position / size /
  pitch, per-row type icons, inline rename (F2 or type-to-rename).
- **Undo/redo coverage** for all major operations via scoped transactions
  (add / delete / duplicate / move / nudge / reorder / rotate / property
  edit).

### Changed
- **UV remap texture format** bumped to **FP32** (`PF_A32B32G32R32F`).
  FP16 caused sub-pixel jitter on source textures larger than ~2K because
  adjacent output pixels rounded to the same source UV, which the source's
  bilinear filter then smudged differently. FP32 is pixel-exact at any
  source resolution. Memory cost: ~132 MB per 4K screen.
- **Preview materials** used by the 3D viewport are now transient
  (`UMaterialInstanceDynamic` on a shared in-memory master). Opening a 3D
  layout no longer writes `/Game/LedLayouts/*.uasset` to disk.
- **Auto-add quick-edit buffer** now commits between fields. Prior behaviour
  stacked digits across Space/Tab presses, which produced 1×1 slices regardless
  of what you typed.
- **Asset editor layout** — Input canvas now sits on top of the Output
  canvas by default (was the other way round). Layout version bumped to
  `LedMapper.Layout.v3`, so existing users pick up the new default.

### Fixed
- Runtime module header missing `Modules/ModuleManager.h` include caused a
  clean-build failure (`FDefaultModuleImpl: base class undefined`) on Epic's
  marketplace scanner. Local builds worked via transitive PCH only.
- Post-process material emitted a sampler-type-mismatch warning because the
  default texture on the `UvRemap` parameter was the engine's sRGB
  `DefaultTexture`. Default now points at the first generated UV remap
  texture (linear, matching the sampler's `LinearColor` type).
- `SceneTexture` input field name corrected (`Coordinates`, not `UVs` —
  differs from other texture expressions in UE).
- Clicking empty canvas now grabs keyboard focus, so Auto-Add and quick-edit
  work from a fresh click without needing to first select a slice.
- Focus tug-of-war between the UV and 2D Input canvases — `OnSelectionChanged`
  no longer force-focuses, which was causing Space-to-advance in quick-edit
  to jump focus to whichever canvas registered its handler last.
- Spurious `OnSelectionChanged` broadcasts removed from field-edit paths
  (quick-edit commit, drag commit, proxy commit). Selection events now only
  fire when selection actually changes.
- Tab-cached asset editor layout was recovering an older arrangement when
  the tab order was changed — layout ID bumped to force the new default.

### Removed
- Unused single-function `AddSlice(int32 ScreenIndex)` overload; the default
  `AddSlice(const FLedSlice&)` covers both cases and correctly broadcasts +
  pushes the asset dirty.
- Redundant broadcasts and push-dirty calls in `SLedLayersPanel` that were
  compensating for a bug in the state layer.
- Unused `FLedMeshBuilder::GetEffectiveBasePath` fallback that created a
  `/Game/<asset>/LedLayouts` directory when no output folder was configured;
  generation now always prompts for a folder.

### Internal / build
- Consolidated ~700 lines of duplicated canvas code into `SLedRectCanvasBase`.
  `SLedUvCanvas` (804 → 45 lines) and `SLed2DInputPanel` (803 → 57 lines)
  are now thin subclasses that only supply data accessors.
- Collapsed `ULedSliceProxy` and `ULedScreenProxy` from mirrored-field
  structs to embedded `Data` fields with `ShowOnlyInnerProperties`; adding
  a field to the runtime struct no longer requires updating the proxy.
- Added copyright headers to every C++ source and header file.
- Added `EngineVersion` and per-module `PlatformAllowList` keys to
  `LedMapper.uplugin` per Epic's Technical Requirements Checklist.
- Normalized all source files to UTF-8 no-BOM, CRLF line endings, tab
  indentation. Added `.editorconfig` at repo root to keep it that way.

---

## [1.1.1] — internal build

Initial internal scaffold. Core custom asset editor, 3D mesh generation,
preliminary 2D workflow.
