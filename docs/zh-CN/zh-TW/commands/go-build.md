---
description: Fix Go build errors, go vet warnings, and linter issues incrementally. Invokes the go-build-resolver agent for minimal, surgical fixes.
---

# Go 构建与修复

此指令呼叫 **go-build-resolver** Agent，以最小变更增量修复 Go 构建错误。

## 此指令的功能

1. **执行诊斷**：执行 `go build`、`go vet`、`staticcheck`
2. **解析错误**：依档案分组并依嚴重性排序
3. **增量修复**：一次一个错误
4. **验证每次修复**：每次变更后重新执行构建
5. **報告摘要**：显示已修复和剩余的问题

## 何时使用

在以下情况使用 `/go-build`：
- `go build ./...` 失敗并出现错误
- `go vet ./...` 報告问题
- `golangci-lint run` 显示警告
- 模块相依性损坏
- 拉取破坏构建的变更后

## 执行的诊斷指令

```bash
# 主要构建检查
go build ./...

# 静态分析
go vet ./...

# 扩展 linting（如果可用）
staticcheck ./...
golangci-lint run

# 模块问题
go mod verify
go mod tidy -v
```

## 常见修复的错误

| 错误 | 典型修复 |
|------|----------|
| `undefined: X` | 新增 import 或修正打字错误 |
| `cannot use X as Y` | 型別转换或修正赋值 |
| `missing return` | 新增 return 陈述式 |
| `X does not implement Y` | 新增缺少的方法 |
| `import cycle` | 重组套件 |
| `declared but not used` | 移除或使用变量 |
| `cannot find package` | `go get` 或 `go mod tidy` |

## 修复策略

1. **构建错误优先** - 程序代码必须编译
2. **Vet 警告次之** - 修复可疑构造
3. **Lint 警告第三** - 风格和最佳实务
4. **一次一个修复** - 验证每次变更
5. **最小变更** - 不要重构，只修复

## 停止条件

Agent 会在以下情况停止并報告：
- 3 次嘗试后同样错误仍存在
- 修复引入更多错误
- 需要架构变更
- 缺少外部相依性

## 相关指令

- `/go-test` - 构建成功后执行测试
- `/go-review` - 审查程序代码品质
- `/verify` - 完整验证回圈

## 相关

- Agent：`agents/go-build-resolver.md`
- 技能：`skills/golang-patterns/`
