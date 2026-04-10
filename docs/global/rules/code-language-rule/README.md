# 语言规则索引

> 此目录包含各语言的 Agent 指导规则。
> 当项目确定技术栈后，Agent 应读取对应语言文件获取具体规范。

## 使用方式

1. 项目启动时确定技术栈
2. 读取对应语言规则文件
3. 全局规范提供原则，语言规则提供具体实践

## 可用规则

| 文件 | 语言 | 核心关注点 |
|------|------|------------|
| [typescript.md](./typescript.md) | TypeScript / JavaScript | 类型系统、ES Modules、async/await、构建工具 |
| [python.md](./python.md) | Python | Type Hints、Pydantic、FastAPI、ruff |
| [go.md](./go.md) | Go | 错误处理、并发模式、接口设计、表驱动测试 |
| [rust.md](./rust.md) | Rust | 所有权系统、Result 错误处理、async/await |
| [java.md](./java.md) | Java / Spring Boot | 分层架构、异常处理、依赖注入、JPA |

## 扩展规则

如需添加新语言规则，创建 `<language>.md` 文件，包含以下章节：

1. **语言特性**：版本、工具链、核心特性
2. **类型系统**（如适用）：类型安全实践
3. **项目结构**：推荐的目录组织
4. **错误处理**：该语言的错误处理范式
5. **测试规范**：测试框架和模式
6. **禁止清单**：语言特定的反模式
