# 补充答疑：Field 与相关跨章节错题

> 原讲解的补充问答，涉及 Field、路由顺序、Query 别名等，不属于单一官方章节的正文。

## 对话回合 26｜原提问

> 11,17,26，36，49.请你重点讲解这几道题，然后为我讲解Field函数的相关内容

## 对话回合 26｜学习内容

# 📘 FastAPI 错题专项讲解：第 11、17、26、36、49 题 + Field() 系统学习

这五道题值得重点复习，因为它们分别对应五种不同的知识：

| 题号 | 正确答案 | 重点知识 | 所属章节 |
|---|---|---|---|
| 11 | C | 路由匹配顺序 | 第 1 章 |
| 17 | A | HTTP 请求方法与 405 | FastAPI 基础 |
| 26 | A | `Query(alias=...)` 参数别名 | 第 4 章 |
| 36 | B | `Field(default_factory=list)` | 第 6、8 章 |
| 49 | A | 字符串长度校验与空白字符 | 第 8 章 |

其中，第 36 题和第 49 题都直接涉及 `Field()`，所以后面会专门把这个函数讲透。

---

## 一、第 11 题：为什么路由的定义顺序很重要？

**原题：** 同时存在 `/users/me` 和 `/users/{user_id}`，应该怎样注册？

正确答案：**C. 固定的 `/users/me` 放在动态路由前面。**

### 1. 先看错误的代码

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}


@app.get("/users/me")
def get_me():
    return {"user": "current user"}
```

假设用户访问：

```text
GET /users/me
```

你可能认为 FastAPI 会执行：

```python
get_me()
```

但实际上，FastAPI 会按注册顺序寻找匹配的路由。

它首先遇到：

```python
@app.get("/users/{user_id}")
```

由于 `{user_id}` 可以匹配一个动态路径段，因此 `/users/me` 也能匹配它。

接下来尝试把：

```python
user_id = "me"
```

转换为：

```python
user_id: int
```

但 `"me"` 不是整数。

因此，**请求通常返回 HTTP 422**。

重点是：FastAPI 不会因为这个参数校验失败，就自动尝试后面定义的 `/users/me` 接口。

### 2. 正确写法

```python
@app.get("/users/me")
def get_me():
    return {"user": "current user"}


@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

现在：

| 请求 | 匹配的接口 | 结果 |
|---|---|---|
| `/users/me` | `/users/me` | 200 |
| `/users/123` | `/users/{user_id}` | 200 |
| `/users/abc` | `/users/{user_id}` | 422 |

这里要记住一个原则：

**先定义具体的固定路由，再定义范围更广的动态路由。** 这也是 FastAPI 官方教程明确说明的规则。

### 🤖 在 Agent 开发中有什么作用？

假设你的 Agent 系统提供：

```text
GET /agents/me
GET /agents/{agent_id}
```

其中：

- `/agents/me`：查看当前用户默认使用的 Agent。
- `/agents/{agent_id}`：根据 ID 查询指定 Agent。

如果动态路由写在前面，`me` 就可能被当作 Agent ID。

所以设计 Agent 路由时，同样要考虑路径的匹配顺序。

---

## 二、第 17 题：为什么浏览器访问 POST 接口会返回 405？

**原题：** 浏览器地址栏直接访问只定义了 POST 的 `/chat` 接口，通常是什么情况？

正确答案：**A. 浏览器一般发送 GET，请求可能得到 405。**

你之前已经遇到过类似问题：启动 FastAPI 服务后，直接在浏览器地址栏访问接口，却发现不能正常执行。

问题的关键不在 FastAPI，而在 **HTTP 请求方法**。

### 1. GET 和 POST 不一样

先看代码：

```python
@app.post("/chat")
def chat():
    return {"answer": "你好"}
```

这段代码的意思是：

> 当客户端通过 POST 方法请求 `/chat` 时，执行 chat 函数。

注意：

```python
@app.post(...)
```

不是：

```python
@app.get(...)
```

但你在浏览器地址栏输入：

```text
http://127.0.0.1:8000/chat
```

浏览器通常会发送：

```http
GET /chat HTTP/1.1
```

它不会仅仅因为服务器定义了 POST 接口，就自动改用 POST。

所以出现了冲突：

```text
浏览器发送：GET /chat

服务器支持：POST /chat
```

### 2. 为什么是 405，而不是 404？

下面是你需要记住的四种状态：

| HTTP 状态码 | 含义 | 示例 |
|---|---|---|
| **200** | 请求成功 | POST /chat，数据正确 |
| **404** | 没有找到对应路径或业务资源 | GET /unknown |
| **405** | 路径存在，但不支持当前 HTTP 方法 | GET /chat，而接口只支持 POST |
| **422** | 请求参数或请求体校验失败 | POST /chat，缺少必填 message |

例如：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class ChatRequest(BaseModel):
    message: str


@app.post("/chat")
def chat(request: ChatRequest):
    return {"answer": request.message}
```

四种请求：

| 请求 | 结果 |
|---|---|
| GET `/chat` | 405 |
| GET `/unknown` | 404 |
| POST `/chat`，Body 为 `{}` | 422 |
| POST `/chat`，Body 为 `{"message":"你好"}` | 200 |

注意：这里假设服务器没有额外定义其他路由或自定义异常处理。

### 3. 如何正确测试 POST？

方法很简单：

1. 运行 `fastapi dev main.py`。
2. 浏览器打开 `http://127.0.0.1:8000/docs`。
3. 找到 `POST /chat`。
4. 点击 **Try it out**。
5. 填写请求体并点击 **Execute**。

`/docs` 会替你构造真正的 POST 请求。

### 🤖 与 Agent 客户端有什么关系？

真实的 Agent 客户端通常会执行类似：

```javascript
fetch("/chat", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        message: "你好 Agent"
    })
});
```

这里的：

```javascript
method: "POST"
```

就明确指定了 HTTP 方法。

**用户只负责在聊天框输入问题，前端负责用正确的 HTTP 方法和 JSON 格式调用 FastAPI。**

所以你需要理解 405，但不需要让最终用户自己处理 POST 和 JSON。

---

## 三、第 26 题：Query(alias="item-query") 到底是什么意思？

**原题：** 使用 `Query(alias="item-query")`，客户端应该通过哪个参数名传值？

正确答案：**A. item-query。**

这里涉及一个 Python 初学者很容易混淆的概念：**内部变量名与对外参数名可以不同。**

### 1. 为什么需要 alias？

Python 变量名不能直接包含减号：

```python
item-query = "python"
```

这不是合法的变量赋值语法，因为 Python 会把 `-` 识别为减法运算符。

但 HTTP URL 中可以有：

```text
/items/?item-query=python
```

那么，如何把外部的 `item-query` 映射到 Python 变量 `q` 呢？

答案就是 `alias`。

### 2. 官网代码思路

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
def read_items(
    q: Annotated[
        str | None,
        Query(alias="item-query")
    ] = None
):
    return {"q": q}
```

最重要的一行：

```python
q: Annotated[
    str | None,
    Query(alias="item-query")
] = None
```

拆成两部分：

- `q`：Python 函数内部的参数名称。
- `"item-query"`：客户端在 URL 中应该使用的参数名称。

因此访问：

```text
GET /items/?item-query=python
```

FastAPI 会执行类似：

```python
q = "python"
```

最终返回：

```json
{
  "q": "python"
}
```

### 3. 如果传 `?q=python` 呢？

在上面的代码中，已经声明：

```python
alias="item-query"
```

所以 FastAPI 会按照 `item-query` 这个名称提取数据。

如果客户端仅传：

```text
/items/?q=python
```

通常不会把它当作 `q` 参数的有效输入；由于本例的 q 可省略，函数使用默认的 `None`。

结果为：

```json
{
  "q": null
}
```

你不能认为 `alias` 会自动同时接受两个名称。

FastAPI 官方教程也说明，`alias` 会用于从 URL 中提取参数值，并体现在自动生成的文档中。

### 🤖 Agent 开发实例

假设前端设计的搜索参数叫：

```text
agent-query
```

但你希望后端 Python 变量叫：

```python
keyword
```

就可以写：

```python
@app.get("/agents/")
def search_agents(
    keyword: Annotated[
        str | None,
        Query(alias="agent-query")
    ] = None
):
    return {"keyword": keyword}
```

用户请求：

```text
GET /agents/?agent-query=code
```

后端得到：

```python
keyword = "code"
```

**记忆：alias 是给外部客户端看的名字，Python 内部仍然用你定义的参数名。**

---

## 四、第 36 题：Field(default_factory=list) 到底在干什么？

**原题：** `Field(default_factory=list)` 主要表示什么？

正确答案：**B. 需要默认值时创建一个空列表。**

这道题涉及两个知识：

1. Pydantic 的 `Field()`。
2. Python 的列表和函数对象。

我们先看最普通的写法。

### 1. 默认空列表

```python
from pydantic import BaseModel


class AgentFilter(BaseModel):
    tags: list[str] = []
```

如果没有提供 tags：

```python
filter1 = AgentFilter()
```

那么：

```python
filter1.tags
```

就是：

```python
[]
```

这在 Pydantic v2 中是可以使用的。

但还有一种更明确的写法：

```python
from pydantic import BaseModel, Field


class AgentFilter(BaseModel):
    tags: list[str] = Field(default_factory=list)
```

它同样表示：

> 用户没有提供 tags 时，为该模型创建一个新的空列表。

### 2. 为什么 `list` 后面没有括号？

这是关键。

你平时写：

```python
list()
```

意思是：

**立即调用 list 函数，创建一个空列表。**

执行：

```python
print(list())
```

得到：

```python
[]
```

而：

```python
default_factory=list
```

这里的 `list` 并没有被立即调用。

它的意思是：

> 把 `list` 这个可调用对象交给 Pydantic，需要默认值时再调用它。

可以把它理解成：

```python
def create_default_tags():
    return []


class AgentFilter(BaseModel):
    tags: list[str] = Field(
        default_factory=create_default_tags
    )
```

两种方式表达相同的思路。

### 3. 为什么要为每个模型创建独立的列表？

看下面的例子：

```python
class AgentFilter(BaseModel):
    tags: list[str] = Field(default_factory=list)


a = AgentFilter()
b = AgentFilter()

a.tags.append("python")

print(a.tags)
print(b.tags)
```

结果：

```text
['python']
[]
```

因为 a 和 b 的 tags 分别拥有自己的列表。

  **为什么默认列表要独立？**

    

        **AgentFilter A**

        `tags = ["python"]`

向 A 添加了 python

    

      

        **AgentFilter B**

        `tags = []`

B 的列表不会受到影响

    

如果不同请求错误地共享同一个可变列表，就可能出现一个用户添加的标签影响另一用户的情况。

### 4. 一个重要的技术细节

你可能听过 Python 的建议：

> 不要使用 `[]` 作为函数参数默认值。

比如普通 Python：

```python
def add_tag(tag, tags=[]):
    tags.append(tag)
    return tags
```

这种代码确实存在共享默认列表的问题。

但需要区分：**Pydantic v2 的 BaseModel 不完全遵循普通函数默认参数的行为。**

对于默认列表这类不可哈希的可变默认值，Pydantic 会为模型实例创建深拷贝，因此：

```python
tags: list[str] = []
```

在 Pydantic v2 中不会简单地把同一个列表共享给所有实例。

那么为什么还推荐：

```python
Field(default_factory=list)
```

因为它能够更明确地表达：

**“我想在需要默认值的时候，为每个模型创建一个独立列表。”**

而且，这种写法在很多需要动态默认值的场景中都很实用。

### 🤖 Agent 中有哪些列表适合这样写？

```python
class AgentRequest(BaseModel):
    tags: list[str] = Field(default_factory=list)
    tools: list[str] = Field(default_factory=list)
```

例如：

```json
{
  "tags": ["python", "fastapi"],
  "tools": ["calculator", "search"]
}
```

这里的 tags 和 tools 都是独立列表。

要注意：客户端传入 `"tools": ["search"]` 不代表服务端应该直接授权使用搜索工具。真实工具权限仍应由后端检查。

---

## 五、第 49 题：为什么三个空格也能通过 min_length=1？

**原题：**

```python
message: str = Field(min_length=1)
```

如果用户输入：

```python
"   "
```

会怎样？

正确答案：**A. 通常能够通过长度校验。**

这道题最容易让人产生误解：

> 字符串明明什么内容都没有，为什么会被允许？

因为**“字符串长度不为 0”与“字符串具有有效内容”不是一回事**。

### 1. Python 如何计算字符串长度？

Python 有一个内置函数：

```python
len()
```

它用于计算字符串长度。

例如：

```python
print(len("hello"))  # 5
print(len(""))       # 0
print(len("   "))    # 3
print(len("a b"))    # 3
```

注意：

```python
"   "
```

虽然看起来是空的，但实际上包含三个空格字符。

因此：

```python
len("   ") == 3
```

### 2. Field(min_length=1) 检查的是什么？

```python
message: str = Field(min_length=1)
```

检查的是字符串长度：

\[
\operatorname{len}(message) \geq 1
\]

它并没有额外规定“必须包含非空白字符”。

| 用户输入 | 字符串长度 | 是否通过 min_length=1 |
|---|---:|---|
| `""` | 0 | 否 |
| `" "` | 1 | 是 |
| `"   "` | 3 | 是 |
| `"a"` | 1 | 是 |
| `"你好"` | 2 | 是 |

所以原题的 `"   "` 会通过最小长度校验。

### 3. 如何拒绝只有空格的 Agent 消息？

这里需要补充一个非常有用的 Python 方法：

```python
strip()
```

它可以去掉字符串**首尾的空白字符**，包括常见空格、换行、制表符等。

例如：

```python
"  hello  ".strip()
# "hello"

"   ".strip()
# ""

"\n 你好 \n".strip()
# "你好"
```

所以判断用户输入是否只有空白字符，可以这样写：

```python
message = "   "

if not message.strip():
    print("用户没有输入有效内容")
```

这里：

```python
message.strip()
```

得到空字符串：

```python
""
```

而：

```python
not ""
```

在 Python 中为 `True`。

因此可以判定消息无有效内容。

### 4. 如何写进 Pydantic 模型？

使用 Pydantic 的 `field_validator`：

```python
from pydantic import BaseModel, Field, field_validator


class ChatRequest(BaseModel):

    message: str = Field(
        min_length=1,
        max_length=2000
    )

    @field_validator("message")
    @classmethod
    def validate_message(cls, value: str) -> str:

        cleaned = value.strip()

        if not cleaned:
            raise ValueError("消息不能只包含空格")

        return cleaned
```

先认识三个新知识：

- `@field_validator("message")`：为 message 字段添加自定义验证。
- `@classmethod`：声明一个类方法；这是 Pydantic 验证器常用的写法。
- `raise ValueError(...)`：明确指出这个值不合法。

当用户提交：

```json
{
  "message": "   "
}
```

执行流程是：

```text
收到 message = "   "
        ↓
Field(min_length=1) 检查长度
        ↓
长度为 3，基础校验通过
        ↓
执行 validate_message()
        ↓
strip() 得到 ""
        ↓
发现没有有效内容
        ↓
抛出 ValueError
        ↓
FastAPI 返回 HTTP 422
```

对于：

```json
{
  "message": "  你好 Agent  "
}
```

验证器返回清理后的：

```python
"你好 Agent"
```

注意要在验证器最后返回值，否则后面的业务代码就可能拿不到预期数据。这也是 Pydantic 官方验证器文档特别强调的地方。

**对 Agent 而言，这个校验很有实际价值：它能避免把纯空白消息发送到 LLM，从而浪费请求和计算资源。**

---

---

# 六、系统讲解 Pydantic 的 `Field()` 函数

现在我们正式进入重点：**你要真正理解 `Field()` 是什么、怎么写，以及为什么 Agent 项目会经常用到它。**

官方教程：[FastAPI：请求体 - 字段](https://fastapi.tiangolo.com/zh/tutorial/body-fields/)

Pydantic 官方文档：[Pydantic：Fields](https://docs.pydantic.dev/latest/concepts/fields/)

## 1. Field() 本质上是什么？

先回忆最简单的 Pydantic 模型：

```python
from pydantic import BaseModel


class ChatRequest(BaseModel):
    message: str
    max_steps: int = 3
```

它规定：

- `message` 必须是字符串。
- `max_steps` 必须是整数，默认值是 3。

但它没有规定：

- message 最多多长？
- max_steps 最小和最大是多少？
- 这些字段在 API 文档中应该怎么解释？

所以我们引入 `Field()`：

```python
from pydantic import BaseModel, Field


class ChatRequest(BaseModel):

    message: str = Field(
        min_length=1,
        max_length=2000
    )

    max_steps: int = Field(
        default=3,
        ge=1,
        le=5
    )
```

现在不仅能检查类型，还能检查每个字段的具体规则。

可以用下面的方式理解：

  

    **BaseModel**
    
规定整个请求的数据结构

  

  

    **Field()**

规定模型内部每个字段的：

      
> *省略对话原版动态控件。*

  

  

    **FastAPI 接收并验证 JSON**
    
合法数据进入业务代码，非法数据通常返回 422

注意一个重要区别：`Field()` 本身不是用来调用 Agent 的函数，它返回字段配置，**Pydantic 会在创建或验证模型时使用这些规则**。

## 2. Field() 最常用的参数

这张表建议你重点记忆：

| 参数 | 含义 | 示例 |
|---|---|---|
| `default` | 默认值 | `default=3` |
| `default_factory` | 动态生成默认值 | `default_factory=list` |
| `min_length` | 最小长度 | `min_length=1` |
| `max_length` | 最大长度 | `max_length=2000` |
| `gt` | 严格大于 | `gt=0` |
| `ge` | 大于等于 | `ge=1` |
| `lt` | 严格小于 | `lt=10` |
| `le` | 小于等于 | `le=10` |
| `pattern` | 正则格式约束 | `pattern=r"^agent-"` |
| `title` | 字段标题 | `title="User Message"` |
| `description` | 字段说明 | `description="用户输入的问题"` |
| `examples` | 文档中的示例值 | `examples=["你好 Agent"]` |
| `alias` | 字段对外使用的名称 | `alias="userMessage"` |

这里，**前八章最值得优先掌握的是前八个参数，以及 title、description**。

## 3. Field(default=...)：默认值与必填字段

这是容易出错的第一组规则。

```python
class ChatRequest(BaseModel):
    message: str = Field(min_length=1)
    max_steps: int = Field(default=3, ge=1, le=5)
```

为什么 message 必填，而 max_steps 可以省略？

因为：

```python
message: str = Field(min_length=1)
```

没有声明默认值。

而：

```python
max_steps: int = Field(default=3, ge=1, le=5)
```

明确指定了默认值 3。

所以只提交：

```json
{
  "message": "你好"
}
```

完全合法。

Pydantic 会创建类似：

```python
ChatRequest(
    message="你好",
    max_steps=3
)
```

### 三种字段声明对比

| 写法 | 是否必填 |
|---|---|
| `name: str = Field(min_length=1)` | 是 |
| `name: str = Field(default="Tom")` | 否 |
| `name: str \| None = Field(default=None)` | 否 |

特别记住：**Field 的默认值控制能否省略字段；类型中的 `| None` 控制是否允许空值。**

另外，在 Pydantic v2 中，默认值通常不会自动进行与用户输入相同的验证；特殊情况下可以使用 `validate_default=True`。因此，自己写的默认值也应该确保满足约束。

## 4. Field 的数值校验

这和你第 5 章学过的 `Path()`、`Query()` 数值校验是一样的思路。

例如：

```python
class RunConfig(BaseModel):
    max_steps: int = Field(
        default=3,
        ge=1,
        le=5
    )

    temperature: float = Field(
        default=0.7,
        ge=0.0,
        le=1.0
    )
```

对应的约束是：

\[
1 \le max\_steps \le 5
\]

\[
0 \le temperature \le 1
\]

例如：

```json
{
  "max_steps": 3,
  "temperature": 0.7
}
```

合法。

但是：

```json
{
  "max_steps": 10,
  "temperature": 1.5
}
```

会返回 422。

这里的 `temperature` 范围只是我们为教学示例设定的范围。真实项目应当根据你所使用的模型服务支持的参数范围来设计。

## 5. Field 的字符串校验

例如：

```python
class ChatRequest(BaseModel):
    message: str = Field(
        min_length=1,
        max_length=2000
    )
```

它表示：

\[
1 \le \operatorname{len}(message) \le 2000
\]

你现在需要记住：

**min_length 和 max_length 检查长度，但不自动判断字符串的语义是否有效。**

因此：

```python
"   "
```

能通过最小长度为 1 的验证。

如果你希望拒绝纯空白消息，就要像第 49 题那样增加 `field_validator`。

如果想限制字符串的形式，还可以使用 `pattern`：

```python
class AgentInfo(BaseModel):
    agent_code: str = Field(
        pattern=r"^agent-[0-9]+$"
    )
```

示例：

```text
agent-123   合法
agent-456   合法
hello       不合法
agent-abc   不合法
```

这个正则表达式表示：字符串以 `agent-` 开头，后面至少跟一个数字，并以数字结尾。

## 6. Field(default_factory=...)：动态默认值

现在回到第 36 题。

```python
class AgentRequest(BaseModel):
    tags: list[str] = Field(default_factory=list)
```

最应该记住的是：

```python
default_factory=list
```

而不是：

```python
default_factory=list()
```

区别：

- `list`：把可调用对象交给 Pydantic。
- `list()`：立即调用并得到一个列表，不是这里需要的工厂函数。

除了列表，还能用于其他动态默认值，但你现在先把 `default_factory=list` 掌握就够了。

## 7. Field 的文档信息：title、description、examples

例如：

```python
class ChatRequest(BaseModel):

    message: str = Field(
        min_length=1,
        max_length=2000,
        title="User Message",
        description="用户发送给 Agent 的问题",
        examples=["请帮我解释 FastAPI"]
    )
```

这一段里面有两种不同的功能：

**真正的验证规则：**

```python
min_length=1
max_length=2000
```

**面向开发者的文档信息：**

```python
title="User Message"
description="用户发送给 Agent 的问题"
examples=["请帮我解释 FastAPI"]
```

后面这些内容会用于 JSON Schema 和 FastAPI 的 OpenAPI 文档，但不会让模型自动理解用户问题或生成回答。

打开 `/docs` 时，你就可以看到更清晰的字段说明。

---

## 七、Field() 与 Query()、Path()、Body() 的关系

这里我想特别强调一个你之后一定会用到的概念。

| 工具 | 主要职责 |
|---|---|
| `Path()` | HTTP 路径参数的声明和校验 |
| `Query()` | HTTP 查询参数的声明和校验 |
| `Body()` | HTTP 请求体参数的声明和校验 |
| `Field()` | Pydantic 模型内部字段的声明和校验 |

例如：

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query
from pydantic import BaseModel, Field

app = FastAPI()


class ChatRequest(BaseModel):
    message: str = Field(min_length=1)
    max_steps: int = Field(default=3, ge=1, le=5)


@app.post("/agents/{agent_id}/chat")
def chat(
    agent_id: Annotated[int, Path(ge=1)],
    request: ChatRequest,
    debug: Annotated[bool, Query()] = False
):
    return {
        "agent_id": agent_id,
        "message": request.message,
        "max_steps": request.max_steps,
        "debug": debug
    }
```

请求：

```text
POST /agents/2/chat?debug=true
```

Body：

```json
{
  "message": "请解释 FastAPI",
  "max_steps": 3
}
```

FastAPI 的分工：

- `Path(ge=1)`：检查 Agent ID。
- `Query()`：从 URL 获取 debug。
- `request: ChatRequest`：从 JSON Body 获取数据。
- `Field(min_length=1)`：检查 message。
- `Field(ge=1, le=5)`：检查 max_steps。

**Field 不负责决定模型来自 URL 还是 Body。** 例如第六章的查询参数模型同样可以使用 `Field()`；它的数据来源则由外面的 `Query()` 决定。

---

# 八、综合练习：用一个 Agent API 巩固这五道错题

这一部分把五个知识点整合在一个可以运行的项目里。

新建文件：

`field_agent_practice.py`

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query, HTTPException
from pydantic import BaseModel, Field, field_validator


app = FastAPI(title="Agent Field Practice")

AGENTS = {
    1: "Chat Agent",
    2: "Code Agent"
}


# ===== 请求体模型 =====

class ChatRequest(BaseModel):

    message: str = Field(
        min_length=1,
        max_length=2000,
        description="用户输入的 Agent 问题"
    )

    max_steps: int = Field(
        default=3,
        ge=1,
        le=5
    )

    tags: list[str] = Field(
        default_factory=list
    )

    session_id: str | None = Field(
        default=None
    )

    @field_validator("message")
    @classmethod
    def validate_message(cls, value: str) -> str:

        cleaned = value.strip()

        if not cleaned:
            raise ValueError("消息不能只包含空格")

        return cleaned


# ===== 响应模型 =====

class ChatResponse(BaseModel):
    agent_id: int
    agent_name: str
    answer: str
    executed_steps: list[str]
    tags: list[str]
    session_id: str | None = None


# ===== 第 11 题：固定路由必须先注册 =====

@app.get("/agents/me")
def get_my_agent():
    return {"agent": "current"}


@app.get("/agents/{agent_id}")
def get_agent(
    agent_id: Annotated[int, Path(ge=1)]
):
    if agent_id not in AGENTS:
        raise HTTPException(
            status_code=404,
            detail="Agent not found"
        )

    return {
        "agent_id": agent_id,
        "name": AGENTS[agent_id]
    }


# ===== 第 17、26、36、49 题综合应用 =====

@app.post(
    "/agents/{agent_id}/chat",
    response_model=ChatResponse
)
def chat(
    agent_id: Annotated[int, Path(ge=1)],

    request: ChatRequest,

    search_key: Annotated[
        str | None,
        Query(
            alias="agent-query",
            min_length=2
        )
    ] = None
):
    if agent_id not in AGENTS:
        raise HTTPException(
            status_code=404,
            detail="Agent not found"
        )

    planned_steps = [
        "理解问题",
        "模拟查询工具",
        "整理结果",
        "输出回答",
        "检查答案"
    ]

    steps = planned_steps[:request.max_steps]

    answer = (
        f"{AGENTS[agent_id]} 收到了：{request.message}"
    )

    if search_key:
        answer += f"；筛选词：{search_key}"

    return ChatResponse(
        agent_id=agent_id,
        agent_name=AGENTS[agent_id],
        answer=answer,
        executed_steps=steps,
        tags=request.tags,
        session_id=request.session_id
    )
```

### 运行

```bash
fastapi dev field_agent_practice.py
```

在本机打开：

```text
http://127.0.0.1:8000/docs
```

### 测试正常请求

URL：

```text
POST /agents/2/chat?agent-query=python
```

Body：

```json
{
  "message": "   请解释 Field()   ",
  "max_steps": 2,
  "tags": ["fastapi", "agent"],
  "session_id": "test001"
}
```

预期响应：

```json
{
  "agent_id": 2,
  "agent_name": "Code Agent",
  "answer": "Code Agent 收到了：请解释 Field()；筛选词：python",
  "executed_steps": [
    "理解问题",
    "模拟查询工具"
  ],
  "tags": ["fastapi", "agent"],
  "session_id": "test001"
}
```

请注意，这里实际演示了：

- 第 11 题：固定路由先注册。
- 第 17 题：必须使用 POST 方法调用聊天接口。
- 第 26 题：URL 参数叫 `agent-query`，Python 参数叫 `search_key`。
- 第 36 题：tags 不提供时，默认生成空列表。
- 第 49 题：message 会通过验证器去掉首尾空白，并拒绝纯空白输入。

另外，`max_steps` 在此只控制模拟步骤列表的长度，没有执行真正的 LangGraph 工作流。

### 再测试五种情况

| 测试请求 | 预期结果 | 对应知识 |
|---|---|---|
| GET `/agents/me` | 200 | 固定路由优先 |
| GET `/agents/2/chat` | 405 | HTTP 方法不匹配 |
| POST `/agents/2/chat?agent-query=x`，Body 合法 | 422 | Query 的长度限制 |
| POST `/agents/2/chat`，Body 中省略 tags | 200，tags 为 `[]` | default_factory |
| POST `/agents/2/chat`，message 为 `"   "` | 422 | field_validator |

---

## 九、你现在最需要记住的六句话

1. **路由顺序**：固定路径通常放在可能与之冲突的动态路径前面。
2. **405**：URL 路径存在，但当前 HTTP 方法不被支持。
3. **alias**：客户端使用别名传参，Python 函数内部使用原变量名。
4. **default_factory=list**：需要默认列表时，由 Pydantic 生成一个新列表。
5. **min_length=1**：只要求至少一个字符，不保证字符串包含有效内容。
6. **Field()**：为 Pydantic 模型内部字段设置默认值、校验规则和文档信息。

对你接下来学习 Agent 开发，我最希望你掌握的是：

**FastAPI 的 Path、Query、Body 和 Pydantic 的 Field、field_validator 共同构成了 Agent 的输入验证层。它们负责让用户输入变得规范、可靠；真正执行 LLM、工具调用和 LangGraph 工作流的，是后面的 Agent 业务逻辑。**

你这五道错题并不是彼此孤立的知识点。把它们全部掌握之后，你设计 Agent API 时，对请求从哪里来、为什么报错、数据怎样进入执行器的理解就会清晰很多。💪
