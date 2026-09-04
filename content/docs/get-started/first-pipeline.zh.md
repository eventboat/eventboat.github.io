---
title: 第一条管道
linkTitle: 第一条管道
weight: 20
description: >-
    写一个三段式 YAML 管道，验证、测试并运行。
---

## 三段式格式

每个 Eventboat 管道是一个 YAML 文件，三个顶层段：`sources`、`transforms`、
`sinks`——通过 `from` 连边。

```yaml
apiVersion: eventboat/v3
kind: Pipeline
metadata: { name: my-first-pipeline }

sources:
  ingest:
    cron: { expression: "*/5 * * * *" }   # 每 5 分钟

transforms:
  hello:
    from: [ingest]
    script: |
      payload.greeting = "hello from eventboat"
      payload.timestamp = meta.ingest_time

sinks:
  console:
    from: [hello]
    drop: {}                              # 丢弃（演示用）
```

## 验证（关卡 1）

```bash
eventboat verify --config pipeline.yaml
```

输出：`pipeline.yaml: 0 error(s), 0 warning(s)`

## 运行

```bash
eventboat run --config pipeline.yaml
```

管道启动；每 5 分钟 cron 源触发，Starlark 脚本丰富消息，sink 接收。

## 加分支（CEL 谓词）

```yaml
sinks:
  important:
    from: { hello: { when: 'payload.score > 100' } }
    http: { url: "https://api.example.com/alerts" }

  archive:
    from: [hello]                          # 无条件边
    drop: {}
```

## Agent 模式

```bash
# 启动 MCP 服务器 — AI Agent 经 stdio 连接
eventboat mcp --stdio

# 或带管理界面
eventboat mcp --http
```
