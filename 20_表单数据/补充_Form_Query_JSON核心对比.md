# Form、Query 与 JSON Body 的核心区别

> 来源：20-21_表单数据与表单模型.md 中后续问答。

## 对话回合 38｜原提问

> 我累了。你就用简洁的语言告诉我表单数据跟前面的query等等其他参数的区别在哪里

## 对话回合 38｜学习内容

你只需要记住一点：**Query、Form、JSON Body 等参数的主要区别，就是数据放在 HTTP 请求的什么位置，以及采用什么格式。**

假设我们都要传递 `name = alice`：

| 参数类型 | 数据放在哪里 | 示例 |
|---|---|---|
| **Path** | URL 路径中 | `/users/alice` |
| **Query** | URL 问号 `?` 后面 | `/users?name=alice` |
| **Header** | HTTP 请求头中 | `X-Name: alice` |
| **Cookie** | 请求头中的 Cookie 里 | `Cookie: name=alice` |
| **JSON Body** | HTTP 请求体中，使用 JSON 格式 | `{"name":"alice"}` |
| **Form** | HTTP 请求体中，使用表单编码 | `name=alice` |

### 最容易混淆的是 Form、Query 和 JSON

- **Query**：数据放在 URL 里。
- **Form**：数据放在请求体里，通常写成 `name=alice&age=20`。
- **JSON Body**：数据也放在请求体里，但写成 `{"name":"alice","age":20}`。

对应 FastAPI：

```python
name: Annotated[str, Query()]  # 从 URL 读取
name: Annotated[str, Form()]   # 从表单请求体读取
```

而 JSON 通常使用：

```python
class User(BaseModel):
    name: str
```

**使用场景：**搜索、筛选常用 Query；登录表单、传统网页提交常用 Form；复杂的 Agent 消息和配置常用 JSON Body。

**一句话总结：Form 和 JSON Body 的位置相同，都是请求体；区别只是数据格式不同。**

今天先记住这个就够了。🌙
