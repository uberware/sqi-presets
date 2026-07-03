# `sqi` Community Presets

Welcome to the `sqi` community preset library. This repository contains ready-to-use **presets** for popular digital content creation (DCC) tools and utilities, designed to work with [`sqi`](https://github.com/uberware/sqi) — a modern, distributed task and render farm manager.

> **Status:** Work in progress. The preset system and library are under active development. See [Roadmap](#roadmap) below.

---

## What is a preset?

In `sqi`, a **preset** is a **product definition file** — catalog metadata paired with an inline OpenJD job template. The two-part structure looks like this:

```
┌──────────────────────────────────────────────────┐
│  Catalog envelope (preset metadata)              │
│  name · title · description · category · version │
│  ┌────────────────────────────────────────────┐  │
│  │  OpenJD template (verbatim)                │  │
│  │  parameterDefinitions · steps · …          │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

The preset layer adds **only** the catalog envelope. Everything else — parameter definitions and their UI control hints, command-line arguments, path handling, machine requirements, setup/teardown steps — lives **inside** the OpenJD template, exactly as it would in any OpenJD job description. There is no separate preset-level schema for those concerns.

When a preset is installed, `sqi` stores it as a product. Artists can then submit jobs from the **Admin hub → Preset Library** without writing YAML: `sqi` renders a form from the template's `userInterface` hints, collects values, and posts the job.

### Example: Arnold Render Preset

```yaml
name: studio/arnold-render
title: Arnold Render
description: Render a scene with Arnold over a frame range.
category: Rendering
version: 1.0.0
template:
  specificationVersion: jobtemplate-2023-09
  name: Arnold Render
  parameterDefinitions:
    - name: SceneFile
      type: PATH
      userInterface:
        control: CHOOSE_INPUT_FILE
        label: Scene File
    - name: Frames
      type: STRING
      default: 1-100
      userInterface:
        control: LINE_EDIT
        label: Frame Range
  steps:
    - name: Render
      parameterSpace:
        taskParameterDefinitions:
          - name: Frame
            type: INT
            range: "{{Param.Frames}}"
      script:
        actions:
          onRun:
            command: kick
            args: ["-i", "{{Param.SceneFile}}", "-frame", "{{Task.Param.Frame}}"]
```

The `name` slug (e.g. `studio/arnold-render`) is the stable identity used by `sqi` to reference the product. Namespaced names (one `/`) are recommended for community presets to avoid collisions.

---

## Index & hosting

`sqi` discovers presets via a static **`index.json`** at the root of this repository, served over GitHub Pages at:

```
https://uberware.github.io/sqi-presets/index.json
```

Each entry in the index points to a definition file and carries the content hash that `sqi` verifies on install:

```json
{
  "presets": [
    {
      "name": "studio/arnold-render",
      "title": "Arnold Render",
      "description": "Render a scene with Arnold over a frame range.",
      "category": "Rendering",
      "version": "1.0.0",
      "definition": "presets/rendering/arnold-render.yaml",
      "sha256": "<hex digest of the definition file's content>"
    }
  ]
}
```

| Field | Description |
|---|---|
| `name` | Stable slug, matches the `name` field in the definition file. |
| `title` | Human-readable name shown in the Preset Library browser. |
| `description` | Short summary shown in the Preset Library browser. |
| `category` | Group label for filtering. |
| `version` | Semver string from the definition file. |
| `definition` | Repo-relative path to the `.yaml` or `.json` definition file. |
| `sha256` | SHA-256 hex digest of the definition file's content. `sqi` verifies this on install and rejects the file if it does not match. |

> **GitHub Pages must be enabled** for the default URL above to resolve. A maintainer enables it once via **Settings → Pages → deploy from `main`, root**. This is not part of contributing a preset.

---

## Using presets

### For artists and operators

1. **Browse available presets** in the `sqi` web UI under **Admin hub → Preset Library**
2. **Install** a preset with one click — `sqi` downloads the definition, verifies its SHA-256 hash, validates the template, and stores it as an installed product
3. **Submit jobs** using the installed preset — select it from the product picker, fill in the form, and submit
4. **Track jobs** — monitor progress in real time through the web UI

Installed presets are **read-only**: the underlying template is managed by the preset library. If you want to customize a preset (adjust defaults, add parameters, change the command), use **Duplicate to custom** — this creates a fully editable local copy that is no longer tied to the library. To remove an installed preset, use **Uninstall** from the product detail page.

### For developers and TDs

- **Read preset specifications** under `docs/preset-spec.md` (coming soon)
- **Browse preset source** in this repository — each tool has its own directory with YAML definitions
- **Create or modify presets** using the `sqi` web UI preset editor, or write it directly in a text editor (YAML is preferred for sharing because comments make presets more understandable and easier to debug)
- **Contribute** back to the community (see [Contributing](#contributing))
- **Integrate** presets into custom pipelines via the Python client API

---

## Available presets

The following presets are planned for `sqi` v1.0 (June 2026):

### Rendering
- **Arnold** — Rendering with full parameter control, denoising, GPU support
- **Blender Cycles** — CPU and GPU rendering, sample counts, output drivers  
- **Cinema 4D** — Standard and Physical rendering, GI settings, stereoscopic
- **Houdini Karma / Mantra** — Houdini's native and RenderMan-compatible renderers
- **Nuke** — Script-based compositing and rendering, batch mode execution
- **Maya Software / Arnold in Maya** — Batch rendering with full scene control

### Simulation & Procedural
- **Houdini SIM** — General-purpose simulation (fluids, particles, cloth, destruction)
- **Maya nCloth** — Cloth and deformable body simulation

### Media Processing
- **ffmpeg** — Video transcoding, color space conversion, format conversion, audio processing
- **ImageMagick** — Batch image manipulation and conversion
- **OCIO / Color Correction** — Color space transforms and grading

### Post-Production & Editing
- **After Effects (beta)** — Compositing and effects rendering via AERENDER
- **Shotcut** — Video editing and export

### Utilities
- **Python Script** — Execute Python scripts with parameter passing and environment control
- **Docker Container** — Run any containerized workload (image-agnostic, supports registry auth)
- **Generic Command** — A catch-all preset for custom scripts and tools

**Status of each preset:**
- ✓ = Implemented and tested
- ◐ = In progress
- ◯ = Planned but not yet started

| Preset | Status |
|--------|--------|
| Python Script | ◯ |
| Docker Container | ◯ |
| Arnold | ◯ |
| Blender | ◯ |
| Cinema 4D | ◯ |
| ffmpeg | ◯ |
| Houdini Karma | ◯ |
| Houdini Mantra | ◯ |
| Houdini SIM | ◯ |
| Maya | ◯ |
| Nuke | ◯ |
| After Effects | ◯ |
| ImageMagick | ◯ |
| OCIO | ◯ |

---

## Contributing

Community contributions are encouraged, especially for:

- **New preset definitions** for tools you use and know well
- **Improvements to existing presets** — better parameter descriptions, new options, edge case handling
- **Bug reports** — if a preset doesn't work as expected
- **Documentation** — tips, tutorials, setup guides

### How to contribute a preset

1. **Fork** this repository
2. **Create a branch** for your preset — e.g., `preset/mycompany-custom-maya`
3. **Author your preset** as a product definition YAML file (catalog metadata + inline OpenJD template — see the example above)
4. **Place the file** under an appropriate directory, e.g. `presets/rendering/maya-render.yaml`
5. **Compute the SHA-256** of your definition file: `sha256sum presets/rendering/maya-render.yaml`
6. **Add an entry to `index.json`** with the `name`, `title`, `description`, `category`, `version`, `definition` path, and `sha256` hash
7. **Test it** against a running `sqi` farm (local or shared) — install from the Preset Library and submit a job
8. **Document it** — include a README in your preset's directory explaining prerequisites (software versions, path setup, etc.)
9. **Submit a pull request** with a clear description of what the preset does and how it was tested

Preset submissions should include:

- The `.yaml` or `.json` product definition file
- An updated `index.json` entry with the correct `sha256`
- `README.md` in the preset's directory explaining prerequisites and non-obvious parameters
- Test results (ideally: "tested on Linux with Arnold 7.1.0 and sqi v0.2.1")

### Contribution guidelines

- **Vendor-neutral**: Avoid hardcoding tool paths; use `PATH`-resolved commands or parameter-driven interpreter paths
- **Clear parameter names and UI hints**: Use descriptive names, `userInterface.label`, and `userInterface.control` hints so the submission form renders clearly
- **Sensible defaults**: Presets should work out of the box for common cases; advanced options can be optional
- **Version-aware**: Call out which versions of a tool the preset supports
- **License-aware**: If the tool requires a commercial license, declare a usage-pool requirement in `hostRequirements.amounts` (the pool itself is configured by the operator, not the preset)
- **Cross-platform**: When possible, support Linux, macOS, and Windows (or document where they differ)

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines and the contributor code of conduct.

---

## Roadmap

### v0.2 — Presets system launch (June 2026)
- [ ] Preset specification finalized and documented
- [ ] Web UI preset editor and installer
- [ ] First 5 presets released (Arnold, Blender, ffmpeg, Houdini Karma, Nuke)
- [ ] Python API support for preset validation and programmatic definition
- [ ] Community submission portal opens

### v0.3 — Preset ecosystem expansion
- [ ] Additional DCCs (Maya, After Effects, Cinema 4D, additional simulation tools)
- [ ] First 10 community-contributed presets merged
- [ ] Preset templates and generation helpers for common patterns

### v1.0 — Production release (August 2026)
- [ ] All planned v1.0 presets complete and battle-tested
- [ ] Comprehensive preset documentation and tutorials
- [ ] Community voting / discoverability system
- [ ] Preset versioning and upgrade path finalized

---

## Documentation

- **[README.md](https://github.com/uberware/sqi/blob/main/README.md)** — Overview of `sqi` and its features (in the main repo)
- **[Preset Specification](docs/preset-spec.md)** — Complete YAML schema reference (coming soon)
- **[FAQ](docs/faq.md)** — Common questions about presets and `sqi` (coming soon)

---

## Getting help

- **Questions about a preset?** Check its README or open an issue in this repository
- **Want to request a preset?** Open an issue with the tag `preset-request` and describe your tool and use case
- **Having trouble installing or using a preset in `sqi`?** See the main `sqi` documentation or open an issue in the [main `sqi` repository](https://github.com/uberware/sqi)
- **Other questions?** Use the [sqi Discussions](https://github.com/uberware/sqi/discussions) to ask questions from the `sqi` community

---

## License

Preset definitions in this repository are licensed under the **Creative Commons Attribution 4.0 International License** (CC-BY-4.0). This means:

- You can use, modify, and distribute presets freely
- You must credit the original author
- You can use presets in any context — commercial, non-commercial, internal, or external

The underlying `sqi` software is licensed under **AGPL v3.0** with a commercial license option. See the [main sqi repository](https://github.com/uberware/sqi) for details.

---

## About

The `sqi` project and preset library are maintained by [Uberware](https://www.uberware.net), in collaboration with the broader studio and VFX community.

Have questions or want to contribute? Reach out at [robin@uberware.net](mailto:robin@uberware.net) or open an issue here.
