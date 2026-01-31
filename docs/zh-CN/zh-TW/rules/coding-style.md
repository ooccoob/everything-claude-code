# 程序代码风格

## 不可变性（关键）

总是建立新对象，绝不变异：

```javascript
// 错误：变异
function updateUser(user, name) {
  user.name = name  // 变异！
  return user
}

// 正确：不可变性
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## 档案组织

多小档案 > 少大档案：
- 高内聚、低耦合
- 通常 200-400 行，最多 800 行
- 从大型元件中抽取工具
- 依功能/领域组织，而非依类型

## 错误处理

总是全面处理错误：

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('Detailed user-friendly message')
}
```

## 输入验证

总是验证使用者输入：

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## 程序代码品质检查清单

在标记工作完成前：
- [ ] 程序代码可读且命名良好
- [ ] 函数小（<50 行）
- [ ] 档案专注（<800 行）
- [ ] 没有深层巢状（>4 层）
- [ ] 适当的错误处理
- [ ] 没有 console.log 陈述式
- [ ] 没有寫死的值
- [ ] 没有变异（使用不可变模式）
