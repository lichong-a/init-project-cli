# TypeScript / JavaScript 规则

> 当项目选用 TypeScript 或 JavaScript 时，Agent 应遵循以下规则。

## 语言特性

- **静态类型**：启用 `strict` 模式，所有公共函数显式标注参数和返回类型
- **模块系统**：使用 ES Modules（`import/export`），禁止 `require`
- **异步模型**：`async/await` 优先，避免裸 Promise 链和回调地狱

## 类型系统

```typescript
// 优先 interface，type 用于联合类型/工具类型
interface UserProps {
  id: string
  name: string
}

type Status = 'active' | 'inactive'
type Nullable<T> = T | null

// 禁止 any，用 unknown + 类型守卫
function process(input: unknown): string {
  if (typeof input === 'string') return input
  throw new Error('Expected string')
}

// 使用 enum 替代魔法值
enum HttpStatus {
  OK = 200,
  NotFound = 404,
}

// 泛型约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

## 不可变性

```typescript
// 使用展开运算符 / map / filter / slice
const added = [...arr, newItem]
const removed = arr.filter(item => item.id !== targetId)
const updated = arr.map(item =>
  item.id === targetId ? { ...item, done: true } : item
)
```

## 导入规范

```typescript
// 分组排序：1. 外部库 → 2. 内部模块 → 3. 类型 → 4. 样式
import { ref, computed } from 'vue'              // 外部库
import { useUserStore } from '@/stores/user'      // 内部模块
import type { UserProfile } from '@/types/user'   // 类型
import './style.css'                              // 样式
```

## 构建工具

| 工具 | 适用场景 |
|------|----------|
| Vite | 前端项目（推荐） |
| tsup / tsdown | 库打包 |
| tsx | 脚本运行 |
| Vitest | 单元测试 |

## 前端框架选择

| 框架 | 状态管理 | 组件风格 |
|------|----------|----------|
| Vue 3 | Pinia | `<script setup>` + Composition API |
| React | Zustand / Redux Toolkit | 函数组件 + Hooks |
| Svelte | Svelte Stores | `.svelte` 单文件组件 |

## 禁止清单

| 禁止项 | 替代方案 |
|--------|----------|
| `any` | `unknown` + 类型守卫 |
| `require()` | `import` |
| `eval()` / `Function()` | 不使用 |
| `var` | `const` / `let` |
| `==` | `===` |
| `console.log` | 正式 logger |
