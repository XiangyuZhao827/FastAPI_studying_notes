# 补充答疑｜原课程追加讲解

> 来源：17-18_响应模型与更多模型.md；按主题归入本章作为补充。

## 对话回合 35｜原提问

> 详细讲解一下刚才自测的第44题

## 对话回合 35｜学习内容

# 📘 第 44 题详解：浏览器如何自动发送 Cookie？

你说的是之前那份 **50 道离线自测题**中的第 44 题，考察的是 **Cookie 与前端浏览器之间的关系**。

## 一、先回顾原题

**题目：**

网页前端调用 Agent API，浏览器已有一个满足发送条件的 `session_id` Cookie。下列哪种认识正确？

A. 必须通过 JavaScript 在 `headers` 中手工设置 Cookie 请求头。

B. 浏览器可根据域名、路径、SameSite 等策略自动携带 Cookie；前端要正确处理请求凭据配置。

C. Cookie 永远自动发送到任意站点。

D. Cookie 由 LangGraph 自动生成。

**正确答案：B。**

这道题最核心的一句话是：

> **Cookie 通常由浏览器负责保存和发送，但浏览器不会无条件发送 Cookie。**

你之前已经学过：

```python
session_id: Annotated[str | None, Cookie()] = None
```

这行代码解决的是：**FastAPI 如何读取客户端已经发来的 Cookie。**

但第 44 题追问的是更前面的环节：

**Cookie 到底是谁发过来的？它什么时候会被发送？前端程序需要做什么？**

我们把这三个问题逐一弄清楚。

---

## 二、Cookie 是怎么从服务器到浏览器，再回到服务器的？

这是理解第 44 题最重要的一步。

假设你开发了一个 Agent 聊天网站，用户第一次建立会话时，服务器希望浏览器保存一个会话标识。

整个过程通常分为三个阶段。

  

    **① 服务器告诉浏览器：保存 Cookie**

    ```http
    Set-Cookie: session_id=demo001; Path=/; HttpOnly; SameSite=Lax
    ```

这是 HTTP 响应中的 Set-Cookie。

  

  

    **② 浏览器保存 Cookie**

    `session_id = demo001`

浏览器记录 Cookie 的值、适用主机、路径和安全属性。

  

  

    **③ 浏览器再次请求 Agent API**

    ```http
    POST /agents/2/chat HTTP/1.1
    Cookie: session_id=demo001
    ```

如果符合发送条件，浏览器自动携带 Cookie。

  

  

    **④ FastAPI 提取 Cookie**

    ```python
    session_id: Annotated[str | None, Cookie()] = None
    ```

最终得到 Python 字符串 "demo001"。

这里要分清两个 HTTP 名称：

| 名称 | 出现的位置 | 作用 |
|---|---|---|
| `Set-Cookie` | 服务器的**响应头** | 告诉浏览器保存或更新 Cookie |
| `Cookie` | 浏览器的**请求头** | 将符合条件的 Cookie 发给服务器 |

因此，虽然 Cookie 在 HTTP 协议中也是通过请求头传输的，但在 FastAPI 里我们通常使用专门的 `Cookie()` 提取它。

---

## 三、为什么其他三个选项都错了？

### A：必须由 JavaScript 手动设置 Cookie 请求头 ❌

你之前学过前端使用 `fetch()`：

```javascript
fetch("/agents/2/chat", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        message: "你好"
    })
});
```

那么可不可以这样写？

```javascript
fetch("/agents/2/chat", {
    method: "POST",
    headers: {
        "Cookie": "session_id=demo001"
    }
});
```

**不可以依赖这种写法。**

因为浏览器将 `Cookie` 视为受限制的请求头，网页 JavaScript 不能通过 `fetch()` 随意设置它。浏览器需要保留对 Cookie 发送行为的控制。

正确的思路是：

**先让浏览器合法地保存 Cookie，再由浏览器在符合条件的请求中自动携带它。**

但这里有一个区别：Python 的 `httpx`、命令行 `curl` 等非浏览器 HTTP 客户端可以显式设置 Cookie。

例如，你之前练习过：

```python
import httpx

response = httpx.get(
    "http://127.0.0.1:8000/demo",
    cookies={"session_id": "demo001"}
)
```

这个写法没有问题。

所以：

- 浏览器 JavaScript：不能随意手动设置 `Cookie` 请求头。
- Python `httpx`：可以通过 `cookies` 参数发送 Cookie。

这是两类运行环境的区别。

### C：Cookie 永远发送到任意网站 ❌

假设浏览器保存了属于：

```text
example.com
```

的 Cookie。

这并不意味着它访问：

```text
other-site.com
```

时也应该把这份 Cookie 发过去。

否则，不同网站就可能轻易获得其他站点的会话信息。

浏览器会根据 Cookie 的作用域与安全规则，判断是否发送它。

最常见的条件包括：

| 条件 | 作用 |
|---|---|
| `Domain` | 限制适用的主机或域名 |
| `Path` | 限制适用的 URL 路径 |
| `Secure` | 通常要求通过 HTTPS 发送 |
| `SameSite` | 控制跨站请求时的 Cookie 发送行为 |
| 有效期 | 过期或已删除的 Cookie 不再正常发送 |
| `fetch credentials` | 影响浏览器请求是否携带凭据 |

所以 C 的问题就在于“**永远**”和“**任意网站**”这两个词。

### D：Cookie 由 LangGraph 自动生成 ❌

LangGraph 是用于构建 Agent 工作流的框架。

Cookie 是浏览器与 HTTP 服务器之间的机制。

两者并不是同一个概念。

```text
Cookie
  ↓
FastAPI 读取会话标识
  ↓
服务器验证用户并确定有权访问的对话
  ↓
LangGraph 使用对应 thread_id 和持久化状态
```

LangGraph 不会仅仅因为你写了：

```python
Cookie()
```

就自动生成浏览器会话 Cookie，也不会自动完成登录认证。

---

# 四、重点：前端的 credentials 到底是什么？

第 44 题的 B 选项还有后半句：

> 前端要正确处理请求凭据配置。

这里说的就是 JavaScript `fetch()` 中的：

```javascript
credentials
```

它有三个常见值。

| 值 | 含义 |
|---|---|
| `"omit"` | 不携带凭据 |
| `"same-origin"` | 只对同源请求使用凭据；**默认值** |
| `"include"` | 允许跨源请求也携带凭据，但仍受 Cookie 和浏览器安全规则约束 |

这里的凭据包括 Cookie 等。`include` 并不代表绕过所有 Cookie 限制。

## 1. 同源请求：你目前最容易遇到的情况

假设网页地址是：

```text
http://127.0.0.1:8000
```

FastAPI 接口也是：

```text
http://127.0.0.1:8000/agents/2/chat
```

那么它们属于同源。

你可以这样调用：

```javascript
fetch("/agents/2/chat", {
    method: "POST",

    credentials: "same-origin",

    headers: {
        "Content-Type": "application/json"
    },

    body: JSON.stringify({
        message: "请解释 FastAPI"
    })
});
```

最重要的是：

```javascript
credentials: "same-origin"
```

表示：

> 对同源请求使用浏览器凭据，包括符合条件的 Cookie。

即使不写这行，现代浏览器 `fetch()` 的默认值通常也是 `"same-origin"`。

注意代码中**没有**：

```javascript
headers: {
    "Cookie": "session_id=demo001"
}
```

因为 Cookie 的发送由浏览器管理。

## 2. 跨源请求：前后端运行在不同端口

未来你可能使用 Vue 或 React 开发 Agent 前端。

例如：

```text
前端：http://localhost:5173
后端：http://localhost:8000
```

虽然都叫 localhost，但**端口不同，因此属于不同的 Origin（源）**。

这时前端可以写：

```javascript
fetch("http://localhost:8000/agents/2/chat", {
    method: "POST",

    credentials: "include",

    headers: {
        "Content-Type": "application/json"
    },

    body: JSON.stringify({
        message: "你好 Agent"
    })
});
```

这里：

```javascript
credentials: "include"
```

表示允许跨源请求使用浏览器凭据。

但还不够。

后端需要正确配置跨源资源共享，也就是 **CORS**。

例如：

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:5173"
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST"],
    allow_headers=["Content-Type"]
)
```

这里：

```python
allow_credentials=True
```

允许跨源请求携带凭据。

```python
allow_origins=[
    "http://localhost:5173"
]
```

指定允许的前端来源。

**当跨源请求需要使用凭据时，不能简单把允许来源写成 `"*"` 来代替明确的来源配置。** 这是 FastAPI 官方 CORS 教程强调的规则。

另外，虽然这里的两个 localhost 端口属于**不同源**，但它们仍可能属于**同站点**。Origin 与 Site 是两个不同概念，因此不要误认为一旦端口不同，就必然需要 `SameSite=None`。

跨站 Cookie 还要考虑 `SameSite`、`Secure` 和浏览器的第三方 Cookie 限制。

---

# 五、Cookie 的几个重要属性

你现在不需要把每一个属性都背下来，但应该知道它们有什么作用。

| 属性 | 作用 | 需要特别记住的点 |
|---|---|---|
| `Domain` | 限制 Cookie 的适用主机 | 不会随意发送给其他域名 |
| `Path` | 限制 Cookie 对应的 URL 路径 | 例如 `/agents` |
| `HttpOnly` | 禁止网页 JavaScript 直接读取 Cookie | **浏览器仍可自动发送 Cookie** |
| `Secure` | 通常只通过 HTTPS 发送 | 生产环境会话 Cookie 应重视 |
| `SameSite=Lax` | 限制部分跨站发送行为 | 常见的安全设置 |
| `SameSite=Strict` | 更严格的跨站限制 | 不适用于所有交互场景 |
| `SameSite=None` | 允许跨站发送 | 通常必须配合 `Secure` |

这些行为在 MDN 的 Cookie 文档中有完整说明。

### 一个容易考的细节：HttpOnly

假设服务器发送：

```http
Set-Cookie: session_id=demo001; HttpOnly; Path=/
```

浏览器保存成功。

那么，网页 JavaScript 通常不能通过：

```javascript
document.cookie
```

直接读出这份 `session_id`。

**但浏览器仍然可以在符合条件时自动把它作为 Cookie 发送给服务器。**

所以：

> HttpOnly 限制的是 JavaScript 直接访问 Cookie，而不是禁止浏览器发送 Cookie。

这是一个非常重要的安全知识点。

---

# 六、FastAPI 完整实战：亲眼看到浏览器自动发送 Cookie

现在我们写一个适合你当前水平的小项目。

目标是验证：

1. 服务器设置 Cookie。
2. 浏览器保存 Cookie。
3. 前端不手写 Cookie 请求头。
4. 浏览器再次请求时自动携带 Cookie。
5. FastAPI 使用 `Cookie()` 成功提取。

## 1. 创建文件 cookie_browser_practice.py

```python
from typing import Annotated

from fastapi import FastAPI, Cookie, Response
from pydantic import BaseModel

app = FastAPI()


class ChatRequest(BaseModel):
    message: str


# 第一步：服务器设置一个教学 Cookie
@app.get("/demo/set-cookie")
def set_demo_cookie(response: Response):

    response.set_cookie(
        key="session_id",
        value="demo001",
        httponly=True,
        samesite="lax",
        path="/",
        secure=False
    )

    return {
        "message": "教学 Cookie 已设置"
    }


# 第二步：Agent 接口读取浏览器发来的 Cookie
@app.post("/agents/2/chat")
def chat(
    request: ChatRequest,

    session_id: Annotated[
        str | None,
        Cookie()
    ] = None
):

    return {
        "answer": f"Agent 收到了：{request.message}",
        "cookie_received": session_id is not None
    }
```

特别提醒：这个例子中的 `demo001` 是固定的**教学标识**，没有认证能力。`secure=False` 也仅用于本地 HTTP 练习；真实登录会话不能照搬这种实现。

## 2. 第一步：启动 FastAPI

```bash
fastapi dev cookie_browser_practice.py
```

## 3. 第二步：让浏览器保存 Cookie

直接在浏览器打开：

```text
http://127.0.0.1:8000/demo/set-cookie
```

服务器会通过响应头告诉浏览器保存：

```http
Set-Cookie: session_id=demo001; HttpOnly; Path=/; SameSite=lax
```

你会看到：

```json
{
  "message": "教学 Cookie 已设置"
}
```

## 4. 第三步：通过浏览器开发者工具发送请求

继续停留在这个页面。

在 Chrome 或 Edge 中按：

```text
F12
```

找到：

```text
Console
```

在控制台中执行：

```javascript
fetch("/agents/2/chat", {
    method: "POST",

    credentials: "same-origin",

    headers: {
        "Content-Type": "application/json"
    },

    body: JSON.stringify({
        message: "你好 Agent"
    })
})
.then(response => response.json())
.then(data => console.log(data));
```

你会发现，这里根本没有手动设置：

```http
Cookie: session_id=demo001
```

但浏览器如果已经保存了 Cookie，并且符合发送条件，就会自动携带它。

预期输出：

```json
{
  "answer": "Agent 收到了：你好 Agent",
  "cookie_received": true
}
```

`cookie_received` 是 `true`，说明 FastAPI 确实收到对应 Cookie。

你还可以打开开发者工具的 **Network** 面板，找到这次 POST 请求，在 Request Headers 中检查是否存在：

```http
Cookie: session_id=demo001
```

即使 Cookie 设了 `HttpOnly`，它仍然可以被浏览器发送。

### 5. 第四步：不发送凭据再试一次

把代码中的：

```javascript
credentials: "same-origin"
```

改成：

```javascript
credentials: "omit"
```

再次发送相同的请求：

```javascript
fetch("/agents/2/chat", {
    method: "POST",

    credentials: "omit",

    headers: {
        "Content-Type": "application/json"
    },

    body: JSON.stringify({
        message: "你好 Agent"
    })
})
.then(response => response.json())
.then(console.log);
```

预期返回：

```json
{
  "answer": "Agent 收到了：你好 Agent",
  "cookie_received": false
}
```

因为：

```javascript
credentials: "omit"
```

阻止了这次 `fetch()` 请求携带浏览器凭据。

这两个实验可以帮助你真正理解：

**Cookie 即使已经保存到浏览器，也不意味着每个请求都会把它发送出去。**

---

# 七、结合 Agent 开发，应该怎样理解 session_id？

这道题还有一个和未来 LangGraph 开发直接相关的知识点。

即使 FastAPI 成功读到了：

```python
session_id = "demo001"
```

也只能说明：

> 请求中存在一份名为 session_id 的 Cookie，并且成功解析了它。

它**不代表**：

- 用户已经真实登录。
- session_id 一定由可信服务器签发。
- 这个用户有权访问当前 Agent。
- LangGraph 已经恢复了历史聊天。

未来更完整的流程应该是：

```text
浏览器自动携带 session_id Cookie
               ↓
FastAPI 使用 Cookie() 提取
               ↓
服务端验证会话真实有效
               ↓
确定当前用户身份
               ↓
检查用户是否有权访问目标对话
               ↓
查找对应的 LangGraph thread_id
               ↓
加载持久化状态并继续 Agent 对话
```

也就是说：

**Cookie 帮助传递会话信息，FastAPI 帮助解析它，而认证、授权和 LangGraph 会话恢复仍需要后端业务代码完成。**

---

## 八、最后用三道小题检查理解

<!-- 已省略对话中仅用于交互演示的状态代码 -->

<!-- 已省略对话中仅用于交互演示的状态代码 -->

  
> *省略对话原版动态控件。*

      

setAnswers(s=>({...s,[i]:{...s[i],checked:true}}))}>检查答案

      

    

  

## 九、第 44 题最终总结

请把这三句话作为本题的记忆重点：

1. **服务器通过 `Set-Cookie` 告诉浏览器保存 Cookie；浏览器在符合条件时通过 `Cookie` 请求头把它发送回来。**
2. **浏览器 JavaScript 不能随意手动设置 Cookie 请求头；`fetch()` 的 `credentials` 会影响凭据发送，同源与跨源的处理不同。**
3. **FastAPI 的 `Cookie()` 只负责读取和验证参数格式，不负责登录认证，也不负责自动恢复 LangGraph 历史状态。**

如果你把第 44 题彻底弄懂了，那么你之前学过的 `Cookie()`、Cookie 参数模型、HTTP Header、前端 `fetch()` 和 Agent 会话管理，就真正串起来了。💪
