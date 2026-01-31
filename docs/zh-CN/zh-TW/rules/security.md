# 安全性指南

## 強制安全性检查

任何提交前：
- [ ] 没有寫死的密钥（API 金钥、密码、Token）
- [ ] 所有使用者输入已验证
- [ ] SQL 注入防护（參数化查询）
- [ ] XSS 防护（清理过的 HTML）
- [ ] 已启用 CSRF 保护
- [ ] 已验证验证/授权
- [ ] 所有端点都有速率限制
- [ ] 错误讯息不会泄漏敏感资料

## 密钥管理

```typescript
// 绝不：寫死的密钥
const apiKey = "sk-proj-xxxxx"

// 总是：环境变量
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## 安全性回应协议

如果发现安全性问题：
1. 立即停止
2. 使用 **security-reviewer** Agent
3. 在继续前修复关键问题
4. 轮換任何暴露的密钥
5. 审查整个程序代码库是否有类似问题
