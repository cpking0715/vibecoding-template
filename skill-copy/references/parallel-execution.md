# 并行工作流执行模式

本文档定义当 Phase 0.8 启用并行模式后，各 Phase 与串行模式的差异。
如果 Phase 0.8 评估为"建议串行"或用户选择了串行，忽略本文档。

## 总览

```
Phase 0-0.5: 与串行相同（PRD + Design + Acceptance）

Phase 0.8:   AI 评估 → 用户选择 → 进入并行模式

Phase 1-N:   以下各 Phase 的并行差异
```

## 核心差异

| 项目 | 串行模式 | 并行模式 |
|------|---------|---------|
| 分支策略 | 一个 feature/xxx 分支 | 每个 Workstream 独立分支 feature/<ws-name> |
| Phase 1 拆解 | 全局拆解成一个线性序列 | 每个 Workstream 独立拆解 |
| Phase 2 TDD | 逐 Step 串行执行 | 多个 Workstream 同时由 subagent 执行 |
| Phase 3 Agent Loop | 修复当前 Step | 修复后回到对应 Workstream |
| Phase 3.5 测试 | 单 Workstream 测试 | 先单个 Workstream 测试 → 合入后集成测试 |
| Phase 4 安全 | 单 PR 门禁 | 每个 Workstream PR 独立门禁 |
| Phase 5 文档 | 一次性生成 | 每个 Workstream 独立文档 |
| Phase 6 合入 | 单 PR → merge → tag | 多 PR 分层合入 → 最后打全局 tag |

## 依赖管理

### 接口冻结契约

并行模式下，跨 Workstream 的接口在设计阶段（Phase 0）一旦确认，写入 `.vibecoding/task-graph.md` 的"接口冻结"部分。
冻结期间任何 Workstream 不得单方面修改共享接口。

如需修改：
1. 提交 DAG-RFC 到 `.vibecoding/dag-rfcs/`
2. 调度器（主 Agent）评估影响范围
3. 通知所有受影响的下游 Workstream
4. 一致同意后解冻 → 修改 → 重新冻结

### 文件所有权

| Workstream | 拥有目录 | 消费者 |
|-----------|---------|--------|
| A | lib/a/ | B, C |
| B | lib/b/ | D |
| C | lib/c/ | D |

每个 Workstream 拥有明确的一组目录，不跨目录修改。

### 测试隔离

并行模式下的测试使用 mock/stub 模拟上游依赖，不依赖实际上游代码：

```
Workstream B（依赖 A）:
  - B 的单元测试 mock A 的接口
  - 不 import A 的实现代码
  - A 的 interface 冻结后，B 直接引用冻结的类型定义（可从 design.md 提取）
```

## Phase 差异详解

### Phase 1（并行版）

1. 读取 `.vibecoding/task-graph.md` 中的 Workstream 定义
2. 对每个 Workstream 独立执行 micro-step 拆解
3. 每个 Workstream 创建独立分支：`feature/<ws-name>`
4. 输出到 `.vibecoding/workstreams/<ws-name>/requirements.md`

### Phase 2（并行版）

1. 主 Agent 根据依赖波次调度 subagent
2. 无上游依赖的 Workstream（波次 1）立即启动
3. 有上游依赖的 Workstream（波次 N）等待上游 interface 的 commit
4. 每个 subagent 独立执行 TDD 循环
5. 每个 subagent 独立 commit 到自己的 Workstream 分支

```
主 Agent:
  ├─ [波次1] 启动 Subagent A: TDD → commit → push
  │          完成后通知主 Agent
  ├─ [波次2] A 的 interface commit 完成 → 启动 Subagent B
  │                                     → 启动 Subagent C
  └─ [波次3] B+C 完成 → 启动 Subagent D
```

### Phase 3.5（并行版）

分两阶段：

**阶段 1：单个 Workstream 测试**
- 每个 Workstream 在各自分支上跑 Phase 3.5 测试
- 测试范围仅限于本 Workstream 的代码（mock 外部依赖）

**阶段 2：集成测试**
- 所有 Workstream 合入到临时集成分支
- 跑跨 Workstream 的全链路 E2E 测试
- 使用真实 API 调用（不 mock）

### Phase 6（并行版）

**分层合入**：

```
Layer 1: 核心 Workstream（无上游依赖）→ 先合入
    └── 用户验收 → ok → merge
Layer 2: 业务 Workstream（依赖 Layer 1）→ 后合入
    └── 用户验收 → ok → merge
Layer 3: 聚合 Workstream（依赖 Layer 1+2）→ 最后合入
    └── 用户验收 → ok → merge → 打全局 tag vX.Y.Z
```

## 退出并行模式

任何时候用户或 Agent 发现并行导致问题（接口冲突、合并困难、上下文混乱），可以：

1. 主 Agent 记录当前所有 Workstream 状态到 `.vibecoding/parallel-state.md`
2. 逐个合入已完成的 Workstream
3. 未完成的 Workstream 拆解为串行 Micro-step 追加到主序列
4. 标记 `parallel_fallback` 到 evolution.md
