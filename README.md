# `sqi` Community Presets

Welcome to the `sqi` community preset library. This repository contains ready-to-use **presets** for popular digital content creation (DCC) tools and utilities, designed to work with [`sqi`](https://github.com/uberware/sqi) — a modern, distributed task and render farm manager.

> **Status:** Work in progress. The preset system and library are under active development. See [Roadmap](#roadmap) below.

---

## What is a preset?

In `sqi`, a **preset** is a ready-to-use **product definition** — a structured description of how to run a specific tool or workload on the farm. It bridges the gap between raw OpenJD job descriptions (low-level, detailed) and user-facing submission forms (high-level, friendly).

A preset defines:

- **What tool or workload it covers** — e.g., "Arnold Rendering", "Houdini Simulation", "ffmpeg Transcoding"
- **User-facing parameters** — the controls an artist or operator sees in a submission form
- **How parameters map to commands** — the translation from user inputs to actual command-line arguments, environment variables, or other execution details
- **Path handling** — how input files, outputs, and working directories are resolved across different worker environments
- **Environment setup** — software versions, licensing, dependencies, and configuration
- **Compute requirements** — CPU, GPU, memory, and license constraints

Once a preset is installed, artists can submit jobs to the farm without writing code or understanding OpenJD. They fill out a simple form tailored to their tool, and `sqi` handles the rest.

### Example: Arnold Render Preset

```yaml
name: arnold-render
description: "Arnold rendering with support for multiple output drivers and sample controls"
tool: arnold
version: "7.1.0"

parameters:
  scene_file:
    type: file_input
    description: "Scene file to render (.ass, .usd, .mb, etc.)"
    required: true
  output_dir:
    type: path
    description: "Output directory for rendered frames"
    required: true
  frame_range:
    type: frame_range
    description: "Frames to render (e.g., 1-100, 1-100x5 for every 5th frame)"
  samples:
    type: integer
    description: "Camera (AA) samples"
    default: 128
    min: 1
    max: 1024
  ai_denoiser:
    type: boolean
    description: "Enable OptiX denoiser"
    default: false
  num_threads:
    type: integer
    description: "Render threads (-1 = auto-detect)"
    default: -1

command_template: |
  kick -i {scene_file} -o {output_dir}/beauty.exr \
    -as {samples} \
    -threads {num_threads} \
    {if:ai_denoiser:-denoiser optix}

path_resolution:
  scene_file: inline
  output_dir: environment_var
  
license_requirements:
  arnold:
    consumes: 1
```

---

## Using presets

### For artists and operators

1. **Browse available presets** in the `sqi` web UI under Settings → Presets
2. **Install** a preset with one click — it's downloaded from this repository and registered with your `sqi` farm
3. **Customize** a preset's defaults if needed (e.g., adjust default sample counts or add studio-specific environment variables)
4. **Submit jobs** — in the web UI or from within your DCC (Maya, Houdini, Nuke) using the matching DCC submitter
5. **Track jobs** — monitor progress in real time through the web UI

Presets are versioned. You can update to newer versions, pin to a specific version, or maintain local variants.

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
3. **Author your preset** using YAML format with helpful comments explaining parameters and choices (see `docs/preset-spec.md` — coming soon)
4. **Test it** against a running `sqi` farm (local or shared)
5. **Document it** — include a README in your preset's directory explaining non-obvious parameters and setup steps
6. **Submit a pull request** with a clear description of what the preset does and how it was tested

Preset submissions should include:

- `.yaml` or `.json` preset definition file(s)
- `README.md` explaining the preset, parameters, and any prerequisites (software versions, licenses, etc.)
- Example usage (screenshot or command line example)
- Test results (ideally: "tested on Windows 10 with Arnold 7.1.0 and sqi v0.2.1")

### Contribution guidelines

- **Vendor-neutral**: Avoid hardcoding tool paths; use environment variables and `sqi`'s path resolution system
- **Clear parameter names**: Use descriptive names and help text; what seems obvious to you may not be to others
- **Sensible defaults**: Presets should work out of the box for common cases; advanced options can be optional
- **Version-aware**: Call out which versions of a tool the preset supports
- **License-aware**: If the tool requires a commercial license, include a `license_requirements` block declaring what license the preset needs and how many units each task consumes (almost always 1). If the tool is free or open source, omit the block entirely — its absence is the signal that no license pool setup is required. Note: presets declare consumption only — the pool limit (how many total licenses are available) is configured by the operator, not the preset.
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
