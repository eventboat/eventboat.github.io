---
title: CLI 参考
linkTitle: 命令
weight: 10
description: >-
    全部 11 个 Eventboat CLI 命令及旗标。
---

## 总览

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

静态验证管道。检查 schema、拓扑不变量（无环、无孤立节点、源无入边、汇无出边、
至少一条源→汇通路）、CEL 谓词编译、Starlark 脚本编译、作业配置和语义 lint。

| 旗标 | 默认值 | 说明 |
|------|--------|------|
| `--config` | （必填） | 管道 YAML 文件 |
| `--strict` | false | 将 warning 升级为 error |

## test {#test}

对真实进程内引擎跑合约测试。测试文件声明注入点、期望捕获和死信断言。

## run {#run}

执行管道。作业管道（含 `run.mode: job`）在作业管理器下运行，带调度和补偿。
`--config-dir` 启动多管道守护进程（含管理界面）。

## trigger {#trigger}

手动触发作业管道一次，可选传参（回补）。

```bash
eventboat trigger --config sync.yaml --parameters '{"from":"2026-08-01","to":"2026-09-01"}'
```

## explain {#explain}

管道的确定性推演。带 `--message` 时做真实 CEL 求值和 Starlark dry-run；
带 `--topology` 时渲染 DAG（mermaid + ASCII）。

## replay {#replay}

将死信（`--dlq`）、spool 窗口（`--spool --from N`）或某次运行的死信
（`--job <run-id>`）重新注入活跃管道。

## repl {#repl}

不跑管道，对样例消息求值 CEL 谓词或执行 Starlark 脚本。

## mcp {#mcp}

启动面向 AI Agent 的 MCP 服务器。

| 旗标 | 说明 |
|------|------|
| `--stdio` | 经 stdin/stdout 说 MCP（供 Agent 宿主拉起） |
| `--http` | HTTP 形态 MCP + Admin REST + SSE + 管理界面 |
