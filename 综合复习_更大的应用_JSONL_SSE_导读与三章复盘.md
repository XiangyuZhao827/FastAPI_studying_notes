# FastAPI 三章学习手册

**更大的应用 · 流式传输 JSON Lines · 服务器发送事件（SSE）**

学习方向：Agent 后端开发　｜　适合阶段：零基础完成 Depends 与安全性之后　｜　更新：2026-11-02

> 学习目标：理解每章解决什么问题、三者为什么不同、在 Agent 的实际产品中该怎么选。代码仅保留与 FastAPI 官方教程对应的核心形式；Agent 实战以思路为主。

## 快速导航：先建立整体认识

| 章节 | 问题 | 一句话记忆 | 典型 Agent 场景 |
| --- | --- | --- | --- |
| 更大的应用 | 接口越来越多如何组织？ | APIRouter 管理一组路径操作 | 用户、Agent、聊天、会话等分模块 |
| 流式 JSON Lines | 结果还没生成完，能否先发一部分？ | 每行一个独立 JSON 值 | 持续发送结构化步骤、日志 |
| SSE | 如何规范地推送持续更新的事件？ | 事件有 data/event/id 等字段 | AI 回复、工具调用与任务进度 |

**核心关系：** 第一章讨论代码如何组织；后两章讨论结果如何传输。SSE 是流式传输的一种具体方案，不与“流式输出”相对立。

---

# 三章统一复盘：如何在 Agent 项目中作选择

| 你希望做到的事情 | 更合适的能力 |
| --- | --- |
| 一份 main.py 写不下几十个业务接口 | APIRouter / include_router |
| 一次性查询 Agent 信息 | 普通 JSON 响应 |
| 让后端服务逐行消费结构化分析记录 | JSON Lines |
| 网页实时显示文本与工具执行状态 | SSE（可考虑） |
| 上传原始音视频或大型二进制流 | 进阶 StreamingResponse，非本章重点 |

## 端到端 Agent 场景

```text
用户在网页输入问题
  ↓  HTTP POST /chat
FastAPI 主应用：路由由 include_router 注册
  ↓
chat.router：参数校验与身份检查
  ↓
Agent / LangGraph：推理、调用搜索工具
  ↓
逐步产出结果或事件
  ├─ JSON Lines：后端消费者逐条读取结构化数据
  └─ SSE：前端接收 progress / token / done 事件并更新界面
```

## 容易混淆的六个结论

1. APIRouter 不是新的 Web 服务器，只是同一个 FastAPI 应用里的接口分组。
2. 业务模块（Agent）与 API 路由模块（Router）职责不同。
3. JSON Lines 和 SSE 都可以流式传输，但协议格式不同。
4. `yield` 在依赖函数中常用于资源管理，在流式路由中用于逐条产生数据。
5. SSE 不是 WebSocket：SSE 主要负责服务器向客户端单向推送；WebSocket 支持双向通信。
6. SSE 的事件 ID 不等于自动持久化、重试可靠性或 Agent 状态恢复。

## 10 道复习题（答案与解析）

**1. 为什么要用 APIRouter？**  答：把相关接口放进不同文件，统一注册到主应用，提升维护性；不是为了自动创建微服务。

**2. `include_router` 做什么？**  答：将一个 Router 的路径操作加入 FastAPI 主应用；/docs 仍展示合并的路由。

**3. `prefix='/agents'` 与 `@router.get('/{agent_id}')` 组合后是什么路径？**  答：`GET /agents/{agent_id}`；prefix 为一组接口统一添加前缀。

**4. JSON Lines 能否写成一个包在 `[]` 里的 JSON 数组？**  答：不是这个格式；它以换行分隔多个独立 JSON 值。

**5. 为什么流式路由使用 yield 而不是直接 return 列表？**  答：可以逐条产出数据，使客户端在整个序列准备完之前开始处理。

**6. `AsyncIterable[Item]` 在这个章节里主要表示什么？**  答：该异步函数逐步产生 Item；声明类型还能帮助验证、序列化与文档生成。

**7. SSE 与流式输出是互相替代的吗？**  答：不是；流式输出是效果/总体行为，SSE 是一种支持该效果的具体传输协议。

**8. SSE 的 `event='progress'` 作用是什么？**  答：告诉客户端事件类型，便于分别更新进度条、文字区等 UI。

**9. 看到 `Last-Event-ID: 4`，能否断言 Agent 会自动恢复？**  答：不能；它只告诉服务器此前接收到的事件标识，恢复需要额外的服务器端实现。

**10. 如果用户需要同时看到文字和工具执行进度，应优先考虑什么？**  答：可以考虑 SSE，把内容和进度定义成不同事件，并由客户端按类型显示。不是强制唯一方案。

## 三道思考题（附答题要点）

**A. 设计一个中型 Agent 项目：用户、聊天、文件与会话接口放在哪？**

答题要点：按 routers/users.py、chat.py、files.py、sessions.py 分组，main.py 统一 include_router；Agent 的业务逻辑独立放到 agents/ 或 services/；共享校验可通过 dependencies.py 提供。

**B. 用户说“我明明在等 Agent，为什么只能看到加载圈，不能看到正在搜索？”**

答题要点：检查内部 Agent 是否产出增量状态、服务器是否分段/按事件发送、客户端是否及时消费并渲染、代理有无缓冲；使用 SSE 只是其中一层条件。

**C. JSON Lines 与 SSE，选择哪个作为研究型 Agent 的返回形式？**

答题要点：如果接收端主要是另一个处理结构化数据的后端，JSON Lines 很直接；如果需要浏览器原生事件机制与区分 progress/token/done，SSE 很方便；选择取决于客户端需求与协议约定。

## 官方文档索引

- 更大的应用－多个文件：https://fastapi.tiangolo.com/zh/tutorial/bigger-applications/
- 流式传输 JSON Lines：https://fastapi.tiangolo.com/zh/tutorial/stream-json-lines/
- 服务器发送事件（SSE）：https://fastapi.tiangolo.com/zh/tutorial/server-sent-events/

**本册最后记住：** 更大的应用负责组织接口；JSON Lines 让结构化数据能逐条传送；SSE 让数据能够以带类型的事件持续推送。它们可以在同一个 Agent 后端中各司其职。
