# LED Mapper

A visual slice-to-LED editor for Unreal Engine. Design your LED wall or volume
mapping directly in-editor and generate the assets you need for either of two
complete workflows — no external tools, no round-trips.

Think of it like a Resolume-style pixel-map editor that lives inside Unreal's
asset system and speaks nDisplay + post-process materials.

> **This repository is the public home for LED Mapper's documentation, release
> notes, and issue tracker.** The plugin itself is distributed through the
> Fab marketplace (the repo has no code).

---

## Get the plugin

- **Install via Fab** — *link will be posted here once the listing is live*
- **Release notes** — [CHANGELOG.md](CHANGELOG.md)
- **Supported engine**: Unreal Engine 5.7+, Windows

## Documentation

Start here if you're new:

- [**Concepts**](docs/concepts.md) — layout assets, screens, slices, and the two workflows
- [**3D mesh workflow**](docs/3d-workflow.md) — nDisplay / in-camera VFX
- [**2D pixel-remap workflow**](docs/2d-workflow.md) — post-process camera compositing
- [**Keyboard & mouse reference**](docs/keyboard-reference.md) — all the shortcuts
- [**Troubleshooting**](docs/troubleshooting.md) — precision, TAA, camera mismatch, focus, and other gotchas

## The two workflows at a glance

| | 3D mesh workflow | 2D pixel-remap workflow |
|---|---|---|
| **Use when** | Driving real LED panels via nDisplay, or ICVFX captures where the virtual camera should see the LED content | Remapping a CineCamera's output onto an LED processor pixelmap (post-process compositing) |
| **You design** | Where pixelmap slices sit on physical panels in the world | What part of the camera image feeds each chunk of the LED pixelmap |
| **You generate** | Static meshes sized to real panel dimensions + unlit material instance per screen | FP32 UV-remap texture + pre-wired post-process material per screen |
| **You then** | Drop meshes in your world, drive them with nDisplay or a scene capture | Plug the material onto a camera / SceneCapture as a blendable |

Both workflows share the same editor, the same asset, and the same canvas.
You just pick which mode at asset-creation time.

## Report a problem or ask a question

- **Bug** → [Open an issue](../../issues/new/choose)
- **Question, idea, or showcase** → [Discussions](../../discussions)

Before filing, please search open issues and discussions to avoid duplicates.
Good bug reports include your Unreal version, plugin version, OS, and
reproduction steps — the bug-report template walks you through it.

## Pull requests

The plugin's source lives in a private repository. Pull requests against this
public docs repo are accepted for documentation fixes / typos (see
[CONTRIBUTING.md](CONTRIBUTING.md)); PRs for the plugin itself aren't.

---

LED Mapper is developed by **Pieter Van Itterbeeck** ([pietervi.com](https://pietervi.com)).
Licensing of the plugin itself is covered by the Fab Standard Content License.
