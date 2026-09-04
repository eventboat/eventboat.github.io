---
title: 什么是 Eventboat？
linkTitle: 概述
weight: 10
description: >-
    Go 单二进制 DAG 事件路由器，为 AI Agent 端到端操作而设计。
---

Eventboat 通过有向无环图（DAG）路由事件——从**源**（Kafka、HTTP、cron、SQL、文件）
到**汇**（Kafka、HTTP、文件），保证 at-least-once 投递、死信可查可回放。

它是 **Agent 原生**的：所有能力都可通过 MCP（Model Context Protocol）、CLI 和语言服务器
访问——AI Agent 可以自主编写、验证、部署和运维事件管道。

## 有什么不同

| 维度 | 方案 |
|------|------|
| **谓词** | CEL（K8s 标准）——零自研 DSL，训练语料巨大 |
| **转换** | Starlark（Python 方言，沙箱，确定性） |
| **验证** | 四道机器关卡：verify、test、explain、operate |
| **可靠性** | 七条不变量测试，spool/settle/checkpoint 引擎（SQLite） |
| **作业** | cron 调度、补偿窗口、类型化参数、回补 |
| **扩展** | CEL → Starlark → WASM → gRPC 进程外插件 |
| **互操作** | CESQL 方言（CloudEvents），官方 TCK 100% |

## 一句话

> Eventboat 让 AI Agent 构建和运行不丢消息的事件管道——因为机器在上线前验证每一步。
