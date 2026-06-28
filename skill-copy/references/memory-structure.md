# .vibecoding/ 目录结构与记忆机制

本文档定义 `.vibecoding/` 下所有文件的用途、创建时机、更新规则。

## 新项目初始化

### Step 1: 确认 Skill 已安装
Skill 已全局安装在 `~/.codex/skills/vibe-coding-workflow/`，无需重复安装。

### Step 2: 复制项目规则模板
```
.codex/rules/       → 从参考文件复制，按项目修改
.cursor/rules/      → 同上
docs/               → 从参考文件复制（SPEC、DECISIONS、ERROR_LIBRARY、CHANGELOG）
VIBECODING-RULES.md → 从参考文件复制
CLAUDE.md           → @VIBECODING-RULES.md
.github/PULL_REQUEST_TEMPLATE.md → 从 pr-template.md 复制
```

### Step 3: TDD 上下文自动同步
首次 Phase 0 完成后自动检测并更新 `vibecoding-tdd.md`。

### Step 4: 确认 `.vibecoding/` 可写且已加入 `.gitignore`
目录在 Phase 0 自动创建。**必须确保 `.vibecoding/` 已被添加到 `.gitignore`**（运行状态不提交）。

```
# .gitignore
.vibecoding/
```

---

## 目录结构

```
.vibecoding/
├── prd.md               当前功能的 PRD
├── design.md            设计方案与架构决策
├── acceptance.md        验收场景列表（Given-When-Then）
├── evolution.md         项目级开发记忆（追加写入，永不覆盖）
├── state.json           当前循环状态（会话级，每次启动读取）
├── requirements.md      Micro-step 拆解结果
├── task-graph.md         工作流依赖图与接口契约（Phase 0.8 并行模式时创建）
├── report.md            阻塞报告（仅 blocked 时存在）
└── logs/
    ├── model-upgrades.log   首次升级时自动创建
    ├── failure-1.md         第 N 次失败分析
    ├── flaky-tests.md       Flaky test 记录
    └── failure-N.md
```

## 各文件定义

### prd.md — 产品需求文档

创建于 Phase 0，需求变更时更新（覆盖）。变更原因必须记入 evolution.md。

### design.md — 设计方案

创建于 Phase 0，设计变更时更新（覆盖）。变更原因必须记入 evolution.md。

### acceptance.md — 验收场景列表

创建于 Phase 0.5（不含 Step 映射）。Phase 1 拆解后补全"对应 Step"列。
每 Step 完成后将对应行标记为 ✅ Done。

更新规则：**追加行**（不覆盖已有历史）。

### evolution.md — 开发记忆

**永不覆盖，只追加**。第一次写入时自动创建。

记录格式：
```
## YYYY-MM-DD HH:mm | Phase N | 阶段名
- 关键信息: ...
- 结果: ...
```

### 归档策略

当 evolution.md 超过 **200 行** 时：
1. 将当前 evolution.md 移入 `.vibecoding/archive/evolution-YYYY-MM.md`
2. 新建 evolution.md，首行记录"前档案：archive/evolution-YYYY-MM.md"

```
.vibecoding/
├── archive/
│   ├── evolution-2026-06.md    ← 历史归档
│   └── evolution-2026-07.md
├── evolution.md                 ← 当前（从 0 开始）
```

### state.json — 当前循环状态

会话级，每次启动读取，每次状态变化覆盖。**不跨会话持久化**。

### logs/ — 运行日志

| 文件 | 创建时机 | 说明 |
|------|---------|------|
| model-upgrades.log | 首次跨级升级 | 记录升级原因 |
| failure-N.md | 每次 Agent Loop 失败 | 失败分析 |
| flaky-tests.md | 首次检测到 flaky test | Flaky test 列表 |

---

## docs/ 全局参考文档

项目根目录的 `docs/` 存放全局参考文件，与 `.vibecoding/`（按功能组织）互补：

```
docs/
├── SPEC.md               产品身份 + 核心功能 + 非目标
├── DECISIONS.md          架构决策记录（防止 AI 反复质疑）
├── ERROR_LIBRARY.md      领域错误模式库（快速匹配修复方案）
└── CHANGELOG.md          需求/设计变更记录 + 设计同步协议
```

| 文件 | 写入方式 | 说明 |
|------|---------|------|
| SPEC.md | 覆盖 | 产品全局定义，不是按功能的 |
| DECISIONS.md | 追加 | 每次架构决策后追加 |
| ERROR_LIBRARY.md | 追加 | 每次踩坑后追加 |
| CHANGELOG.md | 追加 | 每次 PRD/设计变更后追加 |

### CHANGELOG vs evolution.md

| 维度 | evolution.md | CHANGELOG.md |
|------|-------------|-------------|
| 记录什么 | 开发进度（Phase/Step 完成） | 需求/设计变更 |
| 更新频率 | 每个 Phase 完成后 | 每次 PRD/设计变更时 |
| 谁写 | AI 自动追加 | AI + 人工确认 |
| 关键内容 | 阶段完成 + 关键决策 | 变更内容 + 原因 + 影响分析 + 同步检查 |

---

## 读取规则

AI 启动时按顺序读取：

1. **docs/SPEC.md** → 产品全局边界（非目标禁止实现）
2. **evolution.md**（最后 5 条）→ 进度和历史
3. **state.json** → 当前会话状态
4. **prd.md** → 需求
5. **design.md** → 设计方案

AI 遇到错误时：
1. **先搜索 docs/ERROR_LIBRARY.md** → 匹配已知模式
2. 无匹配 → 按 Agent Loop 流程排查
3. 修复后 → 追加 ERROR_LIBRARY.md

## 写入规则

| 文件 | 创建 | 更新 | 写入方式 |
|------|------|------|---------|
| docs/SPEC.md | 项目初始化 | 产品方向变更 | 覆盖 |
| docs/DECISIONS.md | 首次架构决策 | 每次决策 | 追加 |
| docs/ERROR_LIBRARY.md | 首次踩坑 | 每次踩坑 | 追加 |
| docs/CHANGELOG.md | 首次变更 | 每次 PRD/设计变更 | 追加 |
| prd.md | Phase 0 | 需求变更 | 覆盖 |
| design.md | Phase 0 | 设计变更 | 覆盖 |
| acceptance.md | Phase 0.5 | 每 Step 完成 | 追加行 |
| evolution.md | 首次 Phase 完成 | 每阶段完成 | **追加（永不覆盖）** |
| evolution.md 归档 | >200 行 | 自动 | 移入 archive/ |
| state.json | 每次循环 | 每次状态变化 | 覆盖 |
| requirements.md | Phase 1 | — | 覆盖 |
| model-upgrades.log | 首次升级 | 每次升级 | 追加 |
| failure-N.md | 首次失败 | 每次失败 | 创建 |
| flaky-tests.md | 首次 flaky | 每次检测到 | 追加 |
