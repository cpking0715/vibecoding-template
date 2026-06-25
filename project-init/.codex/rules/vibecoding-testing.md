---
title: VibeCoding Comprehensive Testing
impact: HIGH
tags: vibecoding, testing, e2e, performance, security
---

## 综合验收测试

本规则定义 Phase 3.5 的测试命令和通过标准。项目初始化时根据设计文档和架构自动更新。

## 测试命令

| 测试类型 | 命令 | 说明 |
|---------|------|------|
| 业务功能测试 | `npm run test:e2e` 或 `npx vitest run e2e/` | E2E 全链路 |
| 安全测试 | `npm audit` | 依赖漏洞扫描 |
| 性能测试 | `npx autocannon -c 10 -d 10 http://localhost:3000/api/xxx` | 关键 API 压测 |
| 异常测试 | 手动构造异常输入 | 按场景执行 |
| 数据库测试 | `npm run db:rollback` 验证回滚 | 迁移可回滚 |

## 测试框架

| 项目类型 | 推荐 E2E 框架 | 性能工具 |
|---------|--------------|---------|
| Node.js / TypeScript | Vitest + supertest | autocannon |
| Python | pytest + httpx | locust |
| Go | testing + httptest | hey |

## 通过标准

- 功能测试：所有场景 Pass
- 安全测试：无 Critical/High 漏洞
- 性能测试：API p95 ≤ 500ms，错误率 ≤ 1%
- 异常测试：无 500 / crash / 数据损坏
- 数据库测试：迁移可回滚，备份可恢复
