---
name: golang-patterns
description: Idiomatic Go patterns, best practices, and conventions for building robust, efficient, and maintainable Go applications.
---

# Go 开发模式

用於建构稳健、高效且可维护应用程序的惯用 Go 模式和最佳实务。

## 何时启用

- 撰寫新的 Go 程序代码
- 审查 Go 程序代码
- 重构现有 Go 程序代码
- 设计 Go 套件/模块

## 核心原则

### 1. 简单与清晰

Go 偏好简单而非聰明。程序代码应该明显且易读。

```go
// 良好：清晰直接
func GetUser(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    return user, nil
}

// 不良：过於聰明
func GetUser(id string) (*User, error) {
    return func() (*User, error) {
        if u, e := db.FindUser(id); e == nil {
            return u, nil
        } else {
            return nil, e
        }
    }()
}
```

### 2. 让零值有用

设计类型使其零值无需初始化即可立即使用。

```go
// 良好：零值有用
type Counter struct {
    mu    sync.Mutex
    count int // 零值为 0，可直接使用
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// 良好：bytes.Buffer 零值可用
var buf bytes.Buffer
buf.WriteString("hello")

// 不良：需要初始化
type BadCounter struct {
    counts map[string]int // nil map 会 panic
}
```

### 3. 接受接口，回传结构

函数应接受接口參数并回传具体类型。

```go
// 良好：接受接口，回传具体类型
func ProcessData(r io.Reader) (*Result, error) {
    data, err := io.ReadAll(r)
    if err != nil {
        return nil, err
    }
    return &Result{Data: data}, nil
}

// 不良：回传接口（不必要地隐藏实作细节）
func ProcessData(r io.Reader) (io.Reader, error) {
    // ...
}
```

## 错误处理模式

### 帶上下文的错误包装

```go
// 良好：包装错误并加上上下文
func LoadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("load config %s: %w", path, err)
    }

    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parse config %s: %w", path, err)
    }

    return &cfg, nil
}
```

### 自定义错误类型

```go
// 定义领域特定错误
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

// 常见情况的哨兵错误
var (
    ErrNotFound     = errors.New("resource not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInvalidInput = errors.New("invalid input")
)
```

### 使用 errors.Is 和 errors.As 检查错误

```go
func HandleError(err error) {
    // 检查特定错误
    if errors.Is(err, sql.ErrNoRows) {
        log.Println("No records found")
        return
    }

    // 检查错误类型
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        log.Printf("Validation error on field %s: %s",
            validationErr.Field, validationErr.Message)
        return
    }

    // 未知错误
    log.Printf("Unexpected error: %v", err)
}
```

### 绝不忽略错误

```go
// 不良：用空白识别符忽略错误
result, _ := doSomething()

// 良好：处理或明确说明为何安全忽略
result, err := doSomething()
if err != nil {
    return err
}

// 可接受：当错误真的不重要时（罕见）
_ = writer.Close() // 盡力清理，错误在其他地方记录
```

## 并行模式

### Worker Pool

```go
func WorkerPool(jobs <-chan Job, results chan<- Result, numWorkers int) {
    var wg sync.WaitGroup

    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }

    wg.Wait()
    close(results)
}
```

### 取消和逾时的 Context

```go
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, fmt.Errorf("create request: %w", err)
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("fetch %s: %w", url, err)
    }
    defer resp.Body.Close()

    return io.ReadAll(resp.Body)
}
```

### 优雅关闭

```go
func GracefulShutdown(server *http.Server) {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

    <-quit
    log.Println("Shutting down server...")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := server.Shutdown(ctx); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }

    log.Println("Server exited")
}
```

### 协调 Goroutines 的 errgroup

```go
import "golang.org/x/sync/errgroup"

func FetchAll(ctx context.Context, urls []string) ([][]byte, error) {
    g, ctx := errgroup.WithContext(ctx)
    results := make([][]byte, len(urls))

    for i, url := range urls {
        i, url := i, url // 捕获回圈变量
        g.Go(func() error {
            data, err := FetchWithTimeout(ctx, url)
            if err != nil {
                return err
            }
            results[i] = data
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err
    }
    return results, nil
}
```

### 避免 Goroutine 泄漏

```go
// 不良：如果 context 被取消会泄漏 goroutine
func leakyFetch(ctx context.Context, url string) <-chan []byte {
    ch := make(chan []byte)
    go func() {
        data, _ := fetch(url)
        ch <- data // 如果无接收者会永远阻塞
    }()
    return ch
}

// 良好：正确处理取消
func safeFetch(ctx context.Context, url string) <-chan []byte {
    ch := make(chan []byte, 1) // 帶缓冲的 channel
    go func() {
        data, err := fetch(url)
        if err != nil {
            return
        }
        select {
        case ch <- data:
        case <-ctx.Done():
        }
    }()
    return ch
}
```

## 接口设计

### 小而专注的接口

```go
// 良好：单一方法接口
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// 依需要组合接口
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

### 在使用处定义接口

```go
// 在消费者套件中，而非提供者
package service

// UserStore 定义此服务需要的内容
type UserStore interface {
    GetUser(id string) (*User, error)
    SaveUser(user *User) error
}

type Service struct {
    store UserStore
}

// 具体实作可以在另一个套件
// 它不需要知道这个接口
```

### 使用型別断言的可选行为

```go
type Flusher interface {
    Flush() error
}

func WriteAndFlush(w io.Writer, data []byte) error {
    if _, err := w.Write(data); err != nil {
        return err
    }

    // 如果支援则 Flush
    if f, ok := w.(Flusher); ok {
        return f.Flush()
    }
    return nil
}
```

## 套件组织

### 标准专案结构

```text
myproject/
├── cmd/
│   └── myapp/
│       └── main.go           # 进入点
├── internal/
│   ├── handler/              # HTTP handlers
│   ├── service/              # 业务逻辑
│   ├── repository/           # 资料存取
│   └── config/               # 设置
├── pkg/
│   └── client/               # 公开 API 客戶端
├── api/
│   └── v1/                   # API 定义（proto、OpenAPI）
├── testdata/                 # 测试 fixtures
├── go.mod
├── go.sum
└── Makefile
```

### 套件命名

```go
// 良好：简短、小寫、无底线
package http
package json
package user

// 不良：冗长、混合大小寫或冗余
package httpHandler
package json_parser
package userService // 冗余的 'Service' 后缀
```

### 避免套件层级状态

```go
// 不良：全域可变状态
var db *sql.DB

func init() {
    db, _ = sql.Open("postgres", os.Getenv("DATABASE_URL"))
}

// 良好：依赖注入
type Server struct {
    db *sql.DB
}

func NewServer(db *sql.DB) *Server {
    return &Server{db: db}
}
```

## 结构设计

### Functional Options 模式

```go
type Server struct {
    addr    string
    timeout time.Duration
    logger  *log.Logger
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) {
        s.timeout = d
    }
}

func WithLogger(l *log.Logger) Option {
    return func(s *Server) {
        s.logger = l
    }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{
        addr:    addr,
        timeout: 30 * time.Second, // 预设值
        logger:  log.Default(),    // 预设值
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// 使用方式
server := NewServer(":8080",
    WithTimeout(60*time.Second),
    WithLogger(customLogger),
)
```

### 嵌入用於组合

```go
type Logger struct {
    prefix string
}

func (l *Logger) Log(msg string) {
    fmt.Printf("[%s] %s\n", l.prefix, msg)
}

type Server struct {
    *Logger // 嵌入 - Server 获得 Log 方法
    addr    string
}

func NewServer(addr string) *Server {
    return &Server{
        Logger: &Logger{prefix: "SERVER"},
        addr:   addr,
    }
}

// 使用方式
s := NewServer(":8080")
s.Log("Starting...") // 呼叫嵌入的 Logger.Log
```

## 记忆体与效能

### 已知大小时预分配 Slice

```go
// 不良：多次扩展 slice
func processItems(items []Item) []Result {
    var results []Result
    for _, item := range items {
        results = append(results, process(item))
    }
    return results
}

// 良好：单次分配
func processItems(items []Item) []Result {
    results := make([]Result, 0, len(items))
    for _, item := range items {
        results = append(results, process(item))
    }
    return results
}
```

### 频繁分配使用 sync.Pool

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func ProcessRequest(data []byte) []byte {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf)
    }()

    buf.Write(data)
    // 处理...
    return buf.Bytes()
}
```

### 避免回圈中的字串串接

```go
// 不良：产生多次字串分配
func join(parts []string) string {
    var result string
    for _, p := range parts {
        result += p + ","
    }
    return result
}

// 良好：使用 strings.Builder 单次分配
func join(parts []string) string {
    var sb strings.Builder
    for i, p := range parts {
        if i > 0 {
            sb.WriteString(",")
        }
        sb.WriteString(p)
    }
    return sb.String()
}

// 最佳：使用标准函数库
func join(parts []string) string {
    return strings.Join(parts, ",")
}
```

## Go 工具整合

### 基本指令

```bash
# 构建和执行
go build ./...
go run ./cmd/myapp

# 测试
go test ./...
go test -race ./...
go test -cover ./...

# 静态分析
go vet ./...
staticcheck ./...
golangci-lint run

# 模块管理
go mod tidy
go mod verify

# 格式化
gofmt -w .
goimports -w .
```

### 建议的 Linter 设置（.golangci.yml）

```yaml
linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - unconvert
    - unparam

linters-settings:
  errcheck:
    check-type-assertions: true
  govet:
    check-shadowing: true

issues:
  exclude-use-default: false
```

## 快速參考：Go 惯用语

| 惯用语 | 描述 |
|-------|------|
| 接受接口，回传结构 | 函数接受接口參数，回传具体类型 |
| 错误是值 | 将错误视为一等值，而非例外 |
| 不要透过共享记忆体通讯 | 使用 channel 在 goroutine 间协调 |
| 让零值有用 | 类型应无需明确初始化即可工作 |
| 一点复制比一点依赖好 | 避免不必要的外部依赖 |
| 清晰优於聰明 | 优先考虑可读性而非聰明 |
| gofmt 不是任何人的最愛但是所有人的朋友 | 总是用 gofmt/goimports 格式化 |
| 提早返回 | 先处理错误，保持快樂路徑不缩排 |

## 要避免的反模式

```go
// 不良：长函数中的裸返回
func process() (result int, err error) {
    // ... 50 行 ...
    return // 返回什麼？
}

// 不良：使用 panic 作为控制流程
func GetUser(id string) *User {
    user, err := db.Find(id)
    if err != nil {
        panic(err) // 不要这样做
    }
    return user
}

// 不良：在结构中传递 context
type Request struct {
    ctx context.Context // Context 应该是第一个參数
    ID  string
}

// 良好：Context 作为第一个參数
func ProcessRequest(ctx context.Context, id string) error {
    // ...
}

// 不良：混合值和指标接收器
type Counter struct{ n int }
func (c Counter) Value() int { return c.n }    // 值接收器
func (c *Counter) Increment() { c.n++ }        // 指标接收器
// 选择一种风格并保持一致
```

**记住**：Go 程序代码应该以最好的方式无聊 - 可预测、一致且易於理解。有疑虑时，保持简单。
