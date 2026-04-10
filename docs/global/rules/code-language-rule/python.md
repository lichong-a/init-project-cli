# Python 规则

> 当项目选用 Python 时，Agent 应遵循以下规则。

## 语言特性

- **类型注解**：所有公共函数必须有参数和返回类型注解
- **包管理**：优先使用 `uv`，备选 `pip` + `venv`
- **代码格式**：`ruff format`（替代 black），`ruff check`（替代 flake8）
- **Python 版本**：最低 3.10+（支持 `match/case`、`X | Y` 类型联合语法）

## 类型系统

```python
from typing import Optional

# 公共函数显式标注
def get_user_by_id(user_id: str) -> Optional[User]:
    ...

# 使用现代类型语法（3.10+）
def process(items: list[str]) -> dict[str, int]:
    ...

# 使用 Pydantic 做运行时验证 + 类型定义
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    name: str
    age: int = Field(ge=0, le=150)
```

## 项目结构

```
src/
├── <package>/
│   ├── __init__.py
│   ├── models/        # 数据模型
│   ├── services/      # 业务逻辑
│   ├── repositories/  # 数据访问
│   ├── api/           # API 路由（FastAPI）
│   └── utils/         # 工具函数
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── pyproject.toml     # 项目配置
└── ruff.toml          # Lint 配置（可选合并到 pyproject.toml）
```

## 不可变性

```python
# 使用 frozen dataclass
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    id: str
    name: str

# 使用 tuple 替代可变列表（只读场景）
# 使用 copy / deepcopy 创建副本
```

## 框架选择

| 框架 | 适用场景 |
|------|----------|
| FastAPI | Web API（推荐） |
| Django | 全栈 Web 应用 |
| Flask | 轻量 Web 服务 |
| SQLAlchemy | ORM（配合 FastAPI） |
| Pydantic | 数据验证 + 序列化 |

## 测试规范

```python
# 使用 pytest
def test_format_date():
    result = format_date("2024-01-15T08:30:00Z")
    assert result == "2024-01-15"

# 参数化测试
@pytest.mark.parametrize("input,expected", [
    ("active", True),
    ("inactive", False),
])
def test_is_active(input, expected):
    assert is_active(input) == expected
```

## 禁止清单

| 禁止项 | 替代方案 |
|--------|----------|
| `print()` 调试 | `logging` 模块 |
| `*args, **kwargs` 滥用 | 显式参数 |
| 可变默认参数 | `None` + 函数内初始化 |
| 全局变量 | 依赖注入 |
| 裸 `except:` | 明确异常类型 |
