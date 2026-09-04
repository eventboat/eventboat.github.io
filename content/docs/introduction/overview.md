---
title: What is Eventboat?
linkTitle: Overview
weight: 10
description: >-
    A Go single-binary DAG event router designed for AI agents to operate end-to-end.
---

Eventboat routes events through directed acyclic graphs (DAGs) of transforms —
from **sources** (Kafka, HTTP, cron, SQL, files) to **sinks** (Kafka, HTTP, files),
with at-least-once delivery, dead-lettering, and replay.

It is **agent-native**: every capability is accessible through MCP (Model Context
Protocol), CLI, and a language server — so AI agents can write, verify, deploy,
and operate pipelines autonomously.

## What makes it different

| Dimension | Approach |
|-----------|----------|
| **Predicates** | CEL (Kubernetes standard) — zero custom DSL, huge training corpus |
| **Transforms** | Starlark (Python dialect, sandboxed, deterministic) |
| **Verification** | Four machine gates: verify, test, explain, operate |
| **Reliability** | Seven invariant tests, spool/settle/checkpoint engine on SQLite |
| **Jobs** | Cron scheduling, catchup windows, typed parameters, backfill |
| **Extension** | CEL → Starlark → WASM → gRPC out-of-process plugins |
| **Interop** | CESQL dialect (CloudEvents), official TCK 100% |

## The one-line pitch

> Eventboat lets AI agents build and run event pipelines that don't lose messages —
> because machines verify every step before it goes live.
