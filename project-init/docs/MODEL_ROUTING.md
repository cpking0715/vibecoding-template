# 手动模型切换指南

> 适用于 **Cursor / Claude Code** 等不支持自动 Subagent 的 AI 工具。
> 完整指南见 Skill 内的 `references/manual-model-routing.md`。

## 快速切换卡片

```
═══════════════════════════════════════════════
  阶段           切到        切回
───────────────────────────────────────────────
  Phase 0b 架构   → Opus      完事切回 Sonnet
  Phase 2b 写测试 → Haiku     进 2c 前切回
  Phase 2c 核心   → Opus      完事切回
  Phase 2e lint   → Haiku     修完切回
  Phase 3 第3次   → Opus      修完切回
  Phase 4 敏感    → Opus      审完切回
  Phase 5 文档    → Haiku     进 6 前切回
═══════════════════════════════════════════════

  只用 Sonnet ≈ $20/月
  按表切换    ≈ $12-15/月
```

## Cursor

| 操作 | 方式 |
|------|------|
| 切换模型 | `Cmd+Shift+P` → "Switch Model" |
| 查看当前 | 状态栏右下角 |

切换后对话上下文不丢失。

## Claude Code

```bash
export ANTHROPIC_MODEL=claude-3-5-haiku-latest  # Haiku
export ANTHROPIC_MODEL=claude-3-opus-latest     # Opus
# 默认 = Sonnet，不需要设置
```

## 不想手动切？

全程用 Sonnet。每月多花 ~$5，省掉手动切换的麻烦。质量对 90% 任务已经足够。
