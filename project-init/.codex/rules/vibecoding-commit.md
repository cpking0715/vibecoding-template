---
title: VibeCoding Commit & PR Convention
impact: HIGH
tags: vibecoding, git, commit, pr
---

## 提交与合入规范

每个 Micro-step（或一组完成的 step）必须经过提交→PR→合入流程。

## Commit Message 格式

```
<type>(<scope>): <description>

[optional body]
```

| type | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 |
| refactor | 重构（测试保护下） |
| test | 仅测试代码变更 |
| docs | 文档变更 |
| chore | 工具/配置变更 |

## PR 描述

PR 描述按 `.github/PULL_REQUEST_TEMPLATE.md` 的格式生成。
该文件在项目初始化时从 Skill 参考模板复制；如果文件不存在，AI 直接生成 PR 描述。

## PR 模板内容

PR 描述必须包含：

```markdown
## 功能
[对应功能描述]

## Micro-step 完成
- [x] Step 1: [描述]
- [x] Step 2: [描述]

## 测试
- 单元测试: N passed, M failed（参见 acceptance.md）
- 综合验收: 功能/安全/性能/异常/兼容/数据库
- 安全扫描: 无敏感逻辑 / ⚠️ NEEDS HUMAN REVIEW

## 版本
- 合入后 tag: vX.Y.Z
```

## 合并策略

| 条件 | 策略 |
|------|------|
| 无安全标记 + CI 通过 | 自动合并 |
| 有安全标记 | 等待人工审核通过后合并 |
| CI 失败 | 禁止合并，回退到 Agent Loop |
