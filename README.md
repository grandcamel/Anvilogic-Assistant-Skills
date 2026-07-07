# Anvilogic Assistant Skills

Claude Code plugin for Anvilogic SOC detection-engineering automation.

This is a **doc-only plugin**: the skills contain no code. All work is done by
the [`anvilogic-as`](https://github.com/grandcamel/anvilogic-as) Go CLI, and
the CLI's **registry is the only source of truth** for which API operations
exist. Skills never hardcode endpoints or paths - they discover operations
with `anvilogic-as registry list` / `registry describe <id>` and execute them
through the generic `anvilogic-as api <METHOD> <path-or-registry-id>` driver
(e.g. `anvilogic-as api GET detections.list`).

## Repository relationship

| Repo | Role |
|------|------|
| [`anvilogic-as`](https://github.com/grandcamel/anvilogic-as) | Go CLI (binary: `anvilogic-as`) plus the `api-surface/` registry - the captured evidence of the Anvilogic API surface (endpoints, auth, conventions, coverage matrix) |
| this repo | Claude Code plugin: skills + commands that teach Claude how to drive the CLI |

The `api-surface` registry in the CLI repo tracks what has been captured,
implemented, tested, and documented per `domain.operation`. Skills in this
repo only reference operations that exist in that registry.

## Wave roadmap

| Wave | Scope | Status |
|:----:|-------|--------|
| 1 | Public-source API surface capture (`api-surface/` registry) | Done 2026-07-06 |
| 2 | Plugin scaffold - structure, router-hub stub, setup command (this repo) | In progress |
| 3 | Gated capture - live/authenticated endpoint verification | Planned |
| 4 | Domain skill coverage - detections, scenarios, eoi, hunting, alerts, feeds, forge, blueprints, metrics | Planned |
| 5 | First release - tagged versions, brew tap, marketplace listing | Planned |

> **Current limitation:** endpoint paths are not yet captured, so the plugin
> supports **mock mode and registry exploration only**
> (`ANVILOGIC_MOCK_MODE=1`).

## Install

1. Install the CLI:

   ```bash
   go install github.com/grandcamel/anvilogic-as/cmd/anvilogic-as@latest
   ```

   (`brew tap grandcamel/tap` is coming at the first release.)

2. Add this plugin to Claude Code (marketplace listing arrives in Wave 5;
   until then, add the repo as a local plugin/marketplace).

3. Run the setup command inside Claude Code:

   ```text
   /anvilogic-assistant-setup
   ```

## Quick start

```bash
# Verify the binary
anvilogic-as version --check-min 0.1.0

# Explore the registry (works today in mock mode)
ANVILOGIC_MOCK_MODE=1 anvilogic-as registry list
ANVILOGIC_MOCK_MODE=1 anvilogic-as registry describe detections.list

# Credentials (for later waves)
export ANVILOGIC_API_KEY="..."   # or: anvilogic-as auth set
anvilogic-as auth validate
```

Then ask Claude naturally: "list my Anvilogic detections" or "explore the
Anvilogic registry".

## License

MIT - see [LICENSE](LICENSE).
