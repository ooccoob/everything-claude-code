---
name: security-review
description: Use this skill when adding authentication, handling user input, working with secrets, creating API endpoints, or implementing payment/sensitive features. Provides comprehensive security checklist and patterns.
---

# 安全性审查技能

此技能确保所有程序代码遵循安全性最佳实务并识别潜在漏洞。

## 何时启用

- 实作认证或授权
- 处理使用者输入或档案上傳
- 建立新的 API 端点
- 处理密钥或憑证
- 实作支付功能
- 保存或传输敏感资料
- 整合第三方 API

## 安全性检查清单

### 1. 密钥管理

#### ❌ 绝不这样做
```typescript
const apiKey = "sk-proj-xxxxx"  // 寫死的密钥
const dbPassword = "password123" // 在原始码中
```

#### ✅ 总是这样做
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 验证密钥存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 验证步骤
- [ ] 无寫死的 API 金钥、Token 或密码
- [ ] 所有密钥在环境变量中
- [ ] `.env.local` 在 .gitignore 中
- [ ] git 历史中无密钥
- [ ] 生产密钥在托管平台（Vercel、Railway）中

### 2. 输入验证

#### 总是验证使用者输入
```typescript
import { z } from 'zod'

// 定义验证 schema
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 处理前验证
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 档案上傳验证
```typescript
function validateFileUpload(file: File) {
  // 大小检查（最大 5MB）
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // 类型检查
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 副档名检查
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

#### 验证步骤
- [ ] 所有使用者输入以 schema 验证
- [ ] 档案上傳受限（大小、类型、副档名）
- [ ] 查询中不直接使用使用者输入
- [ ] 白名单验证（非黑名单）
- [ ] 错误讯息不泄露敏感资讯

### 3. SQL 注入预防

#### ❌ 绝不串接 SQL
```typescript
// 危险 - SQL 注入漏洞
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 总是使用參数化查询
```typescript
// 安全 - 參数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 或使用原始 SQL
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 验证步骤
- [ ] 所有资料库查询使用參数化查询
- [ ] SQL 中无字串串接
- [ ] ORM/查询建构器正确使用
- [ ] Supabase 查询正确净化

### 4. 认证与授权

#### JWT Token 处理
```typescript
// ❌ 错误：localStorage（易受 XSS 攻击）
localStorage.setItem('token', token)

// ✅ 正确：httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 授权检查
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 总是先验证授权
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 继续删除
  await db.users.delete({ where: { id: userId } })
}
```

#### Row Level Security（Supabase）
```sql
-- 在所有表格上启用 RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 使用者只能查看自己的资料
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 使用者只能更新自己的资料
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 验证步骤
- [ ] Token 保存在 httpOnly cookies（非 localStorage）
- [ ] 敏感操作前有授权检查
- [ ] Supabase 已启用 Row Level Security
- [ ] 已实作基於角色的存取控制
- [ ] 工作阶段管理安全

### 5. XSS 预防

#### 净化 HTML
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 总是净化使用者提供的 HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### Content Security Policy
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 验证步骤
- [ ] 使用者提供的 HTML 已净化
- [ ] CSP headers 已设置
- [ ] 无未验证的动态内容渲染
- [ ] 使用 React 内建 XSS 保护

### 6. CSRF 保护

#### CSRF Tokens
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // 处理请求
}
```

#### SameSite Cookies
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 验证步骤
- [ ] 状态变更操作有 CSRF tokens
- [ ] 所有 cookies 设置 SameSite=Strict
- [ ] 已实作 Double-submit cookie 模式

### 7. 速率限制

#### API 速率限制
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每窗口 100 个请求
  message: 'Too many requests'
})

// 套用到路由
app.use('/api/', limiter)
```

#### 昂贵操作
```typescript
// 搜索的积極速率限制
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 分钟
  max: 10, // 每分钟 10 个请求
  message: 'Too many search requests'
})

app.use('/api/search', searchLimiter)
```

#### 验证步骤
- [ ] 所有 API 端点有速率限制
- [ ] 昂贵操作有更嚴格限制
- [ ] 基於 IP 的速率限制
- [ ] 基於使用者的速率限制（已认证）

### 8. 敏感资料暴露

#### 日志记录
```typescript
// ❌ 错误：记录敏感资料
console.log('User login:', { email, password })
console.log('Payment:', { cardNumber, cvv })

// ✅ 正确：遮蔽敏感资料
console.log('User login:', { email, userId })
console.log('Payment:', { last4: card.last4, userId })
```

#### 错误讯息
```typescript
// ❌ 错误：暴露内部细节
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 正确：通用错误讯息
catch (error) {
  console.error('Internal error:', error)
  return NextResponse.json(
    { error: 'An error occurred. Please try again.' },
    { status: 500 }
  )
}
```

#### 验证步骤
- [ ] 日志中无密码、token 或密钥
- [ ] 使用者收到通用错误讯息
- [ ] 详细错误只在伺服器日志
- [ ] 不向使用者暴露堆叠追踪

### 9. 区块链安全（Solana）

#### 钱包验证
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### 交易验证
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 验证收款人
  if (transaction.to !== expectedRecipient) {
    throw new Error('Invalid recipient')
  }

  // 验证金额
  if (transaction.amount > maxAmount) {
    throw new Error('Amount exceeds limit')
  }

  // 验证使用者有足夠余额
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('Insufficient balance')
  }

  return true
}
```

#### 验证步骤
- [ ] 钱包签章已验证
- [ ] 交易详情已验证
- [ ] 交易前有余额检查
- [ ] 无盲目交易签署

### 10. 依赖安全

#### 定期更新
```bash
# 检查漏洞
npm audit

# 自动修复可修复的问题
npm audit fix

# 更新依赖
npm update

# 检查过时套件
npm outdated
```

#### Lock 档案
```bash
# 总是 commit lock 档案
git add package-lock.json

# 在 CI/CD 中使用以获得可重现的构建
npm ci  # 而非 npm install
```

#### 验证步骤
- [ ] 依赖保持最新
- [ ] 无已知漏洞（npm audit 乾净）
- [ ] Lock 档案已 commit
- [ ] GitHub 上已启用 Dependabot
- [ ] 定期安全更新

## 安全测试

### 自动化安全测试
```typescript
// 测试认证
test('requires authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 测试授权
test('requires admin role', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 测试输入验证
test('rejects invalid input', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// 测试速率限制
test('enforces rate limits', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## 部署前安全检查清单

任何生产部署前：

- [ ] **密钥**：无寫死密钥，全在环境变量中
- [ ] **输入验证**：所有使用者输入已验证
- [ ] **SQL 注入**：所有查询已參数化
- [ ] **XSS**：使用者内容已净化
- [ ] **CSRF**：保护已启用
- [ ] **认证**：正确的 token 处理
- [ ] **授权**：角色检查已就位
- [ ] **速率限制**：所有端点已启用
- [ ] **HTTPS**：生产环境強制使用
- [ ] **安全标头**：CSP、X-Frame-Options 已设置
- [ ] **错误处理**：错误中无敏感资料
- [ ] **日志记录**：无敏感资料被记录
- [ ] **依赖**：最新，无漏洞
- [ ] **Row Level Security**：Supabase 已启用
- [ ] **CORS**：正确设置
- [ ] **档案上傳**：已验证（大小、类型）
- [ ] **钱包签章**：已验证（如果是区块链）

## 资源

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**记住**：安全性不是可选的。一个漏洞可能危及整个平台。有疑虑时，选择谨慎的做法。
