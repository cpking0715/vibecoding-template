# VibeCoding Workflow Rules

## 模型分层策略

| 任务类型 | 推荐模型 | 理由 |
|---------|---------|------|
| 测试生成、文档 | Haiku / Qwen-Turbo | 结构化输出，省钱 |
| 常规代码、重构 | Sonnet / DeepSeek-V3 | 主力模型，质量与成本平衡 |
| 安全、支付、架构 | Opus | 需要高推理能力 |

路由原则：从 T1→T2→T3 逐级升级（不准跳级），允许主动降级（节省成本）。

## 流程总览

Phase 0(PRD+Design) → Phase 1(拆解) → Phase 2(TDD) ⇄ Phase 3(修复) → Phase 3.5(综合验收测试) → Phase 4(安全) → Phase 5(文档) → Phase 6(合入)

【每个 Phase 完成后追加 evolution.md】

## TDD 工作流（Red-Green-Refactor）

严格遵循 Red-Green-Refactor：
1. **拆解需求** → Micro-steps（五原则）
2. **验收标准** → Given-When-Then
3. **Red** → 仅写测试，停等确认
4. **Green** → 仅写最少实现代码
5. **Refactor** → 只做 extract/rename/simplify
6. **每个 Step 独立 commit** → 可精确回滚

**禁令**：禁止在测试验证通过前写业务代码；禁止跳过测试运行；禁止一次性处理过大需求。

## 综合验收测试（Phase 3.5）

全链路验证，详见 testing-spec.md。

| 测试类型 | 自动程度 | 通过标准 |
|---------|---------|---------|
| 业务功能测试 | 可自动 | acceptance.md 所有场景 Pass |
| 安全测试 | 自动 | 无 Critical/High 依赖漏洞 |
| 性能/压力/稳定性 | 半自动 | API p95 ≤ 500ms, 错误率 ≤ 1% |
| 异常测试 | 手动 | 无 500/crash/数据损坏 |
| 兼容性 | 半自动 | 目标环境正常 |
| 数据库备份 | 手动 | 迁移可回滚，备份可恢复 |

全部通过后才允许进入文档和合入阶段。任何不通过 → 回到 Phase 3 修复。

## Agent Loop（迭代修复）

修复循环，区分触发阶段：Red(测试错) / Green(实现错) / Refactor(回归)。
重试：T2(1-2次) → T3(1次) → 人工。

## 需求模板

所有新功能需求必须包含：功能描述、边界条件(≥2)、业务规则(≥1)。

## 安全审核哨兵

AI 自动扫描，检测支付/认证/安全/架构变更后自动标记 PR + CI 门禁。

## 提交与合入

格式：`<type>(<scope>): <description>`
合并：无安全标记→自动合并；有标记→人工审核。

## 文档

新功能完成时生成文档，按约定位置写入（docs/api/、.env.example、README）。

## 开发记忆

- 每个功能维护 `.vibecoding/prd.md`、`design.md`、`acceptance.md`、`evolution.md`
- evolution.md 只追加不覆盖
- AI 启动时先读 evolution.md 取上下文
