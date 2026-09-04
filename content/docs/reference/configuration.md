---
title: Configuration
linkTitle: Configuration
weight: 20
description: >-
    Pipeline YAML sections, edge attributes, variable substitution, and the Runtime config.
---

## Top-level sections

| Section | Required | Purpose |
|---------|----------|---------|
| `apiVersion` / `kind` / `metadata` | ✅ | Resource identity (K8s convention) |
| `sources` | ✅ (≥1) | Topology: where events come from |
| `transforms` | optional | Topology: what happens to events |
| `sinks` | ✅ (≥1) | Topology: where events go |
| `run` | job pipelines only | Job scheduling (mode/schedule/overlap/catchup_window) |
| `parameters` | job optional | Typed job parameters with defaults |
| `constants` | optional | Read-only values visible to scripts and predicates |
| `hooks` | optional | Lifecycle hooks (failure/success → inline sink) |
| `limits` | optional | Per-pipeline resource limits |
| `edge_defaults` | optional | Default edge attributes |
| `codecs` | optional | Named codec declarations |
| `dlq` | optional | Dead-letter policy |

## Three-section topology

Nodes are organized by section; edges declared on the downstream side via `from`:

```yaml
sources:
  ingest:
    decoder: json
    kafka: { brokers: ["${KAFKA_BROKERS}"], topics: [orders] }

transforms:
  enrich:
    from: [ingest]                         # unconditional edge
    script: |
      payload.total = payload.price * payload.qty

sinks:
  eu-out:
    from: { enrich: { when: 'meta.region == "eu"' } }  # conditional edge
    kafka: { topic: orders-eu }
```

## Edge attributes

Attributes on `from` elements:

| Attribute | Type | Description |
|-----------|------|-------------|
| `when` | string or object | CEL predicate (or `{lang: cesql, expr: ...}`) |
| `delivery` | object | `{retries, backoff, timeout_ms}` |
| `required` | bool | `false` = best-effort (failure doesn't block siblings) |
| `buffer` | object | `{max_events, strategy}` |

## Variable substitution

- `${VAR}` — environment variable (unset = error)
- `${?VAR}` — optional (unset = omit key)
- `${constants.name}` — pipeline constant
- Applies to **all** string values

## Built-in plugins

### Sources
| Name | Key config |
|------|-----------|
| `kafka` | brokers, topics, group_id |
| `http_server` | listen, max_body_bytes |
| `cron` | expression (5-field) |
| `file` | path (tail) |
| `sql` | driver (mysql/postgres/sqlite), query, cursor, pagination |

### Sinks
| Name | Key config |
|------|-----------|
| `kafka` | brokers, topic |
| `http` | url, timeout_ms |
| `file` | path (JSON lines) |
| `drop` | (none — discards) |

### Codecs
| Name | Key config |
|------|-----------|
| `json` | (none) |
| `raw` | (none) |
| `csv` | columns or header |
| `avro` | schema (inline or file) |
| `protobuf` | descriptor_set (file path) |

## Job pipeline example

```yaml
apiVersion: eventboat/v3
kind: Pipeline
metadata: { name: nightly-sync }

run:
  mode: job
  schedule: "0 1 * * *"
  overlap: skip
  catchup_window: 2h
  skip_if_successful: true
  retention: { history: 90d }

parameters:
  from: { type: string, default: cursor }
  to:   { type: string, default: now }

sources:
  pull:
    sql:
      driver: mysql
      query: |
        SELECT * FROM orders
        WHERE updated_at >= :from AND updated_at < :to
      args: { from: "${parameters.from}", to: "${parameters.to}" }
      cursor: { column: updated_at }
      pagination: { key: [updated_at, id], page_size: 5000 }

transforms:
  enrich:
    from: [pull]
    script: |
      payload.source_system = constants.source_system

sinks:
  out:
    from: [enrich]
    kafka: { brokers: ["${KAFKA_BROKERS}"], topic: orders-sync }
```
