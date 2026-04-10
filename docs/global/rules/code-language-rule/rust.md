# Rust 规则

> 当项目选用 Rust 时，Agent 应遵循以下规则。

## 语言特性

- **所有权系统**：默认不可变，`mut` 显式标注可变
- **错误处理**：`Result<T, E>` + `?` 操作符，禁止业务代码中使用 `unwrap()`
- **包管理**：`cargo`，依赖声明在 `Cargo.toml`
- **Rust 版本**：最新 stable edition

## 错误处理

```rust
// 使用 thiserror 定义错误类型
#[derive(Debug, thiserror::Error)]
enum AppError {
    #[error("user not found: {id}")]
    NotFound { id: String },

    #[error("validation failed: {0}")]
    Validation(String),

    #[error(transparent)]
    Internal(#[from] anyhow::Error),
}

// 使用 ? 传播错误
fn get_user(id: &str) -> Result<User, AppError> {
    let row = db.query(id)?;  // ? 自动传播
    row.try_into()  // 返回 Result
}
```

## 项目结构

```
├── src/
│   ├── main.rs         # 二进制入口
│   ├── lib.rs          # 库入口
│   ├── handlers/       # 请求处理
│   ├── services/       # 业务逻辑
│   ├── repositories/   # 数据访问
│   ├── models/         # 数据模型
│   └── error.rs        # 错误类型
├── tests/              # 集成测试
├── benches/            # 基准测试
├── Cargo.toml
└── Cargo.lock
```

## 并发模式

```rust
// 使用 tokio 异步运行时
#[tokio::main]
async fn main() -> Result<()> {
    // spawn 并发任务
    let handle = tokio::spawn(async move {
        do_work().await
    });

    let result = handle.await??;
    Ok(())
}
```

## 类型设计

```rust
// 使用 newtype 包装原始类型
struct UserId(String);
struct Email(String);

// 使用枚举建模有限状态
enum OrderStatus {
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
}

// 使用 Builder 模式构建复杂对象
let user = UserBuilder::new()
    .name("Alice")
    .email("alice@example.com")
    .build()?;
```

## 测试规范

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_format_date() {
        let result = format_date("2024-01-15T08:30:00Z").unwrap();
        assert_eq!(result, "2024-01-15");
    }

    #[test]
    fn test_format_date_invalid() {
        assert!(format_date("invalid").is_err());
    }
}
```

## 禁止清单

| 禁止项 | 替代方案 |
|--------|----------|
| `unwrap()` 在业务代码 | `?` + `Result` |
| `expect()` 在业务代码 | `?` + 自定义错误 |
| `panic!` | `Result` 传播 |
| `clone()` 过度使用 | 调整所有权/借用 |
| `unsafe` | 安全抽象封装 |
| 大量 `Arc<Mutex<>>` | 重新设计数据流 |
