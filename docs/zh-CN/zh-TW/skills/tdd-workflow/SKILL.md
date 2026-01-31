---
name: tdd-workflow
description: Use this skill when writing new features, fixing bugs, or refactoring code. Enforces test-driven development with 80%+ coverage including unit, integration, and E2E tests.
---

# 测试驱动开发工作流程

此技能确保所有程序代码开发遵循 TDD 原则，并具有完整的测试覆盖率。

## 何时启用

- 撰寫新功能或功能性程序代码
- 修复 Bug 或问题
- 重构现有程序代码
- 新增 API 端点
- 建立新元件

## 核心原则

### 1. 测试先於程序代码
总是先寫测试，然后实作程序代码使测试通过。

### 2. 覆盖率要求
- 最低 80% 覆盖率（单元 + 整合 + E2E）
- 涵盖所有边界案例
- 测试错误情境
- 验证边界条件

### 3. 测试类型

#### 单元测试
- 个別函数和工具
- 元件逻辑
- 纯函数
- 辅助函数和工具

#### 整合测试
- API 端点
- 资料库操作
- 服务互动
- 外部 API 呼叫

#### E2E 测试（Playwright）
- 关键使用者流程
- 完整工作流程
- 浏览器自动化
- UI 互动

## TDD 工作流程步骤

### 步骤 1：撰寫使用者旅程
```
身为 [角色]，我想要 [动作]，以便 [好处]

范例：
身为使用者，我想要语意搜索市場，
以便即使没有精确关键字也能找到相关市場。
```

### 步骤 2：产生测试案例
为每个使用者旅程建立完整的测试案例：

```typescript
describe('Semantic Search', () => {
  it('returns relevant markets for query', async () => {
    // 测试实作
  })

  it('handles empty query gracefully', async () => {
    // 测试边界案例
  })

  it('falls back to substring search when Redis unavailable', async () => {
    // 测试回退行为
  })

  it('sorts results by similarity score', async () => {
    // 测试排序逻辑
  })
})
```

### 步骤 3：执行测试（应该失敗）
```bash
npm test
# 测试应该失敗 - 我们还没实作
```

### 步骤 4：实作程序代码
撰寫最少的程序代码使测试通过：

```typescript
// 由测试引导的实作
export async function searchMarkets(query: string) {
  // 实作在此
}
```

### 步骤 5：再次执行测试
```bash
npm test
# 测试现在应该通过
```

### 步骤 6：重构
在保持测试通过的同时改善程序代码品质：
- 移除重复
- 改善命名
- 优化效能
- 增強可读性

### 步骤 7：验证覆盖率
```bash
npm run test:coverage
# 验证达到 80%+ 覆盖率
```

## 测试模式

### 单元测试模式（Jest/Vitest）
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button Component', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API 整合测试模式
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('returns markets successfully', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('validates query parameters', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('handles database errors gracefully', async () => {
    // Mock 资料库失敗
    const request = new NextRequest('http://localhost/api/markets')
    // 测试错误处理
  })
})
```

### E2E 测试模式（Playwright）
```typescript
import { test, expect } from '@playwright/test'

test('user can search and filter markets', async ({ page }) => {
  // 导航到市場页面
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 验证页面载入
  await expect(page.locator('h1')).toContainText('Markets')

  // 搜索市場
  await page.fill('input[placeholder="Search markets"]', 'election')

  // 等待 debounce 和结果
  await page.waitForTimeout(600)

  // 验证搜索结果显示
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 验证结果包含搜索词
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // 依状态筛选
  await page.click('button:has-text("Active")')

  // 验证筛选结果
  await expect(results).toHaveCount(3)
})

test('user can create a new market', async ({ page }) => {
  // 先登入
  await page.goto('/creator-dashboard')

  // 填寫市場建立表单
  await page.fill('input[name="name"]', 'Test Market')
  await page.fill('textarea[name="description"]', 'Test description')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // 提交表单
  await page.click('button[type="submit"]')

  // 验证成功讯息
  await expect(page.locator('text=Market created successfully')).toBeVisible()

  // 验证重导向到市場页面
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 测试档案组织

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # 单元测试
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 整合测试
└── e2e/
    ├── markets.spec.ts               # E2E 测试
    ├── trading.spec.ts
    └── auth.spec.ts
```

## Mock 外部服务

### Supabase Mock
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'Test Market' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis Mock
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI Mock
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // Mock 1536 维嵌入向量
  ))
}))
```

## 测试覆盖率验证

### 执行覆盖率報告
```bash
npm run test:coverage
```

### 覆盖率门槛
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 常见测试错误避免

### ❌ 错误：测试实作细节
```typescript
// 不要测试内部状态
expect(component.state.count).toBe(5)
```

### ✅ 正确：测试使用者可见行为
```typescript
// 测试使用者看到的内容
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ 错误：脆弱的选择器
```typescript
// 容易坏掉
await page.click('.css-class-xyz')
```

### ✅ 正确：语意选择器
```typescript
// 对变更有彈性
await page.click('button:has-text("Submit")')
await page.click('[data-testid="submit-button"]')
```

### ❌ 错误：无测试隔离
```typescript
// 测试互相依赖
test('creates user', () => { /* ... */ })
test('updates same user', () => { /* 依赖前一个测试 */ })
```

### ✅ 正确：独立测试
```typescript
// 每个测试设置自己的资料
test('creates user', () => {
  const user = createTestUser()
  // 测试逻辑
})

test('updates user', () => {
  const user = createTestUser()
  // 更新逻辑
})
```

## 持续测试

### 开发期间的 Watch 模式
```bash
npm test -- --watch
# 档案变更时自动执行测试
```

### Pre-Commit Hook
```bash
# 每次 commit 前执行
npm test && npm run lint
```

### CI/CD 整合
```yaml
# GitHub Actions
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## 最佳实务

1. **先寫测试** - 总是 TDD
2. **一个测试一个断言** - 专注单一行为
3. **描述性测试名称** - 解释测试内容
4. **Arrange-Act-Assert** - 清晰的测试结构
5. **Mock 外部依赖** - 隔离单元测试
6. **测试边界案例** - Null、undefined、空值、大值
7. **测试错误路徑** - 不只是快樂路徑
8. **保持测试快速** - 单元测试每个 < 50ms
9. **测试后清理** - 无副作用
10. **检视覆盖率報告** - 识别缺口

## 成功指标

- 达到 80%+ 程序代码覆盖率
- 所有测试通过（绿色）
- 无跳过或停用的测试
- 快速测试执行（单元测试 < 30s）
- E2E 测试涵盖关键使用者流程
- 测试在生产前捕捉 Bug

---

**记住**：测试不是可选的。它们是实现自信重构、快速开发和生产可靠性的安全网。
