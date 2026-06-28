# 错误模式库

> **目的**：把踩过的坑记录下来，让 AI 下次遇到同类错误时能快速定位和修复。
>
> **写入时机**：每次花了超过 10 分钟才修复的错误，都应该追加一条。
> **使用方式**：AI 遇到错误时先搜索本文件，匹配失败模式 → 直接用已知方案修复。

---

## 约定

每条错误记录包含：

| 字段 | 说明 |
|------|------|
| **ID** | `{域名}-{编号}`，如 `DB-001`、`COMFYUI-001` |
| **错误特征** | AI 可以通过 grep/stderr 匹配到的关键信息 |
| **根因** | 为什么会发生 |
| **修复步骤** | 按顺序执行的操作 |
| **预防** | 如何避免再次发生 |
| **发生次数** | 每遇到一次 +1（高频错误需要重点关注） |

---

## 错误记录

<!-- 示例（项目初始化后替换为真实错误）：

### COMFYUI-001 | Workflow Node 找不到

**错误特征**：
```
Node not found: "KSampler"
Workflow validation failed
```

**根因**：ComfyUI 的 Custom Node 版本与 workflow 文件中引用的版本不一致。

**修复步骤**：
1. `cd ComfyUI/custom_nodes/` 检查缺失的 Node 包
2. `git log --oneline -5` 查看最近的版本更新
3. `git checkout <稳定版本 tag>` 回滚到 workflow 确认可用的版本
4. 重启 ComfyUI
5. 在 ComfyUI Manager 中确认所有依赖 Node 已加载

**预防**：
- Workflow 导出时附带 `custom_nodes_versions.json`
- 升级 Custom Node 前先在测试环境验证
- 生产 ComfyUI 锁定 Custom Node 版本

**发生次数**：1

---

### DB-001 | Migration 冲突

**错误特征**：
```
ERROR: relation "xxx" already exists
Migration failed: duplicate column
```

**根因**：多人分支同时创建了 migration，合并后 migration 文件时间戳冲突或操作重复。

**修复步骤**：
1. `npx prisma migrate status`（Prisma）或 `alembic history`（SQLAlchemy）查看 migration 状态
2. 定位冲突的 migration 文件
3. 重命名或合并冲突的 migration
4. `npx prisma migrate dev` 重新运行
5. 验证 DB schema 与期望一致

**预防**：
- 每个功能分支只允许创建 1 个 migration
- 合并前先 rebase 并检查 migration 顺序
- Migration 文件命名包含功能标识

**发生次数**：1

-->

<!-- 新错误记录追加在下方 -->

---

## 高频错误 TOP 3

<!-- 定期更新，帮助 AI 快速判断最常见的问题 -->

| 排名 | ID | 问题 | 次数 |
|------|-----|------|------|
| 1 | — | — | — |
| 2 | — | — | — |
| 3 | — | — | — |
