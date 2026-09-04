---
title: Architecture
linkTitle: Architecture
weight: 20
description: >-
    How the Eventboat engine works: three-layer pipeline, spool+settle+checkpoint reliability, and the four-gate verification model.
---

## Three-layer model

```
YAML (+overlay) → Config (typed) → Static IR → Runtime Engine
                                          ↓
                              Source → Transform → Sink plugins
```

- **Config layer**: YAML parsing, strict schema validation, variable substitution
- **Static IR**: validated DAG + precompiled CEL programs + Starlark programs + schema
- **Runtime**: spool + settle + checkpoint engine, consuming only the IR

## Reliability model

```
source → [spool: append-only durable queue] → in-memory DAG → sinks
                │                                  │
                └── checkpoint ←── settle tracker ←── terminal states
```

- **Spool**: every message hits SQLite before the DAG sees it (invariant 1)
- **Settle**: each message settles when all branches reach terminal state
- **Checkpoint**: advances only over settled prefix (invariant 2)
- **Crash recovery**: kill -9 → restart → replay from checkpoint, never lose (invariant 3)
- **Dead letters**: exhausted retries → DLQ store with query + replay CLI

## Seven invariant tests

Each has a dedicated test that must pass in CI:

1. Spool before visible
2. Checkpoint advances only after settle
3. Kill -9 replay covers all unsettled
4. Dead-letter write failure blocks settle
5. `required: false` edges don't block siblings
6. Redelivery keeps message ID stable
7. Cursor watermark never exceeds settled

## Four machine gates

| Gate | Command | What it does |
|------|---------|-------------|
| **verify** | `eventboat verify` | Schema, topology, CEL+Starlark compile, lint — static, zero side effects |
| **test** | `eventboat test` | Contract tests against the real engine — fixture in, assertions out |
| **explain** | `eventboat explain --message sample.json` | Deterministic path walkthrough with real CEL evaluation and Starlark dry-run |
| **operate** | `eventboat mcp` | MCP server: 15 tools covering the full agent lifecycle |
