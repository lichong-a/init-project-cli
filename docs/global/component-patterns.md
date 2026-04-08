# 组件模式 (Component Patterns)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **Composition API + `<script setup>`**：强制使用
- **Props Down, Events Up**：单向数据流
- **单一职责**：一个组件只做一件事
- **可组合 > 可配置**：通过组合而非 Props 配置实现灵活性

## 组件结构规范

```vue
<!-- MyComponent.vue -->
<script setup lang="ts">
// 1. 导入
import { ref, computed, watch } from 'vue'
import type { PropType } from 'vue'

// 2. Props & Emits 定义
interface Props {
  title: string
  items: Item[]
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
})

const emit = defineEmits<{
  select: [item: Item]
  delete: [id: string]
}>()

// 3. Composables
const router = useRouter()
const userStore = useUserStore()

// 4. 响应式状态
const selectedId = ref<string | null>(null)

// 5. 计算属性
const selectedItem = computed(() =>
  props.items.find(item => item.id === selectedId.value),
)

// 6. 方法
function handleSelect(item: Item): void {
  selectedId.value = item.id
  emit('select', item)
}

// 7. 生命周期
onMounted(() => {
  // 初始化
})
</script>

<template>
  <div class="my-component">
    <h2>{{ title }}</h2>
    <div v-if="loading">加载中...</div>
    <div v-else>
      <div
        v-for="item in items"
        :key="item.id"
        @click="handleSelect(item)"
      >
        {{ item.name }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.my-component {
  /* 样式 */
}
</style>
```

## 组件分类

```
shared/components/
├── ui/                 # 基础 UI 组件（无业务逻辑）
│   ├── Button.vue
│   ├── Input.vue
│   ├── Modal.vue
│   └── Table.vue
├── layout/             # 布局组件
│   ├── AppHeader.vue
│   ├── AppSidebar.vue
│   └── AppLayout.vue
└── business/           # 通用业务组件（跨模块复用）
    ├── UserAvatar.vue
    └── StatusBadge.vue
```

## 通用组件设计模式

### 受控组件模式

```vue
<!-- ControlledInput.vue -->
<script setup lang="ts">
interface Props {
  modelValue: string
  placeholder?: string
  error?: string
}

defineProps<Props>()
const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

function onInput(event: Event): void {
  const value = (event.target as HTMLInputElement).value
  emit('update:modelValue', value)
}
</script>

<template>
  <div class="input-wrapper">
    <input
      :value="modelValue"
      :placeholder="placeholder"
      @input="onInput"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>
```

### 插槽组合模式

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header" />
    </div>
    <div class="card-body">
      <slot />
    </div>
    <div class="card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>

<!-- 使用 -->
<Card>
  <template #header>
    <h3>标题</h3>
  </template>
  内容区域
  <template #footer>
    <Button>确定</Button>
  </template>
</Card>
```

### 加载/错误/空状态模式

```vue
<script setup lang="ts">
interface Props {
  loading: boolean
  error: string | null
  empty: boolean
  emptyText?: string
}

withDefaults(defineProps<Props>(), {
  emptyText: '暂无数据',
})
</script>

<template>
  <div v-if="loading" class="state-loading">
    <slot name="loading">
      <Spinner />
    </slot>
  </div>
  <div v-else-if="error" class="state-error">
    <slot name="error" :error="error">
      <Alert type="error">{{ error }}</Alert>
    </slot>
  </div>
  <div v-else-if="empty" class="state-empty">
    <slot name="empty">
      <EmptyState :text="emptyText" />
    </slot>
  </div>
  <slot v-else />
</template>
```

## 组件命名规范

| 类型 | 前缀 | 示例 |
|------|------|------|
| 基础 UI | 无 | `Button.vue`, `Input.vue` |
| 布局 | `App` | `AppHeader.vue`, `AppLayout.vue` |
| 业务组件 | 业务名 | `UserAvatar.vue`, `OrderList.vue` |
| 页面组件 | Page | `UserListPage.vue` |

## 组件评审清单

- [ ] 使用 `<script setup>` + TypeScript
- [ ] Props 有类型定义和默认值
- [ ] Emits 有类型定义
- [ ] 单一职责，不超过 200 行
- [ ] 使用 scoped 样式
- [ ] 支持 loading/error/empty 状态
- [ ] 可访问性（a11y）基本合规
- [ ] 有 key 绑定的列表渲染
