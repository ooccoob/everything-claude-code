---
description: Enforce TDD workflow for Go. Write table-driven tests first, then implement. Verify 80%+ coverage with go test -cover.
---

# Go TDD 指令

此指令強制执行 Go 程序代码的测试驱动开发方法论，使用惯用的 Go 测试模式。

## 此指令的功能

1. **定义类型/接口**：先建立函数签名骨架
2. **撰寫表格驱动测试**：建立全面的测试案例（RED）
3. **执行测试**：验证测试因正确的原因失敗
4. **实作程序代码**：撰寫最小程序代码使其通过（GREEN）
5. **重构**：在测试保持绿色的同时改进
6. **检查覆盖率**：确保 80% 以上覆盖率

## 何时使用

在以下情况使用 `/go-test`：
- 实作新的 Go 函数
- 为现有程序代码新增测试覆盖率
- 修复 Bug（先撰寫失敗的测试）
- 建构关键商业逻辑
- 学习 Go 中的 TDD 工作流程

## TDD 循环

```
RED     → 撰寫失敗的表格驱动测试
GREEN   → 实作最小程序代码使其通过
REFACTOR → 改进程序代码，测试保持绿色
REPEAT  → 下一个测试案例
```

## 测试模式

### 表格驱动测试
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"case 1", input1, want1, false},
    {"case 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // 断言
    })
}
```

### 平行测试
```go
for _, tt := range tests {
    tt := tt // 撷取
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // 测试内容
    })
}
```

### 测试辅助函数
```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db := createDB()
    t.Cleanup(func() { db.Close() })
    return db
}
```

## 覆盖率指令

```bash
# 基本覆盖率
go test -cover ./...

# 覆盖率 profile
go test -coverprofile=coverage.out ./...

# 在浏览器检视
go tool cover -html=coverage.out

# 依函数显示覆盖率
go tool cover -func=coverage.out

# 帶竞态侦测
go test -race -cover ./...
```

## 覆盖率目标

| 程序代码类型 | 目标 |
|-----------|------|
| 关键商业逻辑 | 100% |
| 公开 API | 90%+ |
| 一般程序代码 | 80%+ |
| 产生的程序代码 | 排除 |

## TDD 最佳实务

**应该做：**
- 在任何实作前先撰寫测试
- 每次变更后执行测试
- 使用表格驱动测试以获得全面覆盖
- 测试行为，不是实作细节
- 包含边界情况（空值、nil、最大值）

**不应该做：**
- 在测试之前撰寫实作
- 跳过 RED 阶段
- 直接测试私有函数
- 在测试中使用 `time.Sleep`
- 忽略不稳定的测试

## 相关指令

- `/go-build` - 修复构建错误
- `/go-review` - 实作后审查程序代码
- `/verify` - 执行完整验证回圈

## 相关

- 技能：`skills/golang-testing/`
- 技能：`skills/tdd-workflow/`
