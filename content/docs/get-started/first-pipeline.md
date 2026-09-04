---
title: Your First Pipeline
linkTitle: First Pipeline
weight: 20
description: >-
    Write a three-section YAML pipeline, verify it, test it, and run it.
---

## The three-section format

Every Eventboat pipeline is a single YAML file with three top-level sections:
`sources`, `transforms`, and `sinks` — connected by `from` edges.

```yaml
apiVersion: eventboat/v3
kind: Pipeline
metadata: { name: my-first-pipeline }

sources:
  ingest:
    cron: { expression: "*/5 * * * *" }   # every 5 minutes

transforms:
  hello:
    from: [ingest]
    script: |
      payload.greeting = "hello from eventboat"
      payload.timestamp = meta.ingest_time

sinks:
  console:
    from: [hello]
    drop: {}                              # discard (demo)
```

## Verify (gate 1)

```bash
eventboat verify --config pipeline.yaml
```

Output: `pipeline.yaml: 0 error(s), 0 warning(s)`

## Run

```bash
eventboat run --config pipeline.yaml
```

The pipeline starts; every 5 minutes the cron source fires, the Starlark
script enriches the message, and the sink receives it.

## Add branching (CEL predicates)

```yaml
sinks:
  important:
    from: { hello: { when: 'payload.score > 100' } }
    http: { url: "https://api.example.com/alerts" }

  archive:
    from: [hello]                          # unconditional edge
    drop: {}
```

## Agent mode

```bash
# Start the MCP server — AI agents connect via stdio
eventboat mcp --stdio

# Or with the admin UI
eventboat mcp --http
```

## Next steps

- [Configuration reference](../../reference/configuration/)
- [CLI reference](../../reference/commands/)
- [Examples](https://github.com/eventboat/eventboat/tree/main/examples)
