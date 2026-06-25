---
title: VibeCoding Documentation
impact: MEDIUM
tags: vibecoding, documentation
---

## 文档自动生成

每个新功能完成时必须生成或更新以下文档：

| 变更类型 | 文档位置 | 内容要求 |
|---------|---------|---------|
| 新增/修改 API | `docs/api/<module>.md` | 路径、参数、返回值、错误码 |
| 新增配置项 | `.env.example` + `README.md` 配置章节 | 变量名、类型、默认值、作用 |
| 修改外部行为 | `README.md` 变更摘要 | 变更原因、影响范围、迁移指南 |
| 新增依赖 | `README.md` 依赖章节 | 包名、版本、用途 |

## 位置约定

- API 文档统一放在 `docs/api/` 目录下，按模块命名 `docs/api/auth.md`
- 如果 `docs/` 目录不存在，在项目根目录创建
- 如果对应文件已存在，追加新内容而非覆盖

## 生成方式

使用 T1 模型（Haiku / Qwen-Turbo）基于最终代码的静态分析生成文档。
