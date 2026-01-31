---
name: coding-standards
description: Universal coding standards, best practices, and patterns for TypeScript, JavaScript, React, and Node.js development.
---

# 程序代码标准与最佳实务

适用於所有专案的通用程序代码标准。

## 程序代码品质原则

### 1. 可读性优先
- 程序代码被阅读的次数远多於被撰寫的次数
- 使用清晰的变量和函数名称
- 优先使用自文件化的程序代码而非注释
- 保持一致的格式化

### 2. KISS（保持简单）
- 使用最简单的解决方案
- 避免过度工程
- 不做过早优化
- 易於理解 > 聰明的程序代码

### 3. DRY（不重复自己）
- 将共用逻辑提取为函数
- 建立可重用的元件
- 在模块间共享工具函数
- 避免复制粘贴程序设计

### 4. YAGNI（你不会需要它）
- 在需要之前不要构建功能
- 避免推测性的通用化
- 只在需要时增加复杂度
- 从简单开始，需要时再重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// ✅ 良好：描述性名称
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 不良：不清楚的名称
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// ✅ 良好：动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 不良：不清楚或只有名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变性模式（关键）

```typescript
// ✅ 总是使用展开运算符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 永远不要直接修改
user.name = 'New Name'  // 不良
items.push(newItem)     // 不良
```

### 错误处理

```typescript
// ✅ 良好：完整的错误处理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ 不良：无错误处理
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 最佳实务

```typescript
// ✅ 良好：可能时并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 不良：不必要的顺序执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 型別安全

```typescript
// ✅ 良好：正确的型別
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 实作
}

// ❌ 不良：使用 'any'
function getMarket(id: any): Promise<any> {
  // 实作
}
```

## React 最佳实务

### 元件结构

```typescript
// ✅ 良好：具有型別的函数元件
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ 不良：无型別、结构不清楚
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 自定义 Hooks

```typescript
// ✅ 良好：可重用的自定义 hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用方式
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 状态管理

```typescript
// ✅ 良好：正确的状态更新
const [count, setCount] = useState(0)

// 基於先前状态的函数更新
setCount(prev => prev + 1)

// ❌ 不良：直接引用状态
setCount(count + 1)  // 在非同步情境中可能过时
```

### 条件渲染

```typescript
// ✅ 良好：清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 不良：三元地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 慣例

```
GET    /api/markets              # 列出所有市場
GET    /api/markets/:id          # 取得特定市場
POST   /api/markets              # 建立新市場
PUT    /api/markets/:id          # 更新市場（完整）
PATCH  /api/markets/:id          # 更新市場（部分）
DELETE /api/markets/:id          # 删除市場

# 过滤用查询參数
GET /api/markets?status=active&limit=10&offset=0
```

### 回应格式

```typescript
// ✅ 良好：一致的回应结构
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功回应
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 错误回应
return NextResponse.json({
  success: false,
  error: 'Invalid request'
}, { status: 400 })
```

### 输入验证

```typescript
import { z } from 'zod'

// ✅ 良好：Schema 验证
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 使用验证过的资料继续处理
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## 档案组织

### 专案结构

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 路由
│   ├── markets/           # 市場页面
│   └── (auth)/           # 认证页面（路由群组）
├── components/            # React 元件
│   ├── ui/               # 通用 UI 元件
│   ├── forms/            # 表单元件
│   └── layouts/          # 版面配置元件
├── hooks/                # 自定义 React hooks
├── lib/                  # 工具和设置
│   ├── api/             # API 客戶端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常数
├── types/                # TypeScript 型別
└── styles/              # 全域样式
```

### 档案命名

```
components/Button.tsx          # 元件用 PascalCase
hooks/useAuth.ts              # hooks 用 camelCase 加 'use' 前缀
lib/formatDate.ts             # 工具用 camelCase
types/market.types.ts         # 型別用 camelCase 加 .types 后缀
```

## 注释与文件

### 何时注释

```typescript
// ✅ 良好：解释「为什麼」而非「什麼」
// 使用指数退避以避免在服务中斷时压垮 API
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 为了处理大阵列的效能，此处刻意使用突变
items.push(newItem)

// ❌ 不良：陈述显而易见的事实
// 将计数器加 1
count++

// 将名称设为使用者的名称
name = user.name
```

### 公开 API 的 JSDoc

```typescript
/**
 * 使用语意相似度搜索市場。
 *
 * @param query - 自然语言搜索查询
 * @param limit - 最大结果数量（预设：10）
 * @returns 按相似度分数排序的市場阵列
 * @throws {Error} 如果 OpenAI API 失敗或 Redis 不可用
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 实作
}
```

## 效能最佳实务

### 记忆化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 良好：记忆化昂贵的计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 良好：记忆化回呼函数
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 延迟载入

```typescript
import { lazy, Suspense } from 'react'

// ✅ 良好：延迟载入重型元件
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### 资料库查询

```typescript
// ✅ 良好：只选择需要的字段
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ 不良：选择所有字段
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 测试标准

### 测试结构（AAA 模式）

```typescript
test('calculates similarity correctly', () => {
  // Arrange（准备）
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act（执行）
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert（断言）
  expect(similarity).toBe(0)
})
```

### 测试命名

```typescript
// ✅ 良好：描述性测试名称
test('returns empty array when no markets match query', () => { })
test('throws error when OpenAI API key is missing', () => { })
test('falls back to substring search when Redis unavailable', () => { })

// ❌ 不良：模糊的测试名称
test('works', () => { })
test('test search', () => { })
```

## 程序代码异味侦测

注意这些反模式：

### 1. 过长函数
```typescript
// ❌ 不良：函数超过 50 行
function processMarketData() {
  // 100 行程序代码
}

// ✅ 良好：拆分为较小的函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 过深巢状
```typescript
// ❌ 不良：5 层以上巢状
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 做某事
        }
      }
    }
  }
}

// ✅ 良好：提前返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 做某事
```

### 3. 魔术数字
```typescript
// ❌ 不良：无解释的数字
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 良好：命名常数
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**记住**：程序代码品质是不可协商的。清晰、可维护的程序代码能实现快速开发和自信的重构。
