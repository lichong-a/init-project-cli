# Go 规则

> 当项目选用 Go 时，Agent 应遵循以下规则。

## 语言特性

- **错误处理**：显式检查每个 error，禁止 `_` 忽略 error
- **包组织**：按领域划分包，避免 `utils` / `helpers` 等泛化包名
- **代码格式**：`gofmt` / `goimports`，使用 `golangci-lint`
- **Go 版本**：最低 1.22+（支持 range over func 等新特性）

## 错误处理

```go
// 正确：显式处理
result, err := service.DoSomething()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}

// 错误：忽略 error
result, _ := service.DoSomething()  // 禁止

// 自定义错误类型
type AppError struct {
    Code    string
    Message string
    Err     error
}

func (e *AppError) Error() string {
    return fmt.Sprintf("[%s] %s: %v", e.Code, e.Message, e.Err)
}
```

## 项目结构

```
├── cmd/                # 应用入口
│   └── server/
│       └── main.go
├── internal/           # 私有代码（不可外部导入）
│   ├── handler/        # HTTP Handler
│   ├── service/        # 业务逻辑
│   ├── repository/     # 数据访问
│   └── model/          # 数据模型
├── pkg/                # 可公开使用的库
├── api/                # API 定义（OpenAPI/Proto）
├── go.mod
└── go.sum
```

## 并发模式

```go
// 使用 context 控制超时和取消
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

// goroutine 必须有退出机制
// 使用 sync.WaitGroup 或 errgroup 管理生命周期
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
    return doWork(ctx)
})

if err := g.Wait(); err != nil {
    return err
}
```

## 接口设计

```go
// 在使用方定义接口，不是实现方
// 小接口，单一方法
type UserRepository interface {
    GetByID(ctx context.Context, id string) (*User, error)
    Create(ctx context.Context, user *User) error
}
```

## 测试规范

```go
// 表驱动测试
func TestFormatDate(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
    }{
        {"valid date", "2024-01-15", "2024-01-15"},
        {"empty input", "", "--"},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := formatDate(tt.input)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```

## 禁止清单

| 禁止项 | 替代方案 |
|--------|----------|
| `panic` 在业务代码 | 返回 `error` |
| `_` 忽略 error | 显式处理 |
| `init()` 滥用 | 显式初始化函数 |
| 全局可变状态 | 依赖注入 |
| 过大的接口 | 小接口（1-3 个方法） |
