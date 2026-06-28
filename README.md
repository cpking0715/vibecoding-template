# VibeCoding Workflow — 通用开发底座

将 AI 驱动的 TDD 开发流程标准化、工程化。一个 Skill + 一套项目规则，覆盖从需求到部署的全链路。

## 一句话说明

> 以后每个新项目，复制 `project-init/` 到项目根目录、改几行配置、说一句"用 VibeCoding 工作流"，AI 自动完成：PRD → 设计 → 拆解 → TDD → 验收测试 → 安全扫描 → 文档 → 合入。

## 目录结构

```
vibecoding-template/
├── skill-copy/                        ← 全局 Skill 副本（供参考/离线使用）
│   ├── SKILL.md                       8 个 Phase 的完整工作流定义
│   ├── agents/openai.yaml             UI 元数据
│   └── references/
│       ├── model-tiering.md           模型分层策略（T1 Haiku / T2 Sonnet / T3 Opus）
│       ├── requirements-template.md   需求输入模板
│       ├── micro-step-standards.md    Micro-step 拆解五原则
│       ├── agent-loop.md              测试失败修复循环
│       ├── testing-spec.md            综合验收测试规范（6类）
│       ├── memory-structure.md        开发记忆与文件结构
│       └── pr-template.md             PR 模板
│
├── project-init/                      ← 新项目初始化文件（复制到项目）
│   ├── .codex/rules/                  Codex 项目规则（7 个）
│   ├── .cursor/rules/                 Cursor 规则（7 个）
│   ├── VIBECODING-RULES.md            Claude Code 规则
│   ├── CLAUDE.md                      Claude Code 入口
│   └── .github/PULL_REQUEST_TEMPLATE.md  PR 模板
│
├── INIT-GUIDE.md                      初始化指南
├── .gitignore
└── README.md
```

## 使用方式

### 新项目

```bash
cp -r vibecoding-template/project-init/ <你的新项目>/
cd <你的新项目>
# 打开 Codex / Cursor / Claude Code
# 说："开始 VibeCoding 工作流，我要开发..."
```

### 已有项目

```bash
cp -r vibecoding-template/project-init/.codex/ <已有项目>/.codex/
cp vibecoding-template/project-init/VIBECODING-RULES.md <已有项目>/
cp vibecoding-template/project-init/CLAUDE.md <已有项目>/
cp -r vibecoding-template/project-init/.cursor/ <已有项目>/.cursor/  # 如果用 Cursor
cp -r vibecoding-template/project-init/docs/ <已有项目>/docs/        # 全局参考文档
```

然后修改 `<已有项目>/.codex/rules/vibecoding-tdd.md` 中的项目上下文（Phase 0 会自动检测同步，也可手动改）。

> **VIBECODING-RULES.md 是所有工具的单一真相来源**：Codex rules 和 Cursor rules 只做工具适配，不重复定义规则。Claude Code 通过 CLAUDE.md → @VIBECODING-RULES.md 加载全部规则。

### Skill 安装（仅 Codex 需要）

Skill 已全局安装在 `~/.codex/skills/vibe-coding-workflow/`。如需手动安装：

```bash
cp -r vibecoding-template/skill-copy/ ~/.codex/skills/vibe-coding-workflow/
```

Cursor 和 Claude Code 不需要安装 Skill — 它们通过 `.cursor/rules/` 和 `CLAUDE.md` 直接加载规则。

## 快速参考

| 阶段 | 做什么 | 产物 |
|------|--------|------|
| Phase 0 | PRD + 设计文档 | .vibecoding/prd.md, design.md |
| Phase 0.5 | 验收场景列表 | .vibecoding/acceptance.md |
| Phase 0.8 | 并行工作流评估（可选） | .vibecoding/task-graph.md |
| Phase 1 | 拆解 Micro-step → 创建分支 | requirements.md, feature/xxx 分支 |
| Phase 2 | TDD Red-Green-Refactor（每 Step commit） | 测试 + 实现代码 |
| Phase 3 | Agent Loop（修复失败测试） | logs/failure-N.md |
| Phase 3.5 | 6 类验收测试 | 测试报告 |
| Phase 4 | 安全 PR 门禁 | PR 标签 |
| Phase 5 | 文档生成 | docs/api/*.md |
| Phase 6 | push → PR → merge → tag → 清理 | PR merged, vX.Y.Z |

## 核心原则

- **TDD 驱动**：Red-Green-Refactor，测试不通过不写业务代码
- **小步提交**：每个 Micro-step 独立 commit，可精确回滚
- **模型分层**：测试/文档用 T1（省钱），常规代码用 T2，核心逻辑用 T3（兜底）
- **全链路验收**：功能/安全/性能/异常/兼容/数据库，六道关卡
- **记忆持久化**：evolution.md 追加式写入，AI 跨会话恢复上下文
