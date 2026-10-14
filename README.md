# agentcad-skill

The **agent skill manifest** for [agentcad](https://agentcad.dev) — a CLI-based CAD tool for AI agents.

This repo is the public entry point for agent skill marketplaces ([ClawHub](https://clawhub.ai/jdilla1277/skills/agentcad), [skills.sh](https://skills.sh/jdilla1277/agentcad-skill/agentcad)). Its `SKILL.md` is generated from the [agentcad CLI source](https://github.com/jdilla1277/agentcad) and follows the required **Inspect → Specify → Build → Validate → Deliver** workflow for both new models and edits. The CLI itself ships via PyPI.

## Install

### skills.sh (Vercel)

```bash
npx skills add jdilla1277/agentcad-skill
```

### ClawHub (OpenClaw)

```bash
clawhub install agentcad
```

### Manually (Claude Code)

Install the CLI and let it drop the skill into your project:

```bash
pip install agentcad
agentcad skill install
```

## What agentcad does

AgentCAD gives agents a CAD feedback loop: inspect inputs, specify what must change, build, validate the result, and deliver the checked artifact. It uses build123d by default and supports CadQuery as an optional compatibility runtime.

- **Execute** — run build123d Python scripts, produce versioned STEP files and geometric metrics
- **Render** — PNG views from any angle for visual verification
- **Export** — STL, GLB, OBJ for 3D printing and web viewers
- **Validate** — check geometry health, dimensions, structural requirements, and edits
- **Inspect** — topology report for debugging geometry issues
- **Diff** — compare versions to track design iteration

See [agentcad.dev](https://agentcad.dev) for the full pitch and live gallery.

## Requirements

- Python 3.10–3.12 (OpenCascade bindings do not support 3.13+)
- `agentcad` CLI on `$PATH` (`pip install agentcad`)

## License

The skill manifest in this repo is licensed under Apache-2.0.

The agentcad CLI itself is open source under [Apache-2.0](https://github.com/jdilla1277/agentcad/blob/main/LICENSE).

## Source

- CLI source: [github.com/jdilla1277/agentcad](https://github.com/jdilla1277/agentcad), distributed via [PyPI](https://pypi.org/project/agentcad/).
- Skill manifest: this repo. Generated from the public CLI repo when its guide changes and on each release.
- Feedback / issues: run `agentcad feedback "your message"` from inside a project, or file an issue here.
