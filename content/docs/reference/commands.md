---
title: CLI Reference
linkTitle: Commands
weight: 10
description: >-
    All 11 Eventboat CLI commands with flags and examples.
---

## Overview

```console
eventboat [--json] verify --config <pipeline.yaml> [--strict]
eventboat [--json] test <testfile-or-dir> [...]
eventboat run --config <pipeline.yaml> [--data-dir DIR] [--ephemeral]
eventboat run --config-dir <dir> [--runtime runtime.yaml]
eventboat [--json] trigger --config <job.yaml> [--parameters '{"from":"..."}']
eventboat [--json] jobs list --config <job.yaml> [--limit N]
eventboat [--json] jobs show <run-id> --config <job.yaml>
eventboat [--json] explain --config <pipeline.yaml> [--message f.json] [--topology]
eventboat [--json] replay --config <pipeline.yaml> (--dlq | --spool --from N | --job <run-id>) [--dry-run]
eventboat repl [--message sample.json] [--cel 'expr' | --script f.star]
eventboat lsp
eventboat [--json] plugin catalog
eventboat [--json] plugin schema <name>
eventboat mcp (--stdio | --http) [--config-dir <dir>] [--data-dir DIR]
```

## verify {#verify}

Statically validate a pipeline. Checks schema, topology invariants
(no cycles, no orphans, source has no in-edges, sink has no out-edges,
at least one source→sink path), CEL predicate compilation, Starlark
script compilation, job configuration, and semantic lint.

| Flag | Default | Description |
|------|---------|-------------|
| `--config` | (required) | Pipeline YAML file |
| `--strict` | false | Upgrade warnings to errors |

## test {#test}

Run contract test suites against the real in-process engine. Test files
declare injection points, expected captures, and DLQ assertions.

```yaml
suite: my-test
pipeline: ../pipeline.yaml
cases:
  - name: vip-order-routed-correctly
    inject: { at: ingest, messages: [fixtures/order.json] }
    expect:
      capture: { at: out }
      messages:
        - payload.total: 12000    # subset match
```

## run {#run}

Execute a pipeline. Job pipelines (with `run.mode: job`) run under the
jobs manager with scheduling and catchup. `--config-dir` starts a
multi-pipeline daemon with the admin surface.

| Flag | Default | Description |
|------|---------|-------------|
| `--config` | — | Single pipeline file |
| `--config-dir` | — | Directory of pipeline files (daemon mode) |
| `--runtime` | `./eventboat.yaml` | Runtime config (telemetry endpoints) |
| `--data-dir` | `data` | SQLite storage directory |
| `--ephemeral` | false | In-memory store (nothing persists) |

## trigger {#trigger}

Manually fire a job pipeline once, optionally with parameters (backfill).

```bash
eventboat trigger --config sync.yaml --parameters '{"from":"2026-08-01","to":"2026-09-01"}'
```

## explain {#explain}

Deterministic walkthrough of a pipeline. With `--message`, performs real
CEL evaluation and Starlark dry-run on the sample. With `--topology`,
renders the DAG (mermaid + ASCII).

## replay {#replay}

Re-inject dead letters (`--dlq`), a spool window (`--spool --from N`),
or one job run's dead letters (`--job <run-id>`) into a live pipeline.

## repl {#repl}

Evaluate CEL predicates and Starlark scripts against one sample message
without running a pipeline.

```bash
eventboat repl --message sample.json --cel 'payload.score > 100'
eventboat repl --message sample.json --script transform.star
```

## mcp {#mcp}

Start the MCP server for AI agents.

| Flag | Description |
|------|-------------|
| `--stdio` | Speak MCP over stdin/stdout (for agent hosts) |
| `--http` | Serve MCP over HTTP with Admin REST + SSE + UI |
| `--config-dir` | Deploy pipelines at startup |
