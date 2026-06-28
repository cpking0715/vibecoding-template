# 开发记忆文件结构

> 精简版。完整版见 Skill 内的 `references/memory-structure.md`。

## 项目文件地图

```
docs/                    全局参考（项目级，可提交）
├── SPEC.md              产品身份 + 非目标
├── DECISIONS.md          架构决策记录
├── ERROR_LIBRARY.md      领域错误模式库
├── CHANGELOG.md          需求/设计变更记录 + 同步协议
└── MEMORY_STRUCTURE.md   本文件

.vibecoding/              功能级运行状态（不提交）
├── prd.md                当前功能 PRD
├── design.md             当前功能设计方案
├── acceptance.md         验收场景（Given-When-Then）
├── evolution.md           开发进度日志（追加，永不覆盖）
├── requirements.md        Micro-step 拆解结果
├── state.json             会话状态
└── logs/                  运行日志
```

## AI 启动读取顺序

1. `docs/SPEC.md` → 产品边界（非目标不实现）
2. `.vibecoding/evolution.md`（最后 5 条）→ 进度上下文
3. `.vibecoding/state.json` → 会话状态（中断恢复）
4. `.vibecoding/prd.md` → 当前功能需求
5. `.vibecoding/design.md` → 设计方案

## AI 写入规则

| 文件 | 何时写 | 方式 |
|------|--------|------|
| SPEC.md | 产品方向变更 | 覆盖 |
| DECISIONS.md | 架构选型后 | 追加 |
| ERROR_LIBRARY.md | 修复耗时 >10min 的错误后 | 追加 |
| CHANGELOG.md | PRD/设计变更时 | 追加（含影响分析） |
| prd.md | Phase 0 | 覆盖 |
| design.md | Phase 0 / 设计变更 | 覆盖 |
| acceptance.md | Phase 0.5 / 每 Step 完成 | 追加行 |
| evolution.md | 每个 Phase 完成 / 发生变更 | **追加（永不覆盖）** |

## 遇到错误时的处理顺序

1. 搜索 `docs/ERROR_LIBRARY.md` 匹配已知模式 → 命中则直接用
2. 未命中 → Agent Loop 排查
3. 修复后 → 追加 ERROR_LIBRARY.md
