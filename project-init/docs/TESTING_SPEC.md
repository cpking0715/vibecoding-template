# 综合验收测试规范

> 精简版。完整版见 Skill 内的 `references/testing-spec.md`。

## 测试顺序

```
Step 3.5a: 数据库备份 → Step 3.5b: 性能压力 → Step 3.5c: 功能 E2E
→ Step 3.5d: 异常测试 → Step 3.5e: 安全测试
```

## 适用性判断

| 测试类型 | 必须跑 | 可跳过 |
|---------|--------|--------|
| 功能测试 | 永远必须 | — |
| 安全测试 | 永远必须 | — |
| 性能测试 | design.md 标记高流量接口 | 简单 CRUD + 用户 < 100 |
| 异常测试 | 涉及外部依赖（DB/API/文件） | 纯计算函数 |
| 兼容性 | 修改了 API 契约 | 纯新增 |
| 数据库测试 | 有 migration | 无 migration |

## 通过标准

| 测试类型 | 通过标准 | 失败处理 |
|---------|---------|---------|
| 功能测试 | acceptance.md 所有场景 Pass | 回 Phase 3 Agent Loop |
| 安全测试 | 无 Critical/High 依赖漏洞；无越权/注入/XSS | `npm audit fix` 或回 Phase 2 |
| 性能测试 | API p95 ≤ 500ms，错误率 ≤ 1% | 检查 N+1 / 索引 / 缓存 |
| 异常测试 | 无 500 / crash / 数据损坏 | 回 Phase 2 对应 Step |
| 兼容性 | 已有 API 行为不变 | 回 Phase 2 修复 |
| 数据库测试 | 迁移可回滚，备份可恢复 | 检查 migration 文件 |

## 环境降级

| 条件 | 测试方式 |
|------|---------|
| 有 staging | E2E 在 staging 跑 |
| 无 staging | 本地 dev server |
| dev server 不可启动 | HTTP mock 集成测试 |

## 整体流程

全部适用测试通过 → Phase 4（安全门禁）。
任何不通过 → Phase 3 Agent Loop 修复。
