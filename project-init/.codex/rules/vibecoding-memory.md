---
title: VibeCoding Memory & Evolution Log
impact: HIGH
tags: vibecoding, memory, evolution, documentation
---

## 开发记忆机制

每个功能必须维护 `.vibecoding/` 下的文档文件，作为 AI 的"长期记忆"。

## 必须维护的文件

| 文件 | 用途 |
|------|------|
| `.vibecoding/prd.md` | 产品需求文档 |
| `.vibecoding/design.md` | 设计方案与架构决策 |
| `.vibecoding/acceptance.md` | 验收标准场景列表 |
| `.vibecoding/evolution.md` | 追加式开发记忆（永不覆盖） |

## 读取顺序

AI 启动工作流时：
1. 先读 evolution.md（后 5 条）了解上下文
2. 再读 prd.md 了解需求
3. 再读 design.md 了解设计方案

## 写入规则

- evolution.md 永远是 **追加写入**，不允许覆盖或编辑历史条目
- prd.md 和 design.md 是 **覆盖写入**（但变更原因要记录到 evolution.md）
- 每个 Phase 完成后必须追加一条 evolution.md 记录
