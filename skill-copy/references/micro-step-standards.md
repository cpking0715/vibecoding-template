# Micro-step 拆解规范

## 为什么需要这个规范

TDD 要求"小步快跑"。但"多小算小"没有标准，AI 容易一脚踩太大或碎步太多。
本文档定义具体的拆解原则和可操作的标准。

## 拆解五原则

### 原则一：一个 step = 一个 Given-When-Then

每个 Micro-step 必须能用一句 Given-When-Then 完整描述行为。做不到说明拆得不够细。

```
✅ GIVEN 用户输入正确的邮箱和密码
   WHEN 调用 login(email, password)
   THEN 返回 { success: true }

✅ GIVEN 用户输入错误的密码
   WHEN 调用 login(email, wrong_password)
   THEN 返回 { success: false, error: "invalid_credentials" }
```

### 原则二：最多 3 个 test case

如果一个 step 需要超过 3 个测试用例才能覆盖，说明这个 step 应该继续拆分。

- Step 1：正常路径（1-2 个 test）
- Step 2：边界条件（1-2 个 test）
- Step 3：业务规则（1-2 个 test）

### 原则三："不可再分"测试

如果一个 step 的测试可以进一步拆为更小的断言，继续拆。

```
❌ test("should validate email and check password and return token")
   — 3 个行为混在一起
✅ test("should return error for invalid email format")
✅ test("should return error for wrong password")
✅ test("should return token on success")
   — 每个测试只验证一个行为
```

### 原则四：一个 step 最多改 2 个文件

跨多个文件的变更必须拆成多个 step。常见拆分：

| Step | 改的文件 | 做什么 |
|------|---------|-------|
| 1 | lib/auth.ts + types/auth.ts | 定义接口签名和类型 |
| 2 | lib/auth.ts | 实现核心逻辑 |
| 3 | app/api/auth/route.ts | 挂载到 API 路由 |

### 原则五：先骨架后肉

第一个 step 永远只定义接口签名和类型，不填实现。后续 step 逐步注入逻辑。

```
Step 1: export function login(email: string, password: string): Promise<LoginResult>
Step 2: 实现密码校验
Step 3: 实现邮箱格式校验
Step 4: 实现登录锁定
Step 5: 实现积分逻辑
```

## 拆解检查清单

提交每个 Micro-step 前自查：

- [ ] 这个 step 能否用一句 Given-When-Then 描述？
- [ ] 测试用例数量 ≤ 3？
- [ ] 改动涉及的文件 ≤ 2？
- [ ] 如果以上都是"是"但仍然感觉太大 → 继续拆

## 拆解示例对比

### 需求：采购入库单 OCR 识别

❌ **太粗（AI 默认拆法）**

```
Step 1: 实现整个 OCR 识别 → 解析 → 存入数据库
  → 测试: 10+ cases, 文件: 5+, 一次改完不可控
```

✅ **正确 Micro-step 拆法**

```
Step 1: 定义 OcrResult 类型 + parseOcrText(text: string) 签名
       测试: 类型定义正确
Step 2: 实现解析供应商名称
       测试: "供应商: 某某公司" → { supplier: "某某公司" }
Step 3: 实现解析商品清单
       测试: 单行商品、多行商品、空行跳过
Step 4: 实现解析金额合计
       测试: 正常金额、含逗号金额、无金额
Step 5: 将解析结果转为入库单数据格式
       测试: OcrResult → PurchaseOrderInput
```
