# Agent Loop 规格

## 触发条件

当以下阶段运行测试失败时进入本循环：

| 触发阶段 | 条件 | 说明 |
|---------|------|------|
| Phase 2 Red | 测试代码编译/import 错误 | 修复测试本身 |
| Phase 2 Green | 实现未通过测试 | 修复实现 |
| Phase 2 Refactor | 重构引入回归 | 修复回归 |
| Phase 3.5 功能测试 | E2E 场景失败 | 修复实现 |
| Phase 3.5 性能测试 | API 响应时间或错误率不达标 | 优化性能 |
| Phase 3.5 安全测试 | 依赖漏洞或代码安全问题 | 修复安全 |
| Phase 3.5 异常测试 | 容错逻辑未通过 | 修复容错 |

⚠️ **Red 阶段的预期失败（assert 正确失败）不进 Loop。**

## 循环结构（修复模式）

```
[分析失败] ──→ [定位问题] ──→ [修复] ──→ [重新运行测试]
    ↑                                         │
    │                                    全部通过？── 是 ──→ 回到触发时的 Phase 步骤
    │                                         │
    └──────────────────────────────────── 否 ──┘
                                              │
                                         通知用户当前状态
```

## 失败分析方法

1. **搜索已知模式**：在 `docs/ERROR_LIBRARY.md` 中搜索错误特征（错误消息关键词 / stack trace 函数名），命中 → 直接用已知修复方案，跳过后续步骤
2. **读错误类型**：编译 / import / 断言 / 超时 / E2E / 性能 / 安全
3. **读 stack trace** / 测试报告：定位文件和行号
4. **对比期望值与实际值**
5. **判断根因**：测试本身 / 实现遗漏 / 回归 / 性能瓶颈 / 安全漏洞
6. **输出修复策略**到 `.vibecoding/logs/` 后再改代码
7. **修复成功后**：如果本次错误不在 ERROR_LIBRARY.md 中，追加一条新记录

## 超时处理

- 测试设置 `--testTimeout 30000`
- 挂起超过 30 秒 → 超时失败
- 超时原因分析：死循环 / 未 mock 的外部调用 / 未关闭的数据库连接

## Flaky Test 处理

如果同一测试在连续 2 次修复后仍偶发失败（有时过有时不过）：
1. 标记为 flaky test
2. 记入 `.vibecoding/logs/flaky-tests.md`
3. **不阻塞合入**
4. 创建 issue 跟踪

## 重试策略与用户通知

| 尝试次数 | 使用模型 | 实现方式 | 用户通知 |
|---------|---------|---------|
| 第1次失败 | T2（Sonnet） | Subagent（T2 Sonnet） | 通知用户"正在修复第1次" |
| 第2次失败 | T2（Sonnet，换思路） | Subagent（T2 Sonnet） | 通知用户"换思路重试第2次" |
| 第3次失败 | T3（Opus） | Subagent（T3 Opus） | 通知用户"升级到 Opus 第3次" |
| 第4次失败 | — | — | 人工介入，生成 `.vibecoding/report.md` |

每次失败后通知用户：当前尝试次数、根因猜测、下一步策略。

## 修复成功后的退出路径

| 触发阶段 | 回到位置 |
|---------|---------|
| Phase 2 Red（测试代码错） | Step 2b 重新确认预期失败 |
| Phase 2 Green（实现未通过） | Step 2c 重新运行测试 |
| Phase 2 Refactor（回归） | Step 2d 重新确认重构后测试通过 |
| Phase 3.5 功能测试 | Phase 3.5 Step 3.5c 重跑 |
| Phase 3.5 性能测试 | Phase 3.5 Step 3.5b 重跑 |
| Phase 3.5 安全测试 | Phase 3.5 Step 3.5e 重跑 |
| Phase 3.5 异常测试 | Phase 3.5 Step 3.5d 重跑 |

## 状态文件

每次循环开始前检查 `.vibecoding/state.json`，循环结束后更新。

```json
{
  "feature": "login",
  "trigger_phase": "green|phase3.5_e2e|phase3.5_perf",
  "step": "tdd-step-2c|phase3.5-step-c",
  "status": "in_progress|passed|blocked",
  "history": [
    {
      "attempt": 1,
      "trigger": "green",
      "model": "sonnet",
      "error_type": "AssertionError",
      "test_results": {"passed": 2, "failed": 1},
      "fix": "added password comparison",
      "duration_ms": 8400,
      "user_notified": true
    }
  ]
}
```

## 失败记录模板

每次失败记录到 `.vibecoding/logs/failure-<N>.md`：

```
## 失败 #N
- 触发阶段: Red / Green / Refactor / Phase3.5_func / Phase3.5_perf / Phase3.5_sec
- 时间: YYYY-MM-DD HH:mm
- 测试: tests/test_login.py::test_login_wrong_password
- 错误类型: AssertionError / ImportError / Timeout / E2E / PerfFail / SecVuln
- 期望值: "error"
- 实际值: "success"
- 根因: 没有对比密码
- 修复: 添加 password === stored.password 判断
- 修复模型: sonnet
- 用户已通知: yes
```

## 退出条件

1. **全部通过** → 回到触发时步骤
2. **达到重试上限（4次）** → 生成 `.vibecoding/report.md`
3. **用户主动中断** → 标记状态，下次继续
