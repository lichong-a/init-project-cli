# 错误处理规范 (Error Handling)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **错误分类**：区分业务错误、系统错误、网络错误
- **错误传播**：底层抛出具体错误，边界层统一处理
- **错误恢复**：提供降级方案，不让用户看到技术细节
- **错误追踪**：关键错误记录日志，便于排查

## 错误分类体系

```
AppError（基础错误）
├── NetworkError（网络层）
│   ├── 连接超时
│   ├── 服务不可用
│   └── HTTP 状态码错误
├── ValidationError（验证层）
│   └── 字段级验证错误
├── BusinessError（业务层）
│   └── 业务规则违反
├── AuthorizationError（权限层）
│   ├── 未认证
│   └── 无权限
└── <Module>Error（模块特定）
```

## 错误处理层次

### 第一层：Service/Repository 层（抛出具体错误）

```
function createUser(input):
  // 输入验证
  if not validate(input):
    throw ValidationError("数据验证失败", fields)

  try:
    return httpClient.post('/users', validated)
  catch NetworkError as e:
    if e.statusCode == 409:
      throw BusinessError("邮箱已被注册", "USER_EMAIL_EXISTS")
    throw e  // 向上传播
```

### 第二层：业务逻辑层（转换和降级）

```
function handleUserCreate(input):
  error = null
  fieldErrors = {}

  try:
    return userService.create(input)
  catch ValidationError as e:
    fieldErrors = e.fields
    error = e.message
  catch BusinessError as e:
    error = e.message
  catch other:
    error = "操作失败，请稍后重试"
    logger.error("Unexpected error", other)

  return { error, fieldErrors }
```

### 第三层：UI 层（展示和兜底）

```
展示错误消息：
  - ValidationError → 表单字段高亮
  - BusinessError   → 弹窗/提示
  - NetworkError    → 全局网络提示
  - 其他            → 兜底错误页
```

### 第四层：全局兜底

```
// 未捕获错误的全局处理器
onUncaughtError(error):
  logger.error("Uncaught", error)

  if error is AuthorizationError:
    navigateTo('/login')
  else if error is NetworkError:
    showGlobalToast("网络连接异常")
  else:
    showGlobalToast("系统异常，请稍后重试")
```

## 异步错误处理模式

### 模式一：Try/Catch（推荐用于需要错误转换的场景）

```
async function fetchData():
  try:
    result = await httpClient.get('/data')
    return result
  catch error:
    throw BusinessError("数据获取失败", error)
```

### 模式二：Result 模式（推荐用于不抛错的场景）

```
type Result<T> =
  | { success: true, data: T }
  | { success: false, error: AppError }

async function safeFetch<T>(url): Result<T>
  try:
    data = await httpClient.get(url)
    return { success: true, data }
  catch error:
    return { success: false, error: normalize(error) }
```

## 错误处理检查清单

- [ ] Service 层抛出具体错误类型
- [ ] 边界层（业务逻辑）处理并降级
- [ ] UI 层展示用户友好的错误消息
- [ ] 全局兜底处理未捕获错误
- [ ] 关键错误记录到日志系统
- [ ] 错误消息不泄露内部实现细节
- [ ] 网络错误有重试机制
