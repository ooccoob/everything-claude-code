---
description: Comprehensive Go code review for idiomatic patterns, concurrency safety, error handling, and security. Invokes the go-reviewer agent.
---

# Go 程序代码审查

此指令呼叫 **go-reviewer** Agent 进行全面的 Go 特定程序代码审查。

## 此指令的功能

1. **识别 Go 变更**：透过 `git diff` 找出修改的 `.go` 档案
2. **执行静态分析**：执行 `go vet`、`staticcheck` 和 `golangci-lint`
3. **安全性掃描**：检查 SQL 注入、命令注入、竞态条件
4. **并行审查**：分析 goroutine 安全性、channel 使用、mutex 模式
5. **惯用 Go 检查**：验证程序代码遵循 Go 慣例和最佳实务
6. **产生報告**：依嚴重性分类问题

## 何时使用

在以下情况使用 `/go-review`：
- 撰寫或修改 Go 程序代码后
- 提交 Go 变更前
- 审查包含 Go 程序代码的 PR
- 加入新的 Go 程序代码库时
- 学习惯用 Go 模式

## 审查类别

### 关键（必须修复）
- SQL/命令注入弱点
- 没有同步的竞态条件
- Goroutine 泄漏
- 寫死的憑证
- 不安全的指标使用
- 关键路徑中忽略错误

### 高（应该修复）
- 缺少帶上下文的错误包装
- 用 Panic 取代 Error 回传
- Context 未传递
- 无缓冲 channel 导致死锁
- 接口未满足错误
- 缺少 mutex 保护

### 中（考虑）
- 非惯用程序代码模式
- 导出项目缺少 godoc 注释
- 低效的字串串接
- Slice 未预分配
- 未使用表格驱动测试

## 执行的自动化检查

```bash
# 静态分析
go vet ./...

# 进阶检查（如果已安装）
staticcheck ./...
golangci-lint run

# 竞态侦测
go build -race ./...

# 安全性弱点
govulncheck ./...
```

## 批准标准

| 状态 | 条件 |
|------|------|
| ✅ 批准 | 没有关键或高优先问题 |
| ⚠️ 警告 | 只有中优先问题（谨慎合并）|
| ❌ 阻擋 | 发现关键或高优先问题 |

## 与其他指令的整合

- 先使用 `/go-test` 确保测试通过
- 如果发生构建错误，使用 `/go-build`
- 提交前使用 `/go-review`
- 对非 Go 特定问题使用 `/code-review`

## 相关

- Agent：`agents/go-reviewer.md`
- 技能：`skills/golang-patterns/`、`skills/golang-testing/`
