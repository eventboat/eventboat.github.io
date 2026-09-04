---
title: 架构
linkTitle: 架构
weight: 20
description: >-
    Eventboat 引擎的工作方式：三层管道模型、spool+settle+checkpoint 可靠性、四道验证关卡。
---

## 三层模型

```
YAML (+overlay) → Config（类型化）→ Static IR → Runtime Engine
                                        ↓
                            Source → Transform → Sink 插件
```

- **Config 层**：YAML 解析、严格 schema 校验、变量替换
- **Static IR**：校验后的 DAG + 预编译 CEL 程序 + Starlark 程序 + schema
- **Runtime**：spool + settle + checkpoint 引擎，只消费 IR

## 可靠性模型

```
source → [spool: 追加式持久队列] → 内存 DAG → sinks
              │                        │
              └── checkpoint ←── settle 跟踪 ←── 终态
```

- **Spool**：每条消息先落 SQLite 再进 DAG（不变量 1）
- **Settle**：消息的全部分支到达终态即 settle
- **Checkpoint**：只推进已 settle 的连续前缀（不变量 2）
- **崩溃恢复**：kill -9 → 重启 → 从 checkpoint 重放，绝不丢（不变量 3）
- **死信**：重试耗尽 → 死信库（可查询 + 可回放）

## 七条不变量测试

每条都有专属测试，CI 必须通过：

1. spool 先于可见
2. settle 先于 checkpoint
3. kill -9 重放覆盖全部未 settle
4. 死信写失败阻塞 settle
5. `required: false` 边不阻塞兄弟分支
6. 重复投递保持 message ID 稳定
7. 水位不超过已 settle 最大值

## 四道机器关卡

| 关卡 | 命令 | 做什么 |
|------|------|--------|
| **verify** | `eventboat verify` | Schema、拓扑、CEL+Starlark 编译、lint — 静态零副作用 |
| **test** | `eventboat test` | 对真实引擎跑合约测试 — fixture 进、断言出 |
| **explain** | `eventboat explain --message sample.json` | 确定性路径推演（真实 CEL 求值 + Starlark dry-run） |
| **operate** | `eventboat mcp` | MCP 服务器：15 个工具覆盖完整 Agent 生命周期 |
