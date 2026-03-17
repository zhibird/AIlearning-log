FastAPI——“**专门拿来写接口的 Python 后端框架**”。

## 1）先讲最官方的定义

FastAPI 官方首页给出的定义是：**FastAPI 是一个现代的、快速（高性能）的 Web 框架，用来用 Python 构建 API，并且它建立在标准 Python 类型提示（type hints）之上。** 官方同时把它概括为：高性能、容易学习、开发速度快、适合生产环境。([FastAPI][1])

把这句话拆开看，核心其实就 4 个词：

1. **Web 框架**：说明它是帮你写后端服务的“工具箱”。([FastAPI][1])
2. **构建 API**：说明它最擅长的不是做传统网页模板，而是做“前端 / App / 其他系统”调用的接口。([FastAPI][1])
3. **高性能**：官方强调它性能很高，并把它列为 Python 里非常快的一类框架。([FastAPI][1])
4. **基于类型提示**：你在函数参数里写 `str`、`int`、`bool`、`BaseModel` 这些类型，不只是给人看，FastAPI 会拿这些类型去做参数解析、数据校验、文档生成。([FastAPI][1])

## 2）用小学生都能懂的话解释

你可以把一个网站系统想成一家餐厅：

* **前端**是顾客
* **后端**是厨房
* **API** 是点餐规则
* **FastAPI** 就是餐厅门口那个特别聪明的接单员

这个接单员有几个厉害的地方：

* 顾客说“我要 2 号套餐”，它知道这是不是合法订单。
* 顾客少填了内容、填错了内容，它会马上拦下来，不让错误订单进厨房。
* 它会自动把“这家店有哪些菜、每道菜怎么点”整理成一本说明书，这就是自动接口文档。
* 它还很擅长同时接很多单，特别适合“查数据库、调别的 API、上传文件、聊天推送”这类需要等待的工作。这个能力和它对异步处理、依赖注入、自动文档这些设计有关。([FastAPI][2])

再压缩成一句话：

**FastAPI = 一个帮你快速写接口、自动检查数据、自动生成接口说明书、还能比较优雅处理高并发请求的 Python 后端框架。** ([FastAPI][1])

## 3）它具体用在什么场景

FastAPI 最典型的使用场景，是你要做一个“**给别人调用的后端接口服务**”时。比如：

### 场景 A：前后端分离项目的后端

前端 Vue / React 页面发请求给 FastAPI，FastAPI 负责收参数、查数据库、返回 JSON。官方教程的核心就是围绕路径参数、查询参数、请求体、响应模型来组织的。([FastAPI][3])

### 场景 B：登录、鉴权、权限控制

官方专门有 Security 章节，说明 FastAPI 很适合做认证与授权这类后端接口场景。([FastAPI][4])

### 场景 C：数据库 CRUD 系统

比如用户管理、文章系统、订单系统、任务系统。官方数据库教程明确说 FastAPI 不强制你用某种数据库；官方示例用的是 SQLModel，而 SQLModel 建立在 SQLAlchemy 和 Pydantic 之上。([FastAPI][5])

### 场景 D：实时通信

比如聊天、在线状态、消息推送。官方提供了 WebSocket 支持。([FastAPI][6])

### 场景 E：接口返回很快，但后面还有事要做

比如“先返回提交成功，再异步发邮件 / 记日志 / 处理文件”。官方提供了 Background Tasks。([FastAPI][7])

### 场景 F：微服务、容器化部署

官方文档专门讲了 Docker / Kubernetes 等容器部署方式，也讲了大应用如何拆成多个文件和模块。([FastAPI][8])

## 4）你第一次该怎么理解它的“工作流”

最基础的 FastAPI 工作流其实就是这几步：

**定义 app → 写路由 → 声明参数类型 → FastAPI 自动校验 → 返回 JSON → 自动生成文档。** 官方 first steps 里也正是这样展开的，而且运行后你可以直接访问 `/docs` 和 `/redoc` 看自动文档。([FastAPI][3])

你可以把它想成：

**请求进来 → FastAPI 按你写的类型把数据拆好、验好 → 调你的函数 → 把结果再整理成 JSON → 顺便把规则写进 OpenAPI 文档里。** OpenAPI schema 还可以拿来生成前端、移动端、IoT 等客户端代码。([FastAPI][2])

## 5）伪代码：看一个最小但完整的脑图版

下面我故意写成“接近 Python、但更重理解”的伪代码：

```python
# 1. 导入框架和数据模型工具
from fastapi import FastAPI, Depends, HTTPException, Query
from pydantic import BaseModel

# 2. 创建 app
app = FastAPI()

# 3. 定义“请求体”的数据格式
class UserCreate(BaseModel):
    username: str
    age: int

# 4. 定义“返回给前端”的数据格式
class UserPublic(BaseModel):
    id: int
    username: str
    age: int

# 5. 定义依赖：比如数据库连接
def get_db():
    db = "数据库会话"
    return db

# 6. 写 GET 接口：查一个用户
@app.get("/users/{user_id}", response_model=UserPublic)
def get_user(user_id: int, verbose: bool = Query(False), db = Depends(get_db)):
    user = db.find_user_by_id(user_id)

    if user 不存在:
        raise HTTPException(status_code=404, detail="用户不存在")

    return user

# 7. 写 POST 接口：新建一个用户
@app.post("/users", response_model=UserPublic)
def create_user(payload: UserCreate, db = Depends(get_db)):
    new_user = db.insert(payload)
    return new_user
```

这段伪代码里，最关键的是 5 个点：

* `@app.get(...)` / `@app.post(...)`：表示“这个函数对应哪个接口”。([FastAPI][3])
* `user_id: int`：表示路径参数必须是整数。([FastAPI][9])
* `payload: UserCreate`：表示请求体要按 Pydantic 模型校验。([FastAPI][10])
* `db = Depends(get_db)`：表示这个函数依赖某个外部资源，让 FastAPI 帮你注入。([FastAPI][11])
* `response_model=UserPublic`：表示响应也按规则过滤和校验，官方还特别强调这对安全很重要，因为它会限制输出字段。([FastAPI][12])

## 6）你学习 FastAPI 时最容易踩的坑

### 坑 1：把它只当“更快的 Flask”

这不完全对。FastAPI 不只是“快”，它真正的特点是：**把 Python 类型提示、参数校验、响应模型、依赖注入、自动文档整合到了一起。** 所以它的开发体验和约束方式，和 Flask 那种“更轻、更自由、很多东西自己拼”很不一样。([FastAPI][1])

### 坑 2：以为 `async def` 一定更快

官方不是这么说的。官方明确给了规则：如果你用的库需要 `await`，就写 `async def`；如果你调用的是不支持 `await` 的阻塞库，就直接写普通 `def`。不是所有接口都该无脑异步。([FastAPI][13])

### 坑 3：以为“自动文档”只是好看

不是。官方说明 OpenAPI schema 不仅驱动 `/docs` 和 `/redoc`，还可以用于自动生成客户端代码，所以这不只是“展示层功能”，而是接口契约的一部分。([FastAPI][2])

## 7）我替你搜过了：FastAPI 常见面试八股文，高频点主要是这些

综合中文面试整理、社区题库和英文面试题整理，FastAPI 高频题目主要集中在：**路由、路径参数与查询参数、Pydantic 请求体、依赖注入、异步/并发、自动文档、数据库集成、中间件/CORS、JWT 鉴权、WebSocket、后台任务、性能优化、测试** 这些主题。([CSDN][14])

你可以先背下面这 10 个最核心的：

### 1. 什么是 FastAPI？它为什么叫 FastAPI？

答题骨架：它是一个基于 Python 类型提示的高性能 API Web 框架；“Fast”既指运行性能，也指开发速度快。([FastAPI][1])

### 2. FastAPI 和 Flask 的区别是什么？

答题骨架：FastAPI 原生围绕类型提示、请求体验证、响应模型、依赖注入、自动 OpenAPI 文档来设计；Flask 更轻、更自由，但很多能力需要自行组合。社区面试文里还常把两者放在异步支持和接口契约清晰度上对比。([CSDN][15])

### 3. 路径参数、查询参数、请求体分别是什么？

答题骨架：`/users/1` 里的 `1` 是路径参数；`?page=2` 是查询参数；POST/PUT/PATCH 等请求里传 JSON 的那部分通常是请求体。FastAPI 会根据函数签名和 Pydantic 模型自动识别并校验。([CSDN][14])

### 4. Pydantic 在 FastAPI 里是干什么的？

答题骨架：它主要负责数据模型定义、请求体验证、序列化，以及生成文档所需的 schema。([FastAPI][10])

### 5. `response_model` 有什么用？

答题骨架：它不仅生成响应文档，还会校验和过滤输出字段；官方明确说这对安全尤其重要，因为它会限制你真正返回给客户端的数据。([FastAPI][12])

### 6. FastAPI 的依赖注入是什么？为什么好用？

答题骨架：你把数据库连接、鉴权逻辑、公共参数、权限校验等抽成依赖函数，再通过 `Depends()` 注入给接口。官方强调它能复用逻辑、共享数据库连接、处理安全要求并减少重复代码。([FastAPI][11])

### 7. `async def` 和 `def` 在 FastAPI 里怎么选？

答题骨架：如果你调用的是可 `await` 的异步库，就用 `async def`；如果是普通阻塞库，就用 `def`。面试官很爱追问：“你在 async 接口里直接写阻塞代码会怎样？”答：会把异步优势打掉。([FastAPI][13])

### 8. FastAPI 为什么能自动生成 Swagger 文档？

答题骨架：因为它会根据你定义的路由、参数类型、请求体模型、响应模型自动生成 OpenAPI schema，然后 `/docs` 用 Swagger UI 展示，`/redoc` 用 ReDoc 展示。([FastAPI][2])

### 9. FastAPI 如何连数据库？

答题骨架：FastAPI 本身不强制数据库方案；官方示例里可以用 SQLModel，而 SQLModel 建立在 SQLAlchemy 和 Pydantic 上。实际项目里也常见直接配 SQLAlchemy。([FastAPI][5])

### 10. FastAPI 适合哪些项目，不适合哪些项目？

答题骨架：它特别适合做接口服务、前后端分离后端、模型服务、微服务、需要清晰接口契约和自动文档的系统；而如果你做的是传统服务端模板网页、历史项目已经高度绑定 Flask/Django 生态，选型就要综合团队习惯和迁移成本。([FastAPI][1])

## 8）你现在最值得先记住的一句话

**FastAPI 的本质，不只是“快”，而是“用 Python 类型提示把接口定义、数据校验、文档生成、依赖管理这些事整合在一起”。** 你一旦理解了这句，后面学路由、Pydantic、Depends、JWT、数据库时就不会散。([FastAPI][1])

接下来如果你愿意，我可以直接按“**12岁也能学懂但又严格正确**”的方式，给你继续讲 FastAPI 的第一课：**路由、路径参数、查询参数、请求体，这四个概念到底分别是什么，它们在一次真实请求里怎么流动。**

[1]: https://fastapi.tiangolo.com/ "FastAPI"
[2]: https://fastapi.tiangolo.com/tutorial/first-steps/ "First Steps - FastAPI"
[3]: https://fastapi.tiangolo.com/tutorial/first-steps/?utm_source=chatgpt.com "First Steps"
[4]: https://fastapi.tiangolo.com/tutorial/security/ "Security - FastAPI"
[5]: https://fastapi.tiangolo.com/tutorial/sql-databases/ "SQL (Relational) Databases - FastAPI"
[6]: https://fastapi.tiangolo.com/advanced/websockets/?utm_source=chatgpt.com "WebSockets"
[7]: https://fastapi.tiangolo.com/tutorial/background-tasks/?utm_source=chatgpt.com "Background Tasks"
[8]: https://fastapi.tiangolo.com/deployment/docker/ "FastAPI in Containers - Docker - FastAPI"
[9]: https://fastapi.tiangolo.com/tutorial/query-params-str-validations/ "Query Parameters and String Validations - FastAPI"
[10]: https://fastapi.tiangolo.com/tutorial/body/ "Request Body - FastAPI"
[11]: https://fastapi.tiangolo.com/tutorial/dependencies/ "Dependencies - FastAPI"
[12]: https://fastapi.tiangolo.com/tutorial/response-model/ "Response Model - Return Type - FastAPI"
[13]: https://fastapi.tiangolo.com/async/?utm_source=chatgpt.com "Concurrency and async / await"
[14]: https://blog.csdn.net/HappyAcmen/article/details/146421303 "关于FastAPI框架的面试题及答案解析_fastapi面试题-CSDN博客"
[15]: https://blog.csdn.net/HelloFif/article/details/150518899?utm_source=chatgpt.com "【Python面试题】为什么选择Fastapi而不是Flask做项目？ 原创"
