# VibeCoding Workflow — 初始化指南

## 完整初始化流程（5 分钟）

### 第 1 步：确认 Skill 已安装

Skill 应已全局安装在 `~/.codex/skills/vibe-coding-workflow/`。

```bash
ls ~/.codex/skills/vibe-coding-workflow/SKILL.md
```

如果不存在，从本模板的 `skill-copy/` 复制：

```bash
cp -r skill-copy/ ~/.codex/skills/vibe-coding-workflow/
```

### 第 2 步：将项目规则复制到目标项目

```bash
# 替换 <target> 为你的项目路径
TARGET=/path/to/your-project

# 创建目录
mkdir -p $TARGET/.codex/rules
mkdir -p $TARGET/.cursor/rules
mkdir -p $TARGET/.github
mkdir -p $TARGET/docs

# 复制规则
cp project-init/.codex/rules/* $TARGET/.codex/rules/
cp project-init/.cursor/rules/* $TARGET/.cursor/rules/
cp project-init/VIBECODING-RULES.md $TARGET/
cp project-init/CLAUDE.md $TARGET/
cp project-init/.github/PULL_REQUEST_TEMPLATE.md $TARGET/.github/

# 复制全局参考文档
cp project-init/docs/* $TARGET/docs/

# 创建 .vibecoding/ 和 .gitignore（如果不存在）
mkdir -p $TARGET/.vibecoding/logs
echo ".vibecoding/" >> $TARGET/.gitignore 2>/dev/null
```

### 第 3 步：修改项目上下文（可选）

`project-init/.codex/rules/vibecoding-tdd.md` 中的项目上下文会在第一次 Phase 0 时自动检测并填充。
如果希望手动配置，编辑该文件的「项目上下文」表格。

### 第 4 步：验证安装

**Codex**：在目标项目中输入：
> "用 VibeCoding 工作流开发一个简单的功能"

如果 Codex 回复工作流启动信息，说明安装成功。

**Cursor**：打开目标项目，Cursor 会自动加载 `.cursor/rules/` 下的规则文件。

**Claude Code**：在目标项目中输入：
> "按照 VIBECODING-RULES.md 的工作流，开发一个简单的功能"

Claude Code 会按 CLAUDE.md 入口 → VIBECODING-RULES.md 规则 → Phase 0→6 流程执行。

---

## 各工具使用方式

| 工具 | 初始化方式 | 规则加载机制 |
|------|-----------|------------|
| **Codex** | 安装 Skill + 复制 `.codex/rules/` | Skill 自动编排 + 7 个规则文件 |
| **Cursor** | 复制 `.cursor/rules/` | 7 个 `.mdc` 文件，按 glob 匹配自动加载 |
| **Claude Code** | 复制 `CLAUDE.md` + `VIBECODING-RULES.md` + `docs/` | CLAUDE.md 入口 → @VIBECODING-RULES.md → docs/ 参考文件 |

**核心原则**：`VIBECODING-RULES.md` 是所有工具共享的单一真相来源。工具专属规则文件（`.codex/rules/`、`.cursor/rules/`）只做适配，不重复定义规则。

---

## 常见问题

### Q: Claude Code 和 Codex / Cursor 的工作流有什么区别？

Claude Code 没有 Skill 自动编排和 `.mdc` glob 匹配机制。它的工作方式：
- 启动时读取 `CLAUDE.md`（入口）→ `VIBECODING-RULES.md`（规则）→ 按 Phase 0→6 流程执行
- 用户说「按照 VibeCoding 工作流开发 XXX」即可启动
- 规则全部在 `VIBECODING-RULES.md` 中，不分散到多个工具专属文件

### Q: 我的项目不是 Node.js/TypeScript，可以用吗？

可以。Phase 0 的 TDD 上下文自动同步会检测你的 `pyproject.toml`、`go.mod`、`Cargo.toml` 等配置文件。
如果检测失败，在 `vibecoding-tdd.md` 中手动填写项目上下文即可。

### Q: 我没有 staging 环境，Phase 3.5 的测试怎么办？

Phase 3.5 会自动降级：
- 有 staging → 在 staging 跑 E2E
- 无 staging → 本地 dev server
- dev server 不可启动 → HTTP mock

### Q: 某些测试类型不需要（如性能测试），能跳过吗？

能。Phase 3.5 开头有适用性判断，AI 根据设计文档判断，你确认后跳过不适用的。

### Q: evolution.md 会变得很大吗？

会。200 行后自动归档到 `archive/evolution-YYYY-MM.md`，新文件从头开始。

### Q: 我不小心把 `.vibecoding/` 提交了怎么办？

确保 `.gitignore` 包含 `.vibecoding/`。如果已提交：

```bash
echo ".vibecoding/" >> .gitignore
git rm -r --cached .vibecoding/
git commit -m "chore: remove .vibecoding from version control"
```

### Q: 什么是 Phase 0.8 并行模式？什么时候用？

Phase 0.8 是可选阶段。当 PRD 明显包含多个独立模块（如用户系统+商品系统+订单系统），
AI 会自动评估是否适合并行开发。你确认后启动并行模式，多个 Workstream 同时由 subagent 开发。

AI 评估为"建议串行"或你不选择并行 → 走标准线性流程，不受影响。
