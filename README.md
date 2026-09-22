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
| **Gerbers / drill / BOM / STEP** | Produces Gerber and drill files for JLCPCB and PCBWay using KiBot's built-in per-manufacturer templates, each in its own directory with its own ZIP archive (JLCPCB's additionally includes an LCSC-formatted BOM and pick-and-place file); a manufacturer-neutral Bill of Materials (HTML and CSV), a pick-and-place file, and a STEP 3D model are generated separately. |
| **3D renders** | Renders top and bottom views of the assembled board. |
| **Visual diff (KiDiff)** | Generates a red/green visual comparison of the PCB and schematic against a prior revision — the pull request base branch when run on a pull request, or the preceding commit when run on a direct push. |
| **Publish latest build** | Consolidates the outputs of the jobs above, regenerates an interactive KiCanvas schematic/PCB viewer and a KiBot `navigate_results` index over the merged tree, and publishes the result to the `gh-pages` branch under `/latest/`, providing a persistent, browsable link to the most recent build of `main`. |

As the hardware design is presently at an early stage, the pipeline defaults to not failing on design-rule violations: KiBot is invoked with `-D` (`--dont-stop`), which always applies, and each step's `continue-on-error` is controlled by the **`KICAD_CI_STRICT`** repository variable (*Settings → Secrets and variables → Actions → Variables*):

- Unset, or any value other than `true` (default) — lenient: a real ERC/DRC/output failure is still reported in the uploaded artifacts, but does not fail the job, so an early-stage or placeholder design doesn't block CI.
- `true` — strict: a real failure fails the job for real, and any job depending on it (fabrication, renders, publish-latest) is skipped, exactly as with any other genuine CI failure.

Setting `KICAD_CI_STRICT=true` once the design is no longer a placeholder is recommended so a broken board or schematic is reflected honestly in the pipeline's status.

#### `kicad-release.yml` — Release Packaging

Triggered by pushing a tag matching the pattern `vX.Y.Z`. Builds the complete set of documentation, fabrication, assembly, and 3D outputs, and attaches the resulting package to a corresponding GitHub Release. A manual invocation (`workflow_dispatch`) performs the build step only, as no tag is available to publish a release against.

#### Build Outputs

Generated files are retained as downloadable artifacts on each workflow run. In addition, the latest successful build of `main` is published at:

<https://ordu1453.github.io/Perde/latest/>
