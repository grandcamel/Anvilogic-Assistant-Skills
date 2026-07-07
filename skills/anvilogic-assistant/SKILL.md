---
name: "anvilogic-assistant"
description: "Anvilogic SOC automation hub for detection engineering via the anvilogic-as CLI. Use when the user mentions Anvilogic, detection rules, threat scenarios, EOI (events of interest), threat hunting, alert triage, feeds, forge, blueprints, or SOC metrics - e.g. 'list my Anvilogic detections', 'explore the Anvilogic registry', 'describe the detections API'. This skill is registry-driven: it discovers operations with 'anvilogic-as registry list' / 'registry describe' and executes them with the generic 'anvilogic-as api' driver. Wave 2 stub: mock mode and registry exploration only; domain skills arrive in Wave 4."
version: "0.1.0"
author: "anvilogic-assistant-skills"
license: "MIT"
allowed-tools: ["Bash", "Read", "Glob", "Grep"]
---

# Anvilogic Assistant (Router Hub - STUB)

This hub routes Anvilogic SOC detection-engineering requests to specialized
skills. It contains no code: the `anvilogic-as` CLI does all the work, and the
CLI's **registry is the only source of truth** for what operations exist and
how to call them. This skill never hardcodes API endpoints or paths.

> **WAVE STATUS (Wave 2 - skeleton)**
>
> Endpoint paths are **not yet captured** in the public registry: every
> Wave-1 entry records `method: UNKNOWN` / `path: UNKNOWN`. Today this
> plugin supports **registry exploration plus a small mock-mode demo**:
>
> - `anvilogic-as registry list` shows which operations are captured so far.
> - Registry ids are **not executable yet** - `anvilogic-as api <METHOD> <id>`
>   exits 1 (uncaptured path), even in mock mode.
> - `ANVILOGIC_MOCK_MODE=1` serves demo fixtures for two raw paths only:
>   `GET /v1/ping` and `GET /v1/items`.
>
> Live API coverage lands in Wave 3 (gated capture) and Wave 4 (domain skills).

## Registry-Driven Approach

Never guess an endpoint. Discover, then drive:

1. **Discover operations** (read-only):

   ```bash
   anvilogic-as registry list
   ```

   Lists every captured operation as `<domain>.<operation>` registry ids
   (e.g. `detections.list`), with capture status and risk marks.

2. **Inspect one operation** (read-only):

   ```bash
   anvilogic-as registry describe <id>
   ```

   Shows the operation's parameters, risk level, and capture provenance.

3. **Execute via the generic driver**:

   ```bash
   anvilogic-as api <METHOD> <path-or-registry-id> [--query k=v ...] [--data <json>|@file]
   ```

   e.g. `anvilogic-as api GET detections.list`. `anvilogic-as api` is the
   single generic entry point. It takes exactly two positional arguments:
   the HTTP `<METHOD>` (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS) and
   the target - a registry id (resolved to its captured path) or an
   explicit path starting with `/`. Query parameters go in repeatable
   `--query key=value` flags; a JSON request body goes in `--data` (inline
   JSON or `@file`). There is no `--param` flag.

   Take the method from `registry describe <id>` - never guess it. If the
   registry does not know an operation (or its method/path is still
   `UNKNOWN`), it cannot be called - do not construct raw URLs or paths.
   The only raw paths you may use are the mock-mode demo fixtures
   (`GET /v1/ping`, `GET /v1/items`).

## Risk Levels

Every registry operation carries a risk mark. Confirm with the user before
any `⚠️` operation; require explicit confirmation plus a dry-run/preview for
`⚠️⚠️` and `⚠️⚠️⚠️`.

| Mark | Meaning |
|:----:|---------|
| `-` | Read-only, safe |
| `⚠️` | Write - creates or modifies a single object |
| `⚠️⚠️` | Bulk - touches many objects at once |
| `⚠️⚠️⚠️` | Destructive - deletes or deploys irreversibly |

## Quick Reference

| I want to work with... | Domain skill | Status |
|------------------------|--------------|--------|
| Detection rules (list, tune, deploy) | anvilogic-detections | Wave 4 - coming |
| Threat scenarios (sequences of detections) | anvilogic-scenarios | Wave 4 - coming |
| Events of Interest (EOI records) | anvilogic-eoi | Wave 4 - coming |
| Threat hunting queries and campaigns | anvilogic-hunting | Wave 4 - coming |
| Alerts and triage | anvilogic-alerts | Wave 4 - coming |
| Intel and content feeds | anvilogic-feeds | Wave 4 - coming |
| Forge content development | anvilogic-forge | Wave 4 - coming |
| Detection blueprints | anvilogic-blueprints | Wave 4 - coming |
| SOC metrics and reporting | anvilogic-metrics | Wave 4 - coming |

Until Wave 4, answer every domain request the same way: explore the registry
(`anvilogic-as registry list`, `registry describe <id>`). Registry ids are
not yet executable - every Wave-1 entry has an uncaptured (`UNKNOWN`)
method and path, so `anvilogic-as api <METHOD> <id>` exits 1 until Wave 3
captures the real ones, even in mock mode. To demonstrate the driver
itself, use the mock-mode demo fixtures, e.g.
`ANVILOGIC_MOCK_MODE=1 anvilogic-as api GET /v1/ping`.

## Setup

If `anvilogic-as` is missing or credentials are not configured, run the
`/anvilogic-assistant-setup` command.
