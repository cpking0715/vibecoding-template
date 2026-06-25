---
name: vibe-coding-workflow
description: >
  AI驱动的自动化开发工作流，实现从需求到部署的TDD闭环。
  当用户需要开发新功能、生成代码、进行TDD流程、或执行多轮AI代码生成-测试-修复循环时使用。
  适用于：新功能开发（CRUD/API/表单）、自动化代码生成、测试驱动开发、AI驱动的迭代开发。
  特点：强制PRD与设计方案、严格TDD（Red-Green-Refactor）、全链路验收测试、模型分层降本、多轮修复循环、安全人工审核哨兵、完整开发记忆。
---

# VibeCoding Workflow

## 触发条件

当用户提出以下请求时使用本 Skill：
- **新功能开发** — CRUD/API/表单/业务逻辑
- **代码生成** — 需要 AI 生成结构化代码
- **TDD 流程** — 测试驱动开发（Red-Green-Refactor）
- **迭代修复** — 需要多轮生成-测试-修复循环

## 工作流总览

```
Phase 0: PRD + Design（文档落地 → TDD 规则同步）
    │
    ▼
Phase 0.5: 验收场景列表（不含 Step 映射，Phase 1 补）
    │
    ▼
Phase 1: 拆解 → 补全 acceptance.md → 创建 feature 分支
    │
    ▼
Phase 2: TDD 循环（lint + typecheck + commit 到 feature 分支）
    │    ↑
    │    │ (修复)
    ▼    │
Phase 3: Agent Loop（新增 Phase 3.5 触发类型）
    │
    ▼
Phase 3.5: 综合验收测试（环境检查 → 按序执行 → 可跳过）
    │
    ▼
Phase 4: 安全哨兵（PR 门禁，与 Phase 3.5e 区分）
    │
    ▼
Phase 5: 文档生成
    │
    ▼
Phase 6: 推送 → PR → 合并 → tag → 清理分支
    │
    ▼
【每次 Phase 完成 → 追加到 .vibecoding/evolution.md】
```

## 引用资源

- **模型分层策略**: references/model-tiering.md
- **需求输入模板**: references/requirements-template.md
- **Micro-step 拆解规范**: references/micro-step-standards.md
- **Agent Loop 规格**: references/agent-loop.md
- **记忆文件结构**: references/memory-structure.md
- **综合验收测试**: references/testing-spec.md
- **PR 模板**: references/pr-template.md
- **并行执行模式**: references/parallel-execution.md

## 项目无关性

本 Skill 是通用的。唯一因项目而异的是 `.codex/rules/vibecoding-tdd.md` 中的测试框架和路径别名。
这些在 Phase 0 完成后根据 `.vibecoding/design.md` 自动检测并同步。

---

## 首次使用

如果 `.vibecoding/` 目录不存在，Codex 在进入 Phase 0 时自动创建。

### TDD 上下文自动同步

首次 Phase 0 完成（design.md 生成后），Codex 自动检测项目上下文并更新 `.codex/rules/vibecoding-tdd.md`：

| 检测来源 | 提取内容 |
|---------|---------|
| package.json | 运行时、测试框架 |
| tsconfig.json | 路径别名 |
| 项目目录结构 | 代码布局 |
| docker-compose / Dockerfile | 数据库类型 |
| design.md | 架构决策 |

如果检测失败，Codex 会要求用户补充。

---

## Phase 0: 需求文档与设计方案

本阶段完成后进入 Phase 0.5。每个 Step 都必须用户确认。

### Step 0a: 创建 PRD

1. 要求用户按 references/requirements-template.md 填写需求
2. 生成为 `.vibecoding/prd.md`，包含：
   - **Business Context**
   - **Scope**: In / Out
   - **功能描述**
   - **边界条件**（≥2）
   - **业务规则**（≥1）
3. 用户确认后才能进入 Step 0b
4. 需求变更时回到本步骤，变更原因写入 evolution.md

### Step 0b: 创建设计方案

1. 生成 `.vibecoding/design.md`，包含：
   - **数据模型** — 核心 interface/type 定义
   - **接口契约** — 函数签名、API 路由、请求/响应格式
   - **架构决策** — 做了什么选择、为什么这么选
   - **组件关系** — 模块依赖、数据流方向

> 使用 T3 模型（Opus）进行架构设计。架构决策直接影响后续所有实现，需要最高推理能力。
2. 用户确认后才能进入 Phase 0.5
3. 实现中发现设计有问题时，更新 design.md 并记录到 evolution.md
4. **最后：运行 TDD 上下文自动同步**

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 0 | PRD
- 创建 PRD / Design / Acceptance
- TDD 上下文已同步（框架: vitest, 别名: @/, 数据库: postgres）
```

---

## Phase 0.5: 验收场景列表

**创建场景但不关联 Step**（因为 Step 还没拆解）。关联在 Phase 1 完成。

1. 生成 `.vibecoding/acceptance.md`，包含：
   - 所有 Given-When-Then 场景的表格
   - 每行：**编号、Given、When、Then、状态（Pending/Done/Blocked）**
   - **"对应 Step"列先空着** — Phase 1 拆解后补
2. 用户确认

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 0.5 | 验收场景
- 已创建 N 个验收场景（Step 映射待 Phase 1 补全）
```

---


---

## Phase 0.8: 并行工作流评估（可选）

本阶段为可选。AI 根据设计文档评估是否可拆分为多个独立 Workstream 并行执行，
输出评估结论后由用户决定是否启用并行模式。

### Step 0.8a: 依赖图分析

1. 读取 .vibecoding/design.md，分析模块边界
2. 识别 Workstream 之间的依赖关系（接口依赖、数据模型依赖）
3. 输出 .vibecoding/task-graph.md，包含：
   - Workstream 列表及每个 Workstream 的依赖项
   - 文件所有权分配
   - 并行波次图
   - 接口冻结契约

### Step 0.8b: 方案评估

AI 输出评估结论：

| 评估项 | 结果 |
|--------|------|
| 是否可并行 | ✅ 可并行 / ❌ 建议串行（原因） |
| 可拆分为几个 Workstream | N 个 |
| 预估加速比 | 串行 Xh → 并行 Yh |
| 主要风险 | 接口冻结 / 合并冲突 / 验收瓶颈 |

### Step 0.8c: 用户决策

- AI 评估可并行 → 用户选择**并行**或**串行**
- AI 评估建议串行 → 直接进入 Phase 1（串行模式）
- 选择并行后 → 后续 Phase 按 
eferences/parallel-execution.md 的并行模式执行
- 选择串行后 → 后续 Phase 按标准串行流程执行

### 记忆记录

```
## YYYY-MM-DD HH:mm | Phase 0.8 | 并行评估
- 评估结论: [可并行/建议串行]
- 用户选择: [并行/串行]
- Workstream 数: N
```

## Phase 1: 需求拆解

> 如果 Phase 0.8 启用了并行模式，本阶段改为为每个 Workstream 独立拆解。详见
`references/parallel-execution.md`。

> 任务拆解涉及对需求的理解和架构分析，使用 T2/T3 模型（Sonnet/Opus）进行。


1. 从 `.vibecoding/prd.md` 读取需求
2. 按 references/micro-step-standards.md 拆解为 Micro-step
3. **输出拆解列表让用户确认**，确认后：
   - 保存到 `.vibecoding/requirements.md`
   - **更新 `.vibecoding/acceptance.md`，补全"对应 Step"列**
4. 用户不确认时重新拆解
5. 拆解确认后创建功能分支：
   - `git checkout -b feature/<feature-name>`
   - 后续所有 commit 落在这个分支上

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 1 | 拆解
- 拆解为 N 个 Micro-step
- acceptance.md "对应 Step"列已补全
- 分支 feature/xxx 已创建
```

---

## Phase 2: TDD 循环（Red-Green-Refactor）

对每个 Micro-step 严格执行。所有 commit 落在 `feature/<name>` 分支上。

### Step 2a: 验收标准
- 从 `.vibecoding/acceptance.md` 取当前 step 对应的场景
- 用 Given-When-Then 输出
- 用户确认

### Step 2b: Red（编写失败测试）
- 仅写当前 step 的测试，最多 3 个 test case
- 启动 T1 Subagent（Haiku / Qwen-Turbo）生成测试代码。主 Agent 传入需求上下文，Subagent 返回测试文件后主 Agent 继续。
- 生成后**立即停止**，提示用户运行测试确认失败
- 如果测试代码有编译错误，进 Agent Loop

### Step 2c: Green（实现最小代码）
- 仅写能让测试通过的最少代码
- 路由模型：CRUD→主 Agent 直接执行（T2 Sonnet），核心→主 Agent 深度推理（T3 Opus），安全/支付逻辑→启动 T3 Subagent
- 运行测试确认全部通过
- 失败则进 Agent Loop

### Step 2d: Refactor（重构）
- 允许：Extract Method / Rename / Simplify Conditional / Extract Constant
- 禁止：改接口签名、改外部行为、架构级变更
- 重构后重新运行测试

### Step 2e: 提交当前 Step（质量门禁）
- 更新代码内注释
- 更新项目的功能映射文档
- **运行质量检查**（如果项目配置了以下命令）：
  - 运行项目的 lint 命令（npm run lint / pylint / golangci-lint 等）→ 修复错误
  - 运行项目的类型检查命令（npm run typecheck / mypy / go vet 等）→ 修复错误
- 更新 acceptance.md ✅ Done
- 追加到 evolution.md（含 commit hash）
- `git add -A && git commit -m "<type>(<scope>): Step N - <description>"`
- 每个 Step 独立 commit → `git revert <hash>` 精确回滚

### Step 2f: 循环
- 询问用户是否进入下一个 Micro-step
- 全部完成后进入 Phase 3.5

---

## Phase 3: Agent Loop（迭代修复）

详见 references/agent-loop.md。

### 进入条件

| 触发阶段 | 条件 | 处理 |
|---------|------|------|
| Red | 测试代码编译/import 错误 | 修复测试本身 |
| Green | 实现未通过测试 | 修复实现 |
| Refactor | 重构引入回归 | 修复回归 |
| Phase 3.5 | 功能/性能/异常测试失败 | 见 agent-loop.md 新增类型 |

Red 阶段的**预期失败**不进 Loop。

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 3 | Agent Loop
- 触发阶段: [Red/Green/Refactor/Phase3.5]
- 错误类型: [AssertionError/ImportError/Timeout/E2E/Perf]
- 根因: [分析]
- 修复模型: [模型名]（第N次）
- 用户通知: 已发送（第N次失败）
```

---

## Phase 3.5: 综合验收测试

所有 Micro-step 完成后，对完整功能进行全链路验收测试。
详细规范、通过标准、失败处理见 `references/testing-spec.md`。

### 环境就绪检查

| 条件 | 测试方式 |
|------|---------|
| 有 staging 环境 | E2E 在 staging 跑 |
| 无 staging | 本地启动 dev server |
| dev server 不可启动 | HTTP mock 集成测试 |
| 无 E2E 框架 | T1 Subagent 生成 HTTP 请求脚本 |

### 适用性判断

AI 根据设计文档判断，用户确认后可跳过不适用的测试类型：

| 测试类型 | 必须跑 | 可跳过 |
|---------|--------|--------|
| 功能测试 | 永远必须 | — |
| 安全测试 | 永远必须 | — |
| 性能测试 | design.md 标记高流量接口 | 简单 CRUD + 用户数 < 100 |
| 异常测试 | 有外部依赖（DB/第三方API） | 纯计算函数 |
| 兼容性 | 修改了 API 契约 | 纯新增 |
| 数据库测试 | 本功能有 migration | 无 migration |

### Step 3.5a: 数据库与备份测试（先跑，保环境干净）

### Step 3.5b: 性能与压力测试（在干净环境跑基准）

### Step 3.5c: 业务流程功能测试（正常路径 E2E）

### Step 3.5d: 异常测试（在功能测试基础上模拟故障）

### Step 3.5e: 安全测试（最后跑，可能涉及破坏性操作）

### 整体通过标准

全部适用测试通过 → 进入 Phase 4。
任何不通过 → 回到 Phase 3 Agent Loop。

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 3.5 | 验收测试
- 功能测试: N passed, 0 failed
- 安全测试: npm audit 0 critical
- 性能测试: API p95 120ms (跳过: 不需要)
- 异常测试: 6 scenarios, all passed
- 数据库: migration rollback OK (跳过: 无 migration)
```

---

## Phase 4: 安全哨兵（PR 门禁）

> 安全审查使用 T3 模型（Opus）进行代码级安全分析，确保零遗漏。

**与 Phase 3.5e 的区分**：
- Phase 3.5e = **主动安全测试**：扫描依赖漏洞、检查代码越权/注入/XSS、发现并修复问题
- Phase 4 = **PR 安全门禁**：轻量扫描 git diff，检测本次变更是否涉及敏感逻辑，如果涉及则标记 PR 需人工审核

AI 自动扫描变更代码 diff：

| 敏感类别 | 自动检测 | 触发处理 |
|---------|---------|---------|
| 支付/计费 | amount/price/charge/subscription | PR 标记 + CI 门禁 |
| 认证/权限 | login/token/role/permission | 同上 |
| 数据安全 | encrypt/decrypt/key/privacy | 同上 |
| 架构变更 | table/schema/migration/version | 同上 |

非敏感代码自动通过。

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 4 | 安全门禁
- 扫描结果: 无敏感逻辑
- Phase 3.5e 已先执行依赖+代码扫描（0 问题）
```

---

## Phase 5: 文档生成

启动 T1 Subagent（Haiku / Qwen-Turbo）基于最终代码生成。主 Agent 传入代码路径，Subagent 返回文档。如果目录/文件不存在则自动创建。

| 变更类型 | 位置 |
|---------|------|
| 新增/修改 API | `docs/api/<module>.md` |
| 新增配置项 | `.env.example` + `README.md` |
| 修改外部行为 | `README.md` 变更摘要 |
| 新增依赖 | `README.md` 依赖章节 |

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 5 | 文档
- docs/api/xxx.md 已创建
- README 已更新
```

---

## Phase 6: 推送与合入

代码由 Phase 2 Step 2e 逐 Step 提交到 `feature/<name>` 分支。

### Step 6a: 推送远程
- `git push origin feature/<name>`

### Step 6b: 创建 PR
- 分支 `feature/<name>` → `main`
- 描述按 `.github/PULL_REQUEST_TEMPLATE.md` 格式（如文件不存在则 AI 直接生成）
> PR 描述由主 Agent 使用 T2 模型（Sonnet）生成，人工确认后提交。
- 有安全标记自动 draft

### Step 6c: 合并
- 无安全标记 + CI 通过 → 自动合并
- 有安全标记 → 人工审核后合并
- CI 失败 → 回 Phase 3

### Step 6d: 打版本标签
- 先读取当前最新版本：`git tag --list 'v*' | sort -V | tail -1`（没有则从 v0.1.0 开始）
- 然后 `git tag v<major>.<minor>.<patch> && git push --tags`
- 版本号规则：
  - `fix` → patch（v1.0.0 → v1.0.1）
  - `feat` → minor（v1.0.0 → v1.1.0）
  - break change → major（v1.0.0 → v2.0.0）

### 分支清理
合并并打完 tag 后删除 feature 分支。

### 记忆记录

```
## 2026-06-22 HH:mm | Phase 6 | 合入
- 分支: feature/xxx → main
- Commit 数: N
- 安全标记: 无
- 合并: 自动
- Tag: v1.2.0
```

---

## 模型使用总规则

- 每步引用 model-tiering.md 决定模型
- 禁止越级升级，允许降级
- 每次升级记录到 `.vibecoding/logs/model-upgrades.log`
- 每个 Phase 完成后必须追加 evolution.md
