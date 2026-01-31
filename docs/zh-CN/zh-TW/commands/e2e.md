---
description: Generate and run end-to-end tests with Playwright. Creates test journeys, runs tests, captures screenshots/videos/traces, and uploads artifacts.
---

# E2E 指令

此指令呼叫 **e2e-runner** Agent 来产生、维护和执行使用 Playwright 的端对端测试。

## 此指令的功能

1. **产生测试旅程** - 为使用者流程建立 Playwright 测试
2. **执行 E2E 测试** - 跨浏览器执行测试
3. **撷取产出物** - 失敗时的截圖、影片、追踪
4. **上傳结果** - HTML 報告和 JUnit XML
5. **识别不稳定测试** - 隔离不稳定的测试

## 何时使用

在以下情况使用 `/e2e`：
- 测试关键使用者旅程（登入、交易、支付）
- 验证多步骤流程端对端运作
- 测试 UI 互动和导航
- 验证前端和后端的整合
- 为生产环境部署做准备

## 运作方式

e2e-runner Agent 会：

1. **分析使用者流程**并识别测试情境
2. **产生 Playwright 测试**使用 Page Object Model 模式
3. **跨多个浏览器执行测试**（Chrome、Firefox、Safari）
4. **撷取失敗**的截圖、影片和追踪
5. **产生報告**包含结果和产出物
6. **识别不稳定测试**并建议修复

## 测试产出物

测试执行时，会撷取以下产出物：

**所有测试：**
- HTML 報告包含时间线和结果
- JUnit XML 用於 CI 整合

**仅在失敗时：**
- 失敗状态的截圖
- 测试的影片录制
- 追踪档案用於调试（逐步重播）
- 网路日志
- Console 日志

## 检视产出物

```bash
# 在浏览器检视 HTML 報告
npx playwright show-report

# 检视特定追踪档案
npx playwright show-trace artifacts/trace-abc123.zip

# 截圖保存在 artifacts/ 目录
open artifacts/search-results.png
```

## 最佳实务

**应该做：**
- ✅ 使用 Page Object Model 以利维护
- ✅ 使用 data-testid 属性作为选择器
- ✅ 等待 API 回应，不要用任意逾时
- ✅ 测试关键使用者旅程端对端
- ✅ 合并到主分支前执行测试
- ✅ 测试失敗时审查产出物

**不应该做：**
- ❌ 使用脆弱的选择器（CSS class 可能改变）
- ❌ 测试实作细节
- ❌ 对生产环境执行测试
- ❌ 忽略不稳定的测试
- ❌ 失敗时跳过产出物审查
- ❌ 用 E2E 测试每个边界情况（使用单元测试）

## 快速指令

```bash
# 执行所有 E2E 测试
npx playwright test

# 执行特定测试档案
npx playwright test tests/e2e/markets/search.spec.ts

# 以可视模式执行（看到浏览器）
npx playwright test --headed

# 调试测试
npx playwright test --debug

# 产生测试程序代码
npx playwright codegen http://localhost:3000

# 检视報告
npx playwright show-report
```

## 与其他指令的整合

- 使用 `/plan` 识别要测试的关键旅程
- 使用 `/tdd` 进行单元测试（更快、更细粒度）
- 使用 `/e2e` 进行整合和使用者旅程测试
- 使用 `/code-review` 验证测试品质

## 相关 Agent

此指令呼叫位於以下位置的 `e2e-runner` Agent：
`~/.claude/agents/e2e-runner.md`
