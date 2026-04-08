# 数据库规范 (Database Conventions)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 命名规范

### 表命名

| 规则 | 格式 | 示例 |
|------|------|------|
| 业务表 | snake_case 复数 | `users`, `orders`, `order_items` |
| 关联表 | 两表名组合 | `user_roles`, `product_tags` |
| 系统表 | `sys_` 前缀 | `sys_configs`, `sys_logs` |

### 字段命名

| 规则 | 格式 | 示例 |
|------|------|------|
| 主键 | `id` | `id` |
| 外键 | `关联表名_id` | `user_id`, `order_id` |
| 布尔 | `is_` 前缀 | `is_active`, `is_deleted` |
| 时间 | `动作_at` 后缀 | `created_at`, `updated_at`, `deleted_at` |
| 状态 | `status` 或 `xxx_status` | `status`, `payment_status` |
| 金额 | `xxx_amount` | `total_amount`, `discount_amount` |

## 表结构模板

```sql
-- 标准业务表模板
CREATE TABLE users (
  -- 主键
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- 业务字段
  email       VARCHAR(255) NOT NULL,
  name        VARCHAR(100) NOT NULL,
  role        VARCHAR(20)  NOT NULL DEFAULT 'user',
  status      VARCHAR(20)  NOT NULL DEFAULT 'active',

  -- 审计字段
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
  deleted_at  TIMESTAMPTZ  NULL,

  -- 约束
  CONSTRAINT uq_users_email UNIQUE (email),
  CONSTRAINT ck_users_role CHECK (role IN ('user', 'admin', 'moderator')),
  CONSTRAINT ck_users_status CHECK (status IN ('active', 'inactive', 'banned'))
);

-- 索引
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_users_status ON users (status);
CREATE INDEX idx_users_created_at ON users (created_at DESC);

-- 软删除过滤（视图）
CREATE VIEW v_active_users AS
  SELECT * FROM users WHERE deleted_at IS NULL;
```

## 数据类型选择

| 场景 | 类型 | 说明 |
|------|------|------|
| 主键 | UUID / BIGSERIAL | UUID 适合分布式，SERIAL 适合单库 |
| 字符串 | VARCHAR(n) | 明确长度限制 |
| 长文本 | TEXT | 无长度限制 |
| 布尔 | BOOLEAN | 避免 tinyint |
| 整数 | INTEGER / BIGINT | 根据范围选择 |
| 金额 | DECIMAL(10,2) | 避免浮点误差 |
| 时间 | TIMESTAMPTZ | 带时区的时间戳 |
| JSON | JSONB | 需要查询的 JSON 数据 |
| 枚举 | VARCHAR + CHECK | 比 ENUM 更灵活 |

## 查询规范

```sql
-- 避免 SELECT *，明确列出字段
SELECT id, email, name, role, status
FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;

-- 使用参数化查询（防 SQL 注入）
-- 好：使用占位符
SELECT * FROM users WHERE email = $1;

-- 坏：字符串拼接
-- SELECT * FROM users WHERE email = '${email}'

-- 分页查询使用游标分页（大数据量）
SELECT id, name, created_at
FROM users
WHERE created_at < $1  -- 上一页最后一条的 created_at
ORDER BY created_at DESC
LIMIT 20;

-- JOIN 规范：使用别名，明确字段来源
SELECT
  u.id AS user_id,
  u.name AS user_name,
  o.id AS order_id,
  o.total_amount
FROM users u
INNER JOIN orders o ON o.user_id = u.id
WHERE u.status = 'active';
```

## 迁移规范

```sql
-- 迁移文件命名：V{版本号}__{描述}.sql
-- V001__create_users_table.sql
-- V002__add_user_phone_column.sql
-- V003__create_orders_table.sql

-- 迁移必须可回滚
-- UP
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- DOWN
ALTER TABLE users DROP COLUMN phone;

-- 安全的列添加（大表）
-- 1. 添加可为空的列
ALTER TABLE users ADD COLUMN nickname VARCHAR(50);
-- 2. 分批回填
-- UPDATE users SET nickname = name WHERE nickname IS NULL LIMIT 1000;
-- 3. 添加约束（回填完成后）
-- ALTER TABLE users ALTER COLUMN nickname SET NOT NULL;
```

## 索引策略

```sql
-- 创建索引的原则
-- 1. WHERE 条件频繁使用的字段
-- 2. JOIN 关联的外键字段
-- 3. ORDER BY 排序字段
-- 4. 组合条件使用复合索引（遵循最左前缀）

-- 单列索引
CREATE INDEX idx_users_email ON users (email);

-- 复合索引（注意顺序）
CREATE INDEX idx_orders_user_status ON orders (user_id, status);

-- 部分索引（只索引需要的数据）
CREATE INDEX idx_users_active ON users (email)
  WHERE deleted_at IS NULL;

-- 避免过度索引
-- - 数据量 < 1000 的表通常不需要索引
-- - 频繁更新的表不要建太多索引
-- - 定期审查未使用的索引
```

## 数据库评审清单

- [ ] 表名/字段名遵循命名规范
- [ ] 主键使用 UUID 或 BIGSERIAL
- [ ] 有 created_at / updated_at 审计字段
- [ ] 外键有对应索引
- [ ] 查询使用参数化，无 SQL 注入风险
- [ ] 金额使用 DECIMAL 类型
- [ ] 时间使用 TIMESTAMPTZ 类型
- [ ] 迁移有回滚方案
- [ ] 大表操作考虑性能影响
