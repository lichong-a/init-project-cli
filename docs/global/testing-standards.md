# 测试标准 (Testing Standards)

> 适用范围：全局规范 | 稳定规则 | 跨 Feature 复用

## 核心原则

- **TDD 强制**：先写测试，再写实现
- **覆盖率门槛**：最低 80%，核心业务逻辑 90%+
- **测试金字塔**：单元 > 集成 > E2E（7:2:1）
- **测试独立性**：每个测试可独立运行，无执行顺序依赖

## 测试分层

```
tests/
├── unit/                    # 单元测试（70%）
│   ├── utils/               # 工具函数测试
│   ├── services/            # 服务层测试
│   └── models/              # 模型测试
├── integration/             # 集成测试（20%）
│   ├── api/                 # API 集成测试
│   └── modules/             # 模块集成测试
└── e2e/                     # 端到端测试（10%）
    ├── auth.flow            # 认证流程
    └── core-flows.flow      # 核心业务流程
```

## 单元测试规范

所有语言统一的测试结构：

```
describe('模块名/函数名')
  describe('方法名或场景')
    it('应该 [预期行为] 当 [条件]')   ← 中文描述
    it('should [expected behavior] when [condition]')  ← 或英文
```

**通用测试要素**：
1. **Arrange**：准备测试数据和环境
2. **Act**：执行被测函数
3. **Assert**：验证结果

```
// 伪代码示例
test('formatDate should format ISO string to YYYY-MM-DD'):
  input = '2024-01-15T08:30:00Z'
  result = formatDate(input)
  assert result == '2024-01-15'

test('formatDate should handle null input'):
  result = formatDate(null)
  assert result == '--'

test('formatDate should throw on invalid input'):
  assert throws: formatDate('invalid')
```

## 服务层测试规范

```
test('userService.list should pass pagination params'):
  // Arrange
  mockHttpClient.get returns { items: [], total: 0 }

  // Act
  userService.list({ page: 2, limit: 10 })

  // Assert
  verify mockHttpClient.get called with ('/users', { page: 2, limit: 10 })
```

## E2E 测试规范

使用 Playwright 或类似工具：

```
test('用户可以登录并跳转到首页'):
  page.goto('/login')
  page.fill('[data-testid="email-input"]', 'test@example.com')
  page.fill('[data-testid="password-input"]', 'password123')
  page.click('[data-testid="login-button"]')

  assert page.url == '/'
  assert page.isVisible('[data-testid="user-menu"]')
```

## Mock 规范

| 原则 | 说明 |
|------|------|
| Mock 最小化 | 只 Mock 外部依赖（HTTP、数据库、文件系统） |
| 不 Mock 被测单元 | 被测函数/类本身不 Mock |
| Mock 行为不实现 | Mock 返回值，不要在 Mock 里写业务逻辑 |
| 集成测试优先网络拦截 | 使用 MSW/WireMock 等工具拦截 HTTP |

## 测试框架映射

| 语言 | 单元测试 | 集成测试 | E2E 测试 |
|------|----------|----------|----------|
| TypeScript | Vitest / Jest | Vitest + MSW | Playwright |
| Python | pytest | pytest + httpx | Playwright |
| Go | testing 包 | httptest | Playwright |
| Rust | cargo test + rstest | reqwest::test | Playwright |
| Java | JUnit 5 + Mockito | Testcontainers | Playwright |

## 测试质量检查清单

- [ ] 测试覆盖率 ≥ 80%
- [ ] 核心业务逻辑覆盖率 ≥ 90%
- [ ] 无跳过的测试（skip/only）
- [ ] Mock 最小化，不 mock 被测单元本身
- [ ] 测试命名清晰描述预期行为
- [ ] 边界条件有覆盖（空值、异常、极限值）
- [ ] 测试独立运行无依赖
- [ ] 异步操作正确等待
