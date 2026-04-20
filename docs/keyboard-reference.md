# Keyboard & mouse reference

All the shortcuts, grouped by what you're trying to do. Everything lives
on the Output canvas and 2D Input canvas — the Details panel and Layers
tree use standard Unreal conventions.

## Viewport

| Shortcut | Action |
|---|---|
| `Ctrl+0` / double-click | Fit canvas to view |
| `Ctrl+1` | Zoom to 100% |
| Middle-mouse drag | Pan canvas |
| `Alt` + LMB drag (empty area) | Pan canvas |
| Mouse wheel | Zoom in/out |
| `Ctrl` + mouse wheel | Fine zoom |

## Selection

| Shortcut | Action |
|---|---|
| LMB on slice | Select only this slice |
| `Shift` / `Ctrl` + LMB on slice | Toggle in selection set |
| LMB drag on empty canvas | Rubber-band select |
| `Shift` + rubber-band | Add to current selection |
| `Ctrl` + rubber-band | Toggle in selection |
| LMB on empty canvas (no drag) | Clear selection |

## Move, resize, rotate

| Shortcut | Action |
|---|---|
| Drag slice body | Move (whole set if multi-selected) |
| Drag corner / edge handle | Resize (primary slice only) |
| Drag rotation dot (2D Input) | Rotate primary slice around its center |
| Arrow keys | Nudge selection by 1 px |
| `Shift` + arrows | Nudge by 10 px |
| `Ctrl` + arrows | Nudge by 5 px |
| `Shift` while dragging | Axis-lock move (dominant axis) |
| `Alt` while dragging | Disable snapping (grid + guides) |
| `Shift` while rotating | Snap rotation to 15° increments |

## Clipboard / edit

| Shortcut | Action |
|---|---|
| `Ctrl+C` | Copy primary slice |
| `Ctrl+V` | Paste slice near selection |
| `Ctrl+D` | Duplicate selection |
| `Alt` + LMB on slice | Duplicate and drag |
| `Delete` / `Backspace` | Delete selection |
| `F2` | Rename selected layer-panel row |
| `Ctrl+Z` / `Ctrl+Y` | Undo / redo (standard UE) |

## Quick-edit HUD

Type a digit on the canvas while a slice is selected and a numeric HUD
appears in the bottom-left.

| Shortcut | Action |
|---|---|
| `0`–`9`, `-`, `.` | Type into the current field |
| `Space` / `Tab` | Advance to next field (`X → Y → W → H → Pitch`) |
| `Shift` + `Tab` | Previous field |
| `Backspace` | Remove last digit from buffer |
| `Enter` | Commit |
| `Escape` | Cancel |

## Auto-add mode

Toggle **Auto Add** in the Layers panel toolbar. With nothing selected
and focus on the canvas:

| Flow | Result |
|---|---|
| `100 Space 200 Space 500 Space 400 Enter` | Creates a slice at `(100, 200)` size `500 × 400` |
| Auto-add on Output canvas | Values go to the output rect |
| Auto-add on 2D Input canvas | Values go to the input rect |

The `Pitch` field is in the cycle (after `H`) — useful for 3D layouts
where you want to set physical pitch on creation.

## Layers panel

| Shortcut | Action |
|---|---|
| LMB on row | Select that screen / slice |
| Drag row | Reorder within screen, or move between screens |
| Click eye icon | Toggle visibility |
| `F2` | Rename |
| Type letters when a row is selected | Start renaming with that character |
| `Del` / `Backspace` | Delete selection |
| `Ctrl+D` | Duplicate |

## Modifier summary

| Modifier | While… | Effect |
|---|---|---|
| `Shift` | Clicking slice | Toggle in selection |
| `Shift` | Dragging slice | Axis-locked move |
| `Shift` | Typing nudge (arrows) | 10 px step |
| `Shift` | Rotating | Snap to 15° |
| `Ctrl` | Clicking slice | Toggle in selection |
| `Ctrl` | Typing nudge | 5 px step |
| `Ctrl` | Mouse wheel | Fine zoom |
| `Ctrl` | Tab / Space in quick-edit | (Tab) Previous field |
| `Alt` | Clicking slice | Duplicate + drag |
| `Alt` | Dragging | Disable snap |
| `Alt` | LMB on empty | Pan canvas |
