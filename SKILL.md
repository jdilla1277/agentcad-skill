---
name: agentcad
description: 'CAD tool for AI agents. Use when the user asks you to design, model,
  or build a 3D object. agentcad executes build123d Python scripts and produces STEP
  files, PNG renders, mesh exports (STL/GLB/OBJ), and geometric metrics.

  '
compatibility: Requires Python 3.10-3.12 and agentcad installed (pip install agentcad).
allowed-tools: Bash(agentcad:*)
version: 0.4.1
metadata:
  openclaw:
    requires:
      bins:
      - agentcad
      anyBins:
      - python3.12
      - python3.11
      - python3.10
---

# agentcad — CAD tool for AI agents

You have access to `agentcad`, a CLI that turns build123d Python scripts into 3D
geometry. All output is JSON. Every command returns `"command"` and `"status"` keys.

## First-time setup

```bash
agentcad init --name <project_name>
agentcad --help   # Read the built-in how-to guide and command reference
```

## Core workflow

1. **Write a script.** No imports needed — build123d primitives,
   `show_object`, and agentcad edit helpers are pre-injected by default.
   `show_object(result)` is required.

2. **Dry-run first** to check metrics without consuming a version:
   ```bash
   agentcad run script.py --output test --dry-run
   ```
   Check `volume`, `dimensions`, `is_valid` in the response.

3. **Run for real.** Visual feedback is on by default:
   ```bash
   agentcad run script.py --output label
   ```
   A normal successful iteration can produce (paths in the JSON response):
   - `preview.png` — 4-view composite (front, right, top, iso). **Read this**
     to confirm the part looks right before iterating. One image, all 4 angles.
   - `diff.side_by_side` — side-by-side PNG vs the most recent successful prior
     version, when one exists and automatic diff is enabled. **Read this** when
     iterating to see what your change did.
   - `diff.overlay` — tinted (green prev, red this) overlay for subtle shifts.
     Read only if side-by-side didn't resolve the question.
   - `viewer.html` — interactive 3D review viewer for the user unless viewer
     artifacts are disabled (humans only;
     you can't render HTML). It opens automatically after a successful run.
     From v2, A=previous and B=current are already loaded with synchronized
     A/B, side-by-side, overlay, diff-image, and Parts-tab change review.

   Pass `--no-preview` only for tight parametric sweeps where latency matters.
   Pass `--no-view` only when browser launch would disrupt an unattended or
   high-volume run.

   For a core-only iteration, pass
   `--no-preview --no-diff --no-view`. This writes `output.step`, the saved
   script, `meta.json` (including metrics), and explicitly requested exports
   without generating previews, automatic comparisons, viewer assets, or
   opening a browser. You can still run an explicit
   `agentcad diff OLD NEW` later.

   When a comparison is slow or incomplete, read `comparison_phases` in the
   JSON response. `source_loading`, `comparison_rendering`,
   `projection_comparison`, `exact_3d_comparison`,
   `approximate_3d_comparison`, `difference_artifact_export`, and
   `viewer_generation` each report a status
   and, when attempted, `duration_ms`. The largest duration identifies the
   expensive stage; a failed exact phase does not erase a successful projection.
   Exact 3D work has a 30-second default worker budget. Set
   `AGENTCAD_DIFF_TIMEOUT_S=N` to override it (`0` disables the dedicated limit
   for diagnostics). A timeout leaves the core version and projection usable,
   then runs a bounded voxel fallback. Approximate results report
   `method=approximate_voxel_volume`, `resolution_mm`, and a non-strict
   `error_estimate`; its `absolute_volume` values are heuristic errors, not
   measurements. `exact_attempt` retains the exact failure. Use
   `AGENTCAD_APPROX_DIFF_TIMEOUT_S` and `AGENTCAD_APPROX_RESOLUTION_MM` to tune
   the fallback. If exact volumes are still needed, run
   `agentcad diff OLD NEW` with a larger budget; do not rerun the original CAD
   command and create a duplicate version.

   Daemon-routed commands may run beyond 30 seconds. Progress heartbeats appear
   on stderr while stdout stays reserved for the final JSON response, and the
   submitted command is never automatically retried. If a silent or lost daemon
   returns `outcome: "unknown"` and `retry_safe: false`, inspect `agentcad
   context`, existing outputs, and `agentcad daemon status` before retrying; the
   original command may already have completed.

4. **Review with the user.** The generated viewer opens automatically. On v2+
   start with its previous/current comparison, then use A/B, Overlay, and Parts
   without selecting files manually. Use `agentcad view old.step new.step` only
   for an explicit non-adjacent comparison.

5. **Inspect if invalid.** If `is_valid: false` or geometry looks wrong:
   ```bash
   agentcad inspect v1_label/output.step
   ```

6. **Measure feature sizes.** For dimensions beyond top-level metrics:
   ```bash
   agentcad measure v1_label/output.step
   ```
   Use this for hole diameters, cylindrical boss diameters, edge lengths,
   face areas, and full per-feature measurements with `--features`.

7. **Check explicit feature requirements.** If the prompt names measurable
   holes, bores, or cylindrical bosses, write them into `spec.json` before
   final handoff:
   ```json
   {"features":[{"name":"bolt_holes","type":"cylinder","diameter_mm":6,"count":4}]}
   ```
   Then run:
   ```bash
   agentcad check-spec v1_label/output.step spec.json
   ```
   Revise the CAD if `passed` is false. `status: success` only means the
   comparison ran; `passed` is the actual spec-check result. If you include
   `axis`, copy it from `agentcad measure`'s `cylindrical_features[].axis`.

8. **Iterate.** Fix the script, run with a new `--output` label. Use
   `agentcad diff 1 2` to compare versions.

## Script writing rules

- `show_object(result)` is required — at least one call.
- These are pre-injected by default (no import needed):
  build123d primitives like `Box`, `Cylinder`, `Sphere`, `Plane`, plus
  `show_object`, `load_step`, `pick_face`, `pick_edge`, `fillet_edges`,
  `chamfer_edges`, `shell_faces`, `cut_pocket`, `boss`, `split_by_plane`,
  `replace_face`, `annular_boss`, and `raise_annulus`.
- For imported STEP/BREP edits, `load_step(path)` returns a build123d `Part`:
  ```python
  base = load_step("v1_vendor/output.step")
  solids = base.solids()
  faces = base.faces()
  edges = base.edges()
  bounds = base.bounding_box()
  ```
  Use `agentcad measure` and `agentcad inspect` for read-only discovery; use
  the loaded `Part` in a script when changing geometry. See
  `agentcad docs editing` for the complete edit workflow.
- For imported STEP annular edits, use the non-fuse workflow:
  ```python
  raw = load_step_shape("v1_vendor/output.step")
  result = raise_annulus(raw, center=(0, 0), inner_diameter=40,
                         outer_diameter=80, height=7, z=5)
  show_object(Compound(result))
  ```
- For OCP internals (`gp_Pnt`, `BRepPrimAPI`, etc.), import manually.
- CadQuery compatibility remains available for existing projects. See
  `agentcad docs runtimes` for that separate workflow.

## Key commands

| Command | Purpose |
|---------|---------|
| `agentcad init --name NAME` | Initialize project |
| `agentcad run SCRIPT --output LABEL` | Execute script, produce STEP + metrics |
| `agentcad run ... --dry-run` | Metrics only, no version consumed |
| `agentcad run ... --no-preview` | Suppress preview (on by default) |
| `agentcad run ... --no-diff` | Suppress automatic prior-version comparison |
| `agentcad run ... --no-view` | Suppress automatic browser review |
| `agentcad run ... --render iso,front` | PNG views |
| `agentcad run ... --export stl,glb` | Mesh export |
| `agentcad run ... --params k=v,k=v` | Override script parameters |
| `agentcad render STEP --view SPEC` | Post-hoc renders with camera control |
| `agentcad export STEP --format stl,glb` | Post-hoc mesh export |
| `agentcad measure STEP` | Dimensional report (overall metrics + feature sizes) |
| `agentcad check-spec STEP spec.json` | Pass/fail checklist against intended cylindrical features |
| `agentcad inspect STEP` | Topology report (validity, free edges) |
| `agentcad parts list REF` | List parts captured for a version |
| `agentcad parts show REF ID` | Show one versioned part by stable id |
| `agentcad diff REF1 REF2` | Compare versions |
| `agentcad context` | Project state and interrupted-version recovery candidates |
| `agentcad recover VERSION_DIR` | Validate and reconcile interrupted history without deleting files |
| `agentcad docs [SECTION]` | Runtime-aware built-in documentation |
| `agentcad instructions install` | Record a short project note so future agents read `agentcad --help` |
| `agentcad view FILE [FILE_B]` | Open one model or an explicit synchronized A/B comparison |

## Debugging playbook

1. **Check metrics first** — `volume` and `dimensions` catch most issues.
2. **Read `preview.png`** — the 4-view composite. Fastest way to spot obvious problems.
3. **Read `diff.side_by_side`** if iterating — confirms your change did what you intended.
4. **Negative volume?** Wire winding is backwards (CW instead of CCW).
5. **Need a hole diameter or edge length?** Run `agentcad measure output.step`.
6. **Need to verify explicit hole/bore counts?** Write `spec.json`, then run
   `agentcad check-spec output.step spec.json`.
7. **is_valid: false?** Run `agentcad inspect` — check `free_edge_count` and shell status.
8. **Hollow shape?** `free_edge_count > 0` means open shell.
9. **Complex profiles (gears, splines)?** Use subtractive construction — cut from
   a blank cylinder/box instead of building up. See `agentcad docs patterns`.

## Patterns

- **Build at origin, then position:** Create geometry at origin, use `translate()`
  and `rotate()` to place it.
- **Compound vs fuse:** `Compound([...])` keeps assembly parts separate; use
  build123d's `+` operator to boolean-fuse solids.
- **Parametric scripts:** Top-level variable assignments become overridable via
  `--params`. Use this for iteration.
- **Named parts:** `show_object(shape, id="wheel_left", name="Left wheel",
  options={"color": "red"})` for stable part handles, per-part metrics, and
  colored GLB export.
