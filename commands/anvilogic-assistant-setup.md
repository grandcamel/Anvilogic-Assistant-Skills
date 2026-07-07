---
name: anvilogic-assistant-setup
description: Set up Anvilogic Assistant Skills with the anvilogic-as CLI and credentials
---

# Anvilogic Assistant Setup

You are helping the user set up Anvilogic Assistant Skills. Guide them through
the process conversationally.

## Step 1: Check the CLI binary

Verify the `anvilogic-as` binary is installed and new enough:

```bash
command -v anvilogic-as
anvilogic-as version --check-min 0.1.0
```

If both succeed, skip to Step 3.

## Step 2: Install the CLI

Install from source with Go (requires Go 1.26+, per the CLI's `go.mod`;
older Go installs with the default `GOTOOLCHAIN=auto` will fetch it
automatically):

```bash
go install github.com/grandcamel/anvilogic-as/cmd/anvilogic-as@latest
```

Make sure `$(go env GOPATH)/bin` is on `PATH`, then re-run the Step 1 checks.

> Homebrew installation (`brew tap grandcamel/tap && brew install
> anvilogic-as`) is **coming at the first tagged release** - use `go install`
> until then.

## Step 3: Get an API key

Tell the user they need an Anvilogic platform API key:

1. Log in to the Anvilogic platform (an **admin role** is required to manage
   API keys).
2. Go to **Settings > Generate API Key**.
3. Copy the key immediately.

## Step 4: Configure credentials

Either export the environment variable (add to `~/.zshrc` / `~/.bashrc`):

```bash
export ANVILOGIC_API_KEY="their-key-here"
```

Or store it with the CLI's credential helper:

```bash
anvilogic-as auth set
```

## Step 5: Validate

```bash
anvilogic-as auth validate
```

If the user has no key yet (or live endpoints are not available in the
current wave), fall back to mock mode. Mock coverage is currently limited
to registry exploration plus two demo fixture paths (`anvilogic-as api GET
/v1/ping` and `GET /v1/items`) - registry ids are not yet executable while
their paths are uncaptured:

```bash
ANVILOGIC_MOCK_MODE=1 anvilogic-as registry list
```

## Step 6: Confirm success

If validation succeeds, tell the user:

"Anvilogic Assistant Skills are set up. Try:

- `anvilogic-as registry list` - see every captured operation
- `anvilogic-as registry describe detections.list` - inspect one operation
- Or just ask naturally: 'list my Anvilogic detections'"

## Troubleshooting

The `anvilogic-as` CLI uses distinct exit codes. Key the diagnosis to the
code:

| Exit code | Meaning | Fix |
|:---------:|---------|-----|
| 1 | Validation / usage error (bad arguments or flags, unknown registry id passed to `api`, or a registry id whose path is not yet captured in the current wave) | Re-run with `--help`; run `anvilogic-as registry list` to see valid ids; for uncaptured paths, wait for Wave 3 or use the mock-mode fixture paths |
| 2 | Authentication failed (no API key configured, or key rejected with HTTP 401) | Set `ANVILOGIC_API_KEY` or run `anvilogic-as auth set`; if the key is rejected, regenerate it in the platform (Settings > Generate API Key, admin role) |
| 3 | Permission denied (HTTP 403) | The key's role lacks access to this operation - use a key with sufficient (admin) privileges |
| 4 | Not found (HTTP 404, or unknown id in `registry describe`) | Check the resource id for typos; run `anvilogic-as registry list` |
| 5 | Rate limit exhausted (HTTP 429) | Wait and retry; reduce request volume or pagination fan-out |
| 6 | Conflict (HTTP 409) | The resource changed underneath you - re-read it and retry the operation |
| 7 | Server or network error (HTTP 5xx, connection failure, timeout) | Check connectivity to the platform (HTTPS/443, proxy settings); retry; if persistent, capture the output and check platform status |

If `command -v anvilogic-as` finds nothing after `go install`, ensure
`$(go env GOPATH)/bin` is on `PATH`. If `version --check-min` fails, upgrade
with the same `go install ...@latest` command.
