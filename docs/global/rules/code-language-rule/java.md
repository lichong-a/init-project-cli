# Java 规则

> 当项目选用 Java 时，Agent 应遵循以下规则。

## 语言特性

- **版本**：Java 17+（LTS），推荐 21+
- **构建工具**：Maven 或 Gradle（Kotlin DSL）
- **代码格式**：`google-java-format` 或 `palantir-java-format`
- **框架**：Spring Boot 3.x+（Web 项目推荐）

## 项目结构

```
src/
├── main/
│   ├── java/com/<org>/<project>/
│   │   ├── controller/     # REST Controller
│   │   ├── service/        # 业务逻辑
│   │   ├── repository/     # 数据访问（JPA/MyBatis）
│   │   ├── model/
│   │   │   ├── entity/     # 数据库实体
│   │   │   ├── dto/        # 数据传输对象
│   │   │   └── request/    # 请求对象
│   │   ├── config/         # 配置类
│   │   ├── exception/      # 异常定义
│   │   └── util/           # 工具类
│   └── resources/
│       ├── application.yml
│       └── db/migration/   # 数据库迁移
└── test/
    ├── java/               # 单元测试
    └── resources/
```

## 错误处理

```java
// 自定义业务异常
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }
}

// 全局异常处理（Spring Boot）
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiError> handleBusiness(BusinessException ex) {
        return ResponseEntity.badRequest()
            .body(new ApiError(ex.getCode(), ex.getMessage()));
    }
}
```

## API 设计

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public ResponseEntity<UserResponse> create(
            @Valid @RequestBody CreateUserRequest request) {
        return ResponseEntity.ok(userService.create(request));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getById(@PathVariable String id) {
        return ResponseEntity.ok(userService.getById(id));
    }
}
```

## 测试规范

```java
// 使用 JUnit 5
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository repository;

    @InjectMocks
    private UserService service;

    @Test
    void shouldCreateUser() {
        // given
        var request = new CreateUserRequest("test@example.com", "Test");

        // when
        var result = service.create(request);

        // then
        assertThat(result.email()).isEqualTo("test@example.com");
        verify(repository).save(any());
    }
}
```

## 禁止清单

| 禁止项 | 替代方案 |
|--------|----------|
| `System.out.println` | SLF4J Logger |
| `null` 返回值 | `Optional<T>` |
| 可变 DTO | Record 或不可变对象 |
| `@Autowired` 字段注入 | 构造函数注入 |
| `e.printStackTrace()` | Logger |
