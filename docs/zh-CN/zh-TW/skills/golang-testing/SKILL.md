---
name: golang-testing
description: Go testing patterns including table-driven tests, subtests, benchmarks, fuzzing, and test coverage. Follows TDD methodology with idiomatic Go practices.
---

# Go 测试模式

用於撰寫可靠、可维护测试的完整 Go 测试模式，遵循 TDD 方法论。

## 何时启用

- 撰寫新的 Go 函数或方法
- 为现有程序代码增加测试覆盖率
- 为效能关键程序代码建立基准测试
- 实作输入验证的模糊测试
- 在 Go 专案中遵循 TDD 工作流程

## Go 的 TDD 工作流程

### RED-GREEN-REFACTOR 循环

```
RED     → 先寫失敗的测试
GREEN   → 撰寫最少程序代码使测试通过
REFACTOR → 在保持测试绿色的同时改善程序代码
REPEAT  → 继续下一个需求
```

### Go 中的逐步 TDD

```go
// 步骤 1：定义接口/签章
// calculator.go
package calculator

func Add(a, b int) int {
    panic("not implemented") // 佔位符
}

// 步骤 2：撰寫失敗测试（RED）
// calculator_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2, 3) = %d; want %d", got, want)
    }
}

// 步骤 3：执行测试 - 验证失敗
// $ go test
// --- FAIL: TestAdd (0.00s)
// panic: not implemented

// 步骤 4：实作最少程序代码（GREEN）
func Add(a, b int) int {
    return a + b
}

// 步骤 5：执行测试 - 验证通过
// $ go test
// PASS

// 步骤 6：如需要则重构，验证测试仍然通过
```

## 表格驱动测试

Go 测试的标准模式。以最少程序代码达到完整覆盖。

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -1, -2, -3},
        {"zero values", 0, 0, 0},
        {"mixed signs", -1, 1, 0},
        {"large numbers", 1000000, 2000000, 3000000},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d",
                    tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

### 帶错误案例的表格驱动测试

```go
func TestParseConfig(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    *Config
        wantErr bool
    }{
        {
            name:  "valid config",
            input: `{"host": "localhost", "port": 8080}`,
            want:  &Config{Host: "localhost", Port: 8080},
        },
        {
            name:    "invalid JSON",
            input:   `{invalid}`,
            wantErr: true,
        },
        {
            name:    "empty input",
            input:   "",
            wantErr: true,
        },
        {
            name:  "minimal config",
            input: `{}`,
            want:  &Config{}, // 零值 config
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseConfig(tt.input)

            if tt.wantErr {
                if err == nil {
                    t.Error("expected error, got nil")
                }
                return
            }

            if err != nil {
                t.Fatalf("unexpected error: %v", err)
            }

            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("got %+v; want %+v", got, tt.want)
            }
        })
    }
}
```

## 子测试

### 组织相关测试

```go
func TestUser(t *testing.T) {
    // 所有子测试共享的设置
    db := setupTestDB(t)

    t.Run("Create", func(t *testing.T) {
        user := &User{Name: "Alice"}
        err := db.CreateUser(user)
        if err != nil {
            t.Fatalf("CreateUser failed: %v", err)
        }
        if user.ID == "" {
            t.Error("expected user ID to be set")
        }
    })

    t.Run("Get", func(t *testing.T) {
        user, err := db.GetUser("alice-id")
        if err != nil {
            t.Fatalf("GetUser failed: %v", err)
        }
        if user.Name != "Alice" {
            t.Errorf("got name %q; want %q", user.Name, "Alice")
        }
    })

    t.Run("Update", func(t *testing.T) {
        // ...
    })

    t.Run("Delete", func(t *testing.T) {
        // ...
    })
}
```

### 并行子测试

```go
func TestParallel(t *testing.T) {
    tests := []struct {
        name  string
        input string
    }{
        {"case1", "input1"},
        {"case2", "input2"},
        {"case3", "input3"},
    }

    for _, tt := range tests {
        tt := tt // 捕获范围变量
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // 并行执行子测试
            result := Process(tt.input)
            // 断言...
            _ = result
        })
    }
}
```

## 测试辅助函数

### 辅助函数

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper() // 标记为辅助函数

    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("failed to open database: %v", err)
    }

    // 测试结束时清理
    t.Cleanup(func() {
        db.Close()
    })

    // 执行 migrations
    if _, err := db.Exec(schema); err != nil {
        t.Fatalf("failed to create schema: %v", err)
    }

    return db
}

func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func assertEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got != want {
        t.Errorf("got %v; want %v", got, want)
    }
}
```

### 临时档案和目录

```go
func TestFileProcessing(t *testing.T) {
    // 建立临时目录 - 自动清理
    tmpDir := t.TempDir()

    // 建立测试档案
    testFile := filepath.Join(tmpDir, "test.txt")
    err := os.WriteFile(testFile, []byte("test content"), 0644)
    if err != nil {
        t.Fatalf("failed to create test file: %v", err)
    }

    // 执行测试
    result, err := ProcessFile(testFile)
    if err != nil {
        t.Fatalf("ProcessFile failed: %v", err)
    }

    // 断言...
    _ = result
}
```

## Golden 档案

使用保存在 `testdata/` 中的预期输出档案进行测试。

```go
var update = flag.Bool("update", false, "update golden files")

func TestRender(t *testing.T) {
    tests := []struct {
        name  string
        input Template
    }{
        {"simple", Template{Name: "test"}},
        {"complex", Template{Name: "test", Items: []string{"a", "b"}}},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Render(tt.input)

            golden := filepath.Join("testdata", tt.name+".golden")

            if *update {
                // 更新 golden 档案：go test -update
                err := os.WriteFile(golden, got, 0644)
                if err != nil {
                    t.Fatalf("failed to update golden file: %v", err)
                }
            }

            want, err := os.ReadFile(golden)
            if err != nil {
                t.Fatalf("failed to read golden file: %v", err)
            }

            if !bytes.Equal(got, want) {
                t.Errorf("output mismatch:\ngot:\n%s\nwant:\n%s", got, want)
            }
        })
    }
}
```

## 使用接口 Mock

### 基於接口的 Mock

```go
// 定义依赖的接口
type UserRepository interface {
    GetUser(id string) (*User, error)
    SaveUser(user *User) error
}

// 生产实作
type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) GetUser(id string) (*User, error) {
    // 实际资料库查询
}

// 测试用 Mock 实作
type MockUserRepository struct {
    GetUserFunc  func(id string) (*User, error)
    SaveUserFunc func(user *User) error
}

func (m *MockUserRepository) GetUser(id string) (*User, error) {
    return m.GetUserFunc(id)
}

func (m *MockUserRepository) SaveUser(user *User) error {
    return m.SaveUserFunc(user)
}

// 使用 mock 的测试
func TestUserService(t *testing.T) {
    mock := &MockUserRepository{
        GetUserFunc: func(id string) (*User, error) {
            if id == "123" {
                return &User{ID: "123", Name: "Alice"}, nil
            }
            return nil, ErrNotFound
        },
    }

    service := NewUserService(mock)

    user, err := service.GetUserProfile("123")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if user.Name != "Alice" {
        t.Errorf("got name %q; want %q", user.Name, "Alice")
    }
}
```

## 基准测试

### 基本基准测试

```go
func BenchmarkProcess(b *testing.B) {
    data := generateTestData(1000)
    b.ResetTimer() // 不计算设置时间

    for i := 0; i < b.N; i++ {
        Process(data)
    }
}

// 执行：go test -bench=BenchmarkProcess -benchmem
// 输出：BenchmarkProcess-8   10000   105234 ns/op   4096 B/op   10 allocs/op
```

### 不同大小的基准测试

```go
func BenchmarkSort(b *testing.B) {
    sizes := []int{100, 1000, 10000, 100000}

    for _, size := range sizes {
        b.Run(fmt.Sprintf("size=%d", size), func(b *testing.B) {
            data := generateRandomSlice(size)
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                // 复制以避免排序已排序的资料
                tmp := make([]int, len(data))
                copy(tmp, data)
                sort.Ints(tmp)
            }
        })
    }
}
```

### 记忆体分配基准测试

```go
func BenchmarkStringConcat(b *testing.B) {
    parts := []string{"hello", "world", "foo", "bar", "baz"}

    b.Run("plus", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            var s string
            for _, p := range parts {
                s += p
            }
            _ = s
        }
    })

    b.Run("builder", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            var sb strings.Builder
            for _, p := range parts {
                sb.WriteString(p)
            }
            _ = sb.String()
        }
    })

    b.Run("join", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            _ = strings.Join(parts, "")
        }
    })
}
```

## 模糊测试（Go 1.18+）

### 基本模糊测试

```go
func FuzzParseJSON(f *testing.F) {
    // 新增种子语料库
    f.Add(`{"name": "test"}`)
    f.Add(`{"count": 123}`)
    f.Add(`[]`)
    f.Add(`""`)

    f.Fuzz(func(t *testing.T, input string) {
        var result map[string]interface{}
        err := json.Unmarshal([]byte(input), &result)

        if err != nil {
            // 随机输入预期会有无效 JSON
            return
        }

        // 如果解析成功，重新编码应该可行
        _, err = json.Marshal(result)
        if err != nil {
            t.Errorf("Marshal failed after successful Unmarshal: %v", err)
        }
    })
}

// 执行：go test -fuzz=FuzzParseJSON -fuzztime=30s
```

### 多输入模糊测试

```go
func FuzzCompare(f *testing.F) {
    f.Add("hello", "world")
    f.Add("", "")
    f.Add("abc", "abc")

    f.Fuzz(func(t *testing.T, a, b string) {
        result := Compare(a, b)

        // 属性：Compare(a, a) 应该总是等於 0
        if a == b && result != 0 {
            t.Errorf("Compare(%q, %q) = %d; want 0", a, b, result)
        }

        // 属性：Compare(a, b) 和 Compare(b, a) 应该有相反符号
        reverse := Compare(b, a)
        if (result > 0 && reverse >= 0) || (result < 0 && reverse <= 0) {
            if result != 0 || reverse != 0 {
                t.Errorf("Compare(%q, %q) = %d, Compare(%q, %q) = %d; inconsistent",
                    a, b, result, b, a, reverse)
            }
        }
    })
}
```

## 测试覆盖率

### 执行覆盖率

```bash
# 基本覆盖率
go test -cover ./...

# 产生覆盖率 profile
go test -coverprofile=coverage.out ./...

# 在浏览器查看覆盖率
go tool cover -html=coverage.out

# 按函数查看覆盖率
go tool cover -func=coverage.out

# 含竞态侦测的覆盖率
go test -race -coverprofile=coverage.out ./...
```

### 覆盖率目标

| 程序代码类型 | 目标 |
|-----------|------|
| 关键业务逻辑 | 100% |
| 公开 API | 90%+ |
| 一般程序代码 | 80%+ |
| 产生的程序代码 | 排除 |

## HTTP Handler 测试

```go
func TestHealthHandler(t *testing.T) {
    // 建立请求
    req := httptest.NewRequest(http.MethodGet, "/health", nil)
    w := httptest.NewRecorder()

    // 呼叫 handler
    HealthHandler(w, req)

    // 检查回应
    resp := w.Result()
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        t.Errorf("got status %d; want %d", resp.StatusCode, http.StatusOK)
    }

    body, _ := io.ReadAll(resp.Body)
    if string(body) != "OK" {
        t.Errorf("got body %q; want %q", body, "OK")
    }
}

func TestAPIHandler(t *testing.T) {
    tests := []struct {
        name       string
        method     string
        path       string
        body       string
        wantStatus int
        wantBody   string
    }{
        {
            name:       "get user",
            method:     http.MethodGet,
            path:       "/users/123",
            wantStatus: http.StatusOK,
            wantBody:   `{"id":"123","name":"Alice"}`,
        },
        {
            name:       "not found",
            method:     http.MethodGet,
            path:       "/users/999",
            wantStatus: http.StatusNotFound,
        },
        {
            name:       "create user",
            method:     http.MethodPost,
            path:       "/users",
            body:       `{"name":"Bob"}`,
            wantStatus: http.StatusCreated,
        },
    }

    handler := NewAPIHandler()

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            var body io.Reader
            if tt.body != "" {
                body = strings.NewReader(tt.body)
            }

            req := httptest.NewRequest(tt.method, tt.path, body)
            req.Header.Set("Content-Type", "application/json")
            w := httptest.NewRecorder()

            handler.ServeHTTP(w, req)

            if w.Code != tt.wantStatus {
                t.Errorf("got status %d; want %d", w.Code, tt.wantStatus)
            }

            if tt.wantBody != "" && w.Body.String() != tt.wantBody {
                t.Errorf("got body %q; want %q", w.Body.String(), tt.wantBody)
            }
        })
    }
}
```

## 测试指令

```bash
# 执行所有测试
go test ./...

# 执行详细输出的测试
go test -v ./...

# 执行特定测试
go test -run TestAdd ./...

# 执行匹配模式的测试
go test -run "TestUser/Create" ./...

# 执行帶竞态侦测器的测试
go test -race ./...

# 执行帶覆盖率的测试
go test -cover -coverprofile=coverage.out ./...

# 只执行短测试
go test -short ./...

# 执行帶逾时的测试
go test -timeout 30s ./...

# 执行基准测试
go test -bench=. -benchmem ./...

# 执行模糊测试
go test -fuzz=FuzzParse -fuzztime=30s ./...

# 计算测试执行次数（用於侦测不稳定测试）
go test -count=10 ./...
```

## 最佳实务

**应该做的：**
- 先寫测试（TDD）
- 使用表格驱动测试以获得完整覆盖
- 测试行为，而非实作
- 在辅助函数中使用 `t.Helper()`
- 对独立测试使用 `t.Parallel()`
- 用 `t.Cleanup()` 清理资源
- 使用描述情境的有意义测试名称

**不应该做的：**
- 不要直接测试私有函数（透过公开 API 测试）
- 不要在测试中使用 `time.Sleep()`（使用 channels 或条件）
- 不要忽略不稳定测试（修复或移除它们）
- 不要 mock 所有東西（可能时偏好整合测试）
- 不要跳过错误路徑测试

## CI/CD 整合

```yaml
# GitHub Actions 范例
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-go@v5
      with:
        go-version: '1.22'

    - name: Run tests
      run: go test -race -coverprofile=coverage.out ./...

    - name: Check coverage
      run: |
        go tool cover -func=coverage.out | grep total | awk '{print $3}' | \
        awk -F'%' '{if ($1 < 80) exit 1}'
```

**记住**：测试是文件。它们展示你的程序代码应该如何使用。清楚地撰寫并保持更新。
