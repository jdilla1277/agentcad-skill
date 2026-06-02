# agentcad-skill

The **agent skill manifest** for [agentcad](https://agentcad.dev) — a CLI-based CAD tool for AI agents.

This repo is the public entry point for agent skill marketplaces ([ClawHub](https://clawhub.ai/), [skills.sh](https://skills.sh)). It contains only the `SKILL.md` manifest — the agentcad CLI itself lives at [jdilla1277/agentcad](https://github.com/jdilla1277/agentcad) and ships via PyPI.

## Install

### skills.sh (Vercel)

```bash
npx skills add jdilla1277/agentcad-skill
```

### ClawHub (OpenClaw)

```bash
clawhub install jdilla1277/agentcad
```

### Manually (Claude Code)

Install the CLI and let it drop the skill into your project:

```bash
pip install agentcad
agentcad skill install
```

## What agentcad does

Agents write bad 3D geometry on the first try. agentcad gives them a tight feedback loop — run, render, inspect, fix — so they converge on printable geometry without you babysitting.

- **Execute** — run CadQuery Python scripts, produce versioned STEP files + geometric metrics
- **Render** — PNG views from any angle for visual verification
- **Export** — STL, GLB, OBJ for 3D printing and web viewers
- **Validate** — pre-execution checks catch errors in <100ms
- **Inspect** — topology report for debugging geometry issues
- **Diff** — compare versions to track design iteration

See [agentcad.dev](https://agentcad.dev) for the full pitch and live gallery.

## Requirements

- Python 3.10–3.12 (CadQuery/OpenCascade does not support 3.13+)
- `agentcad` CLI on `$PATH` (`pip install agentcad`)

## License

The skill manifest in this repo is licensed under Apache-2.0.

The agentcad CLI itself is open source under [Apache-2.0](https://github.com/jdilla1277/agentcad/blob/main/LICENSE).

## Source

- CLI source: [github.com/jdilla1277/agentcad](https://github.com/jdilla1277/agentcad), distributed via [PyPI](https://pypi.org/project/agentcad/).
- Skill manifest: this repo. Generated from the public CLI repo on each release.
- Feedback / issues: run `agentcad feedback "your message"` from inside a project, or file an issue here.
