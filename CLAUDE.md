# CLAUDE.md — Anvilogic-Assistant-Skills

Doc-only Claude Code plugin for Anvilogic SOC detection-engineering
automation. **Skills contain no code** — they teach the agent to drive the
`anvilogic-as` CLI (sibling repo). Project plan, wave runbooks, and migration
runbook live in the CLI repo under `docs/` — read the relevant runbook before
wave work.

## Contract (do not violate)

- **The registry is the only source of truth for endpoints.** Skills may only
  reference operations that exist in the CLI's `api-surface/endpoints.yaml`
  (`anvilogic-as registry list` / `registry describe <id>`). Never invent
  endpoints, paths, or parameters in skill docs.
- **Skill anatomy:** frontmatter `name` + third-person `description` with
  trigger phrases (<1024 chars) + `allowed-tools: ["Bash","Read","Glob","Grep"]`;
  body <500 lines; deep detail goes in the skill's `docs/` (progressive
  disclosure); no deprecated `when_to_use` field.
- **Risk marks** (from the registry `risk` enum): `-` read, ⚠️ write,
  ⚠️⚠️ bulk, ⚠️⚠️⚠️ destructive. Destructive/bulk operations must carry the
  mark and a confirm-before-run note.
- **Exit-code table** in `commands/anvilogic-assistant-setup.md` (1 validation
  / 2 auth / 3 permission / 4 not-found / 5 rate-limit / 6 conflict /
  7 server) mirrors the CLI — change them together or not at all.
- Every documented invocation must be copy-paste runnable — in mock mode
  (`ANVILOGIC_MOCK_MODE=1`) until live endpoints are validated.
- No gated/customer-only content, no employer/customer names, no tenant data
  anywhere in this repo.

## Validate locally (mirrors CI)

```sh
jq empty .claude-plugin/plugin.json .claude-plugin/marketplace.json
npx --yes markdownlint-cli2 "**/*.md" "!node_modules"
# frontmatter: every skills/*/SKILL.md needs name: and description:
```

## Conventions

Conventional commits. Configure `git config user.name/user.email` to match
the account of the remote you push to. Router hub is
`skills/anvilogic-assistant/SKILL.md`; domain skills (Wave 4) follow
`skills/anvilogic-<domain>/` naming — domains: detections, scenarios, eoi,
hunting, alerts, feeds, forge, blueprints, metrics.
