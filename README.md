# Perde

## Repository Layout

| Directory | Contents |
|---|---|
| `hw/` | Hardware design sources (KiCad projects) |
| `fw/` | Firmware sources |
| `mech/` | Mechanical design sources |

## Hardware

The PCB design lives in [`hw/main-board`](hw/main-board), a KiCad 10 project consisting of the schematic (`main-board.kicad_sch`), the board layout (`main-board.kicad_pcb`), and the build configuration for [KiBot](https://github.com/INTI-CMNB/KiBot) (`main-board.kibot.yaml`).

### Continuous Integration / Continuous Delivery

Every change to the hardware design is automatically validated and built by GitHub Actions. The pipeline is implemented with KiBot, executed inside the official `kicad10_auto` / `kicad10_auto_full` container images (KiCad 10), and performs visual revision comparison via [KiDiff](https://github.com/INTI-CMNB/KiDiff), exposed through KiBot's `diff` output. Workflow definitions reside under [`.github/workflows`](.github/workflows).

#### `kicad-ci.yml` — Continuous Integration

Triggered on every push and pull request affecting `hw/main-board/**`. The workflow consists of the following jobs:

| Job | Description |
|---|---|
| **ERC / DRC + docs** | Executes the Electrical Rules Check and Design Rules Check preflights, and generates PDF exports of the schematic and PCB layout. |
| **Gerbers / drill / BOM / STEP** | Produces Gerber and drill files (with a combined ZIP archive for fabrication), pick-and-place data, the Bill of Materials in HTML and CSV formats, and a STEP 3D model. |
| **3D renders** | Renders top and bottom views of the assembled board. |
| **Visual diff (KiDiff)** | Generates a red/green visual comparison of the PCB and schematic against a prior revision — the pull request base branch when run on a pull request, or the preceding commit when run on a direct push. |
| **Publish latest build** | Consolidates the outputs of the jobs above and publishes them to the `gh-pages` branch under `/latest/`, together with a generated index page, providing a persistent link to the most recent build of `main`. |

As the hardware design is presently at an early stage, the pipeline is configured not to fail on design-rule violations: KiBot is invoked with `-D` (`--dont-stop`), and each step is marked `continue-on-error`. Reports and partial outputs are uploaded regardless of outcome, so any issues remain visible without interrupting the build. This policy is intended to be tightened once the design matures beyond a placeholder state.

#### `kicad-release.yml` — Release Packaging

Triggered by pushing a tag matching the pattern `vX.Y.Z`. Builds the complete set of documentation, fabrication, assembly, and 3D outputs, and attaches the resulting package to a corresponding GitHub Release. A manual invocation (`workflow_dispatch`) performs the build step only, as no tag is available to publish a release against.

#### Build Outputs

Generated files are retained as downloadable artifacts on each workflow run. In addition, the latest successful build of `main` is published at:

<https://ordu1453.github.io/Perde/latest/>
