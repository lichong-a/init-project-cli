# 编码标准 (Coding Standards)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **不可变性优先**：永远创建新对象/数据结构，不要修改原对象
- **函数式风格**：纯函数、无副作用、显式数据流
- **小文件优先**：单文件 200-400 行为佳，上限 800 行
- **小函数优先**：单函数 <50 行，单一职责

## 命名规范

### 文件命名

| 类型 | 格式 | 示例 |
|------|------|------|
| 组件/类文件 | PascalCase 或 camelCase | `UserProfile`, `userProfile` |
| 工具/函数文件 | camelCase / snake_case | `formatDate`, `format_date` |
| 类型/接口文件 | 同项目约定 | `userTypes`, `user_types` |
| 常量文件 | UPPER_SNAKE / 同项目约定 | `API_ENDPOINTS` |
| 测试文件 | 被测文件名 + test/spec 后缀 | `formatDate.test`, `format_date_spec` |

### 代码命名

```
变量/函数：camelCase 或 snake_case（跟随语言惯例）
  - JavaScript/TypeScript/Java: getUserById
  - Python/Rust/Go: get_user_by_id

类/接口/类型：PascalCase
  - UserService, UserProfile, UserRole

常量：UPPER_SNAKE_CASE
  - MAX_RETRY_COUNT, API_BASE_URL

布尔变量：is/has/can/should 前缀
  - isLoading, has_permission, can_edit

私有成员：遵循语言惯例
  - Python: _private
  - Java: private field
  - Go: 小写开头
  - Rust: _field（惯例）
```

## 类型安全

> 任何语言都应追求类型安全，具体方式因语言而异。

| 语言 | 类型安全方式 |
|------|-------------|
| TypeScript | 严格模式、显式类型注解、避免 `any` |
| Python | Type Hints + mypy/pyright |
| Java/Kotlin | 静态类型系统 |
| Go | 静态类型、显式错误处理 |
| Rust | 类型系统 + 编译器强制 |

**通用规则**：
- 公共函数必须有参数和返回类型声明
- 避免"魔法类型"（如 any、Object、interface{}）
- 使用枚举/常量替代魔法值
- 泛型/模板用于可复用的类型安全抽象

## 不可变性实践

```
// 所有语言通用原则：
// WRONG: 修改原数据
function addItem(cart, item) {
  cart.items.push(item)  // MUTATION!
  return cart
}

// CORRECT: 创建新数据
function addItem(cart, item) {
  return { ...cart, items: [...cart.items, item] }
}
```

**语言特定实现**：
- TypeScript/JavaScript: 展开运算符、Object.freeze
- Python: `@dataclass(frozen=True)`, tuple, `copy.deepcopy`
- Rust: 所有权系统天然不可变，`mut` 显式标注
- Go: 值传递、避免指针修改

## 导入规范

所有语言遵循统一的导入分组顺序：

```
1. 标准库 / 外部依赖
2. 内部模块
3. 类型定义
4. 样式 / 资源（如适用）
```

## 禁止清单

| 禁止项 | 原因 |
|--------|------|
| 调试输出遗留 | 用正式 logger 替代 console.log/print/etc |
| 动态类型逃逸 | 用 `unknown` + 类型守卫替代 `any`/`Object` |
| 硬编码值 | 抽取为常量/配置 |
| 嵌套 > 4 层 | 提前返回/抽取函数 |
| 魔法数字/字符串 | 定义常量/枚举 |
| 直接修改参数 | 保持不可变性 |
| 动态代码执行 | `eval()` / `exec()` 等安全风险 |

## 代码质量检查清单

- [ ] 代码可读且命名清晰
- [ ] 函数 <50 行，文件 <800 行
- [ ] 无深层嵌套（≤4 层）
- [ ] 错误处理完善
- [ ] 无调试输出遗留
- [ ] 无硬编码值
- [ ] 使用不可变模式
- [ ] 类型安全（利用语言机制）
