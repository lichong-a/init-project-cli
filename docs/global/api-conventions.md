# API 接口规范 (API Conventions)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## RESTful 设计原则

### URL 规范

```
GET    /api/v1/users           # 获取列表
GET    /api/v1/users/:id       # 获取详情
POST   /api/v1/users           # 创建
PUT    /api/v1/users/:id       # 全量更新
PATCH  /api/v1/users/:id       # 部分更新
DELETE /api/v1/users/:id       # 删除

# 嵌套资源
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# 动作类接口（非 CRUD）
POST   /api/v1/users/:id/activate
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
```

### 统一响应格式

**成功响应**
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "hasNext": true
  }
}
```

**错误响应**
```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "用户可读的错误消息",
    "details": {
      "email": ["邮箱格式不正确"]
    }
  }
}
```

### 分页参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| page | int | 1 | 页码，从 1 开始 |
| limit | int | 20 | 每页数量，最大 100 |
| sort | string | | 排序，如 `createdAt:desc` |

## Service 层规范

Service 层是 API 调用的唯一出口，所有 HTTP 请求必须通过 Service 封装：

```
Service 层职责：
├── 封装 HTTP 调用细节（URL 拼接、请求构建）
├── 处理响应解析（提取 data、转换格式）
├── 将 HTTP 错误转换为业务错误
└── 提供类型安全的接口给上层调用

Service 层不负责：
├── 状态管理（由 Store/调用方管理）
├── UI 交互（由 Component 管理）
└── 路由跳转（由调用方决定）
```

## HTTP 客户端要求

不论使用什么语言/框架，HTTP 客户端必须支持：

| 能力 | 说明 |
|------|------|
| 统一基础 URL | 通过配置/环境变量注入 |
| 自动认证 | Token 自动附加到请求头 |
| 统一错误处理 | HTTP 错误码 → 业务异常 |
| 请求取消 | 支持 AbortController / Context 超时 |
| 请求/响应拦截 | 统一处理日志、重试等 |

## 错误码规范

```
全局错误码：COMMON_*
认证错误码：AUTH_*
用户模块码：USER_*
订单模块码：ORDER_*

格式：<MODULE>_<NUMBER>
示例：USER_3001, AUTH_2001, COMMON_1001
```

| 范围 | 前缀 | 说明 |
|------|------|------|
| 通用 | COMMON_1xxx | 验证、404、冲突、限流 |
| 认证 | AUTH_2xxx | 凭证、Token、权限 |
| 用户 | USER_3xxx | 用户相关业务错误 |
| 自定义 | <MODULE>_Nxxx | 按模块分配号段 |

## API 评审清单

- [ ] URL 符合 RESTful 规范
- [ ] 响应格式统一（success/data/error）
- [ ] 分页参数标准化
- [ ] 错误码遵循编号规范
- [ ] Service 层封装完整
- [ ] 认证 Token 自动注入
- [ ] 请求取消支持
