# VibeCoding 工作流 — Claude Code 入口

## 启动流程

每次新会话 / 新任务，按顺序执行：

1. **读取 `docs/SPEC.md`** — 产品边界，知道什么不能做
2. **读取 `.vibecoding/evolution.md`（最后 5 条）** — 了解当前进度
3. **读取 `.vibecoding/prd.md`** — 当前功能需求
4. **读取 `.vibecoding/design.md`** — 当前设计方案

## 核心规则

@VIBECODING-RULES.md

## 快速索引

| 场景 | 动作 |
|------|------|
| 遇到错误 | 先搜索 `docs/ERROR_LIBRARY.md`，匹配不到再走 Agent Loop |
| 涉及架构选型 | 先读 `docs/DECISIONS.md`，已有决策不得重新质疑 |
| PRD / 设计变更 | 按 `docs/CHANGELOG.md` 中的同步协议执行 |
| 不确定技术方案 | 停止编码，列出选项 + 推荐，等用户确认 |
| 宣告任务完成前 | 对照 Quality Gate 清单逐项确认 |
| 记忆文件写入规则 | 参考 `docs/MEMORY_STRUCTURE.md` |
