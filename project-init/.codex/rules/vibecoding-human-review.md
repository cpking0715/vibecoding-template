---
title: VibeCoding Human Review Checkpoints
impact: HIGH
tags: vibecoding, security, review
---

## 人工审核哨兵

以下场景必须标记人工审核，不允许自动合并或自动部署：

| 场景 | 自动检测方式 | 具体触发条件 |
|------|------------|------------|
| 支付/计费 | AI 扫描含 amount/price/charge/subscription 的业务逻辑 | 涉及金额计算、订单生成、订阅逻辑、支付网关 |
| 认证/权限 | AI 扫描含 login/token/role/permission 的代码 | 登录、Token 签发、权限校验、角色管理 |
| 数据安全 | AI 扫描含 encrypt/decrypt/key/privacy 的处理 | 加密解密、密钥管理、个人隐私数据处理 |
| 架构变更 | AI 扫描含 table/schema/migration/version 的模式变更 | 数据库表结构修改、API 版本不兼容变更 |

## 触发机制（两步）

1. **AI 实时扫描**：代码提交前自动扫描 diff，检测敏感逻辑的关键词和上下文
2. **CI 门禁**：当 PR 包含 `⚠️ NEEDS HUMAN REVIEW` 标签时：
   - 自动将 PR 标记为 draft
   - 阻止 `merge` 按钮（GitHub Branch Protection 配合）
   - 发送通知到项目频道

## 标注方式

AI 自动在 PR 描述末尾追加：
```
⚠️ NEEDS HUMAN REVIEW: [原因，如：涉及支付金额计算]
```

## 自动通过

非上述场景的常规代码，AI 自动完成，无需人工介入。
