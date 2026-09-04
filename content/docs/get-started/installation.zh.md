---
title: 安装
linkTitle: 安装
weight: 10
description: >-
    在你的平台上安装 Eventboat CLI。
---

## 前提

- Go 1.25+（从源码构建）
- 零运行时依赖——全部编译进一个二进制

## 安装

```bash
go install github.com/eventboat/eventboat/cmd/eventboat@latest
```

验证：

```bash
eventboat help
```

应看到帮助界面，列出 11 个命令。

## 下一步

- [写第一条管道](../first-pipeline/)
- [CLI 参考](../../reference/commands/)
