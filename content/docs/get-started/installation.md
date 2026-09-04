---
title: Installation
linkTitle: Install
weight: 10
description: >-
    Install the Eventboat CLI binary on your platform.
---

## Prerequisites

- Go 1.25+ (for building from source)
- No runtime dependencies — everything is compiled into one binary

## Install

```bash
go install github.com/eventboat/eventboat/cmd/eventboat@latest
```

Verify:

```bash
eventboat help
```

You should see the help screen with 11 commands listed.

## What's in the binary

| Component | Included |
|-----------|----------|
| Engine (spool/settle/checkpoint) | ✅ |
| CLI (verify/test/run/trigger/jobs/explain/replay/repl/lsp/plugin/mcp) | ✅ |
| Built-in sources (kafka/http_server/cron/file/sql) | ✅ |
| Built-in sinks (kafka/http/file/drop) | ✅ |
| Built-in codecs (json/raw/csv/avro/protobuf) | ✅ |
| MCP server (15 tools) | ✅ |
| LSP (diagnostics/completion/hover) | ✅ |
| Admin REST + SSE + read-only UI | ✅ |
| OpenTelemetry (OTLP + Prometheus) | ✅ |
| SQLite storage (pure Go, no CGO) | ✅ |

## Next steps

- [Write your first pipeline](../first-pipeline/)
- [CLI reference](../../reference/commands/)
