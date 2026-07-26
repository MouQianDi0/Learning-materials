# 后端开发从零到进阶：系统学习指南

> 面向基础学习者的渐进式后端教程  
> 示例主线：Node.js + TypeScript + Express + PostgreSQL  
> 扩展技术栈：Java/Spring Boot、Python/FastAPI、Go/Gin、Redis、MinIO、Docker、Nginx、消息队列、微服务  
> 项目案例：NoteFlow 移动端笔记应用

---

## 目录

- [0. 阅读方式与学习原则](#0-阅读方式与学习原则)
- [1. 后端到底是什么](#1-后端到底是什么)
- [2. 一次请求是怎样完成的](#2-一次请求是怎样完成的)
- [3. 开始后端前必须掌握的基础](#3-开始后端前必须掌握的基础)
- [4. 编程语言与后端技术栈怎么选](#4-编程语言与后端技术栈怎么选)
- [5. Node.js 与异步编程](#5-nodejs-与异步编程)
- [6. HTTP、HTTPS 与 REST API](#6-httphttps-与-rest-api)
- [7. 用 Express 写出第一个后端服务](#7-用-express-写出第一个后端服务)
- [8. 从单文件走向真实项目结构](#8-从单文件走向真实项目结构)
- [9. 数据库与 SQL 基础](#9-数据库与-sql-基础)
- [10. PostgreSQL 数据建模](#10-postgresql-数据建模)
- [11. 索引、事务与并发控制](#11-索引事务与并发控制)
- [12. ORM、Prisma 与原生 SQL](#12-ormprisma-与原生-sql)
- [13. 登录、Session、JWT 与权限控制](#13-登录sessionjwt-与权限控制)
- [14. 参数校验、异常处理与统一响应](#14-参数校验异常处理与统一响应)
- [15. 后端安全基础](#15-后端安全基础)
- [16. Redis 与缓存](#16-redis-与缓存)
- [17. 文件上传、对象存储与 MinIO](#17-文件上传对象存储与-minio)
- [18. 定时任务、消息队列与异步处理](#18-定时任务消息队列与异步处理)
- [19. 日志、监控与可观测性](#19-日志监控与可观测性)
- [20. 后端测试](#20-后端测试)
- [21. Docker、Nginx 与生产部署](#21-dockernginx-与生产部署)
- [22. 性能优化与高并发入门](#22-性能优化与高并发入门)
- [23. 单体、模块化单体与微服务](#23-单体模块化单体与微服务)
- [24. Java、Python、Go 后端快速认识](#24-javapythongo-后端快速认识)
- [25. NoteFlow 后端完整案例](#25-noteflow-后端完整案例)
- [26. 由浅入深的练习项目](#26-由浅入深的练习项目)
- [27. 24 周学习路线](#27-24-周学习路线)
- [28. 后端开发固定思维](#28-后端开发固定思维)
- [29. 常见故障排查清单](#29-常见故障排查清单)
- [30. 面试题与自测题](#30-面试题与自测题)
- [31. 常用术语表](#31-常用术语表)
- [32. 下一步行动清单](#32-下一步行动清单)

---

## 0. 阅读方式与学习原则

这份文档不是一张只告诉你“学什么”的路线图，而是一份尽量说明“为什么学、怎么学、学完能做什么”的教程。

建议采用下面的节奏：

1. 第一次阅读时先理解概念，不要求记住所有语法。
2. 每一章至少手写一个小例子，不要只复制代码。
3. 代码运行失败时先读错误信息，再搜索或询问 AI。
4. 每完成一个阶段，就把知识放进一个真实项目。
5. 不要同时深入学习五种语言，先用一种语言打通完整后端流程。

### 0.1 推荐的主线

如果你已经接触 JavaScript、TypeScript、React 或 React Native，推荐先走：

```text
JavaScript / TypeScript
        ↓
Node.js
        ↓
Express 或 NestJS
        ↓
PostgreSQL
        ↓
Redis + MinIO
        ↓
Docker + Nginx
        ↓
测试、监控、CI/CD
```

这条路线能让你较快地把 NoteFlow 之类的项目完整做出来。

### 0.2 “知道”和“会用”的区别

例如你知道 JWT 是一种登录凭证，并不代表你已经会做登录系统。真正掌握至少包括：

- 能解释 Token 从哪里生成。
- 知道密码为什么不能明文保存。
- 知道 Access Token 过期后怎么办。
- 知道 Token 泄漏可能造成什么后果。
- 能写鉴权中间件并测试非法请求。
- 能区分“已经登录”和“有权操作这个资源”。

学习后端时，要不断问自己：

> 如果这段代码出错了、被攻击了、并发执行了、部署到服务器了，它还可靠吗？

---

## 1. 后端到底是什么

### 1.1 用餐厅理解前后端

可以把一个应用想象成餐厅：

- 前端像菜单和服务员：负责展示、交互、收集用户需求。
- 后端像厨房：接收订单、检查规则、制作结果。
- 数据库像仓库和账本：保存食材、订单、会员信息。
- API 像服务员和厨房之间的标准传单。
- 服务器像餐厅所在的建筑和设备。

当用户在 NoteFlow 中点击“新建笔记”时：

```text
用户输入标题和内容
      ↓
React Native 组织请求
      ↓
POST /api/notes
      ↓
后端检查登录状态和数据格式
      ↓
PostgreSQL 写入笔记
      ↓
后端返回创建结果
      ↓
前端更新页面
```

### 1.2 后端的核心职责

一个真实后端通常负责：

1. 接收并解析客户端请求。
2. 校验用户输入。
3. 执行业务规则。
4. 读写数据库。
5. 进行身份认证和权限判断。
6. 调用第三方服务。
7. 处理文件、缓存、消息和定时任务。
8. 记录日志并监控运行状态。
9. 在高并发和异常情况下保持可靠。

### 1.3 后端不等于数据库

数据库只负责存取数据，后端还要决定：

- 谁能读取这条数据？
- 这条数据是否符合规则？
- 删除用户时是否要删除他的笔记？
- 标题为空时应该返回什么？
- 短时间内请求一万次是否需要限流？

数据库像仓库，后端像仓库管理员。仓库能存东西，但“谁可以拿、一次拿多少、拿走后如何登记”是管理员的工作。

---

## 2. 一次请求是怎样完成的

### 2.1 从手机到服务器

假设 App 请求：

```http
GET https://api.example.com/api/notes
Authorization: Bearer eyJ...
```

大致经过：

1. DNS 把 `api.example.com` 解析成服务器 IP。
2. 手机与服务器建立 TCP 连接。
3. HTTPS 通过 TLS 建立加密通道。
4. Nginx 接收到请求。
5. Nginx 把 `/api/` 请求转发到 Node.js 服务。
6. Express 匹配 `GET /api/notes` 路由。
7. 鉴权中间件检查 Token。
8. Service 执行业务逻辑。
9. Repository 查询 PostgreSQL。
10. 数据按照 JSON 返回客户端。

### 2.2 分层理解

可以把网络通信看成快递：

- DNS：查找收件地址。
- IP：城市和道路地址。
- TCP：建立可靠运输通道。
- TLS：把包裹锁起来。
- HTTP：规定快递单怎么填写。
- JSON：包裹中物品的组织格式。
- API：双方约定能寄什么、收到什么。

### 2.3 JSON 示例

请求体：

```json
{
  "title": "学习 HTTP",
  "content": "今天理解了请求与响应"
}
```

成功响应：

```json
{
  "success": true,
  "data": {
    "id": 101,
    "title": "学习 HTTP",
    "content": "今天理解了请求与响应"
  }
}
```

失败响应：

```json
{
  "success": false,
  "error": {
    "code": "INVALID_TITLE",
    "message": "笔记标题不能为空"
  }
}
```

---

## 3. 开始后端前必须掌握的基础

### 3.1 编程基础

至少需要掌握：

- 变量和数据类型
- 条件判断
- 循环
- 函数
- 数组和对象
- 模块导入导出
- 异常处理
- 异步编程
- 基础面向对象思想

TypeScript 示例：

```ts
// 定义笔记的数据结构。
interface Note {
  id: number;
  title: string;
  content: string;
  createdAt: Date;
}

// 函数参数和返回值都写出类型，方便编译器提前发现错误。
function getNoteTitle(note: Note): string {
  return note.title;
}
```

### 3.2 命令行基础

后端开发经常运行在没有图形界面的 Linux 服务器上，因此要逐渐熟悉命令行。

```bash
# 查看当前目录
pwd

# 查看文件列表
ls -la

# 创建目录
mkdir backend-demo

# 进入目录
cd backend-demo

# 查看正在监听的端口
ss -lntp

# 查看进程
ps aux

# 实时查看日志
tail -f app.log
```

不要死记命令。先记住“我想做什么”，再知道使用哪个命令。

### 3.3 Git 基础

建议理解：

- 工作区、暂存区、提交
- 分支
- 合并
- 冲突
- 远程仓库
- Pull Request

常见流程：

```bash
# 从主分支创建功能分支
git switch -c feature/note-search

# 查看改动
git status

# 添加需要提交的文件
git add src/modules/notes

# 提交一个完整的小功能
git commit -m "feat: add note search API"

# 推送到远程仓库
git push -u origin feature/note-search
```

### 3.4 数据结构与算法需要学到什么程度

初期不需要先刷几百道算法题，但要理解：

- 数组、链表、栈、队列
- 哈希表
- 树的基础
- 时间复杂度
- 空间复杂度
- 排序与查找

例如哈希表常用于：

- 内存缓存
- 根据 ID 快速查找对象
- 统计次数
- 去重

```ts
const noteCountByCategory = new Map<string, number>();

for (const note of notes) {
  const oldCount = noteCountByCategory.get(note.category) ?? 0;
  noteCountByCategory.set(note.category, oldCount + 1);
}
```

---

## 4. 编程语言与后端技术栈怎么选

### 4.1 常见技术栈对比

| 语言 | 常见框架 | 特点 | 适合方向 |
|---|---|---|---|
| JavaScript/TypeScript | Express、NestJS | 前后端语言统一，生态丰富 | Web API、全栈、实时应用 |
| Java | Spring Boot | 工程体系成熟，企业应用多 | 大型企业系统、金融、电商 |
| Python | FastAPI、Django | 语法清晰，AI/数据生态强 | Web API、AI 服务、数据平台 |
| Go | Gin、Fiber | 编译快、部署简单、并发能力强 | 云原生、网关、基础设施 |
| C# | ASP.NET Core | 工程化强，微软生态完善 | 企业系统、游戏服务 |
| PHP | Laravel | Web 生态成熟，部署普遍 | 内容网站、管理系统、电商 |
| Rust | Axum、Actix Web | 性能和内存安全优秀，学习曲线陡 | 高性能系统、基础设施 |

### 4.2 不要陷入“最好语言”争论

语言像交通工具：

- 去附近超市，骑自行车很方便。
- 跨城运输，货车更合适。
- 飞越海洋，需要飞机。

没有一个工具在所有场景中都最好。对学习者来说，最重要的是：

> 先用一种语言完整经历开发、测试、数据库、部署和维护，再学习第二种语言比较差异。

### 4.3 推荐策略

你的主线可以先用 TypeScript + Node.js，因为能复用前端语言经验。之后：

- 想走企业后端：补 Java + Spring Boot。
- 想结合 AI：补 Python + FastAPI。
- 想学云原生和高性能服务：补 Go。
- 想深入系统底层：再考虑 Rust、C/C++。

---

## 5. Node.js 与异步编程

### 5.1 Node.js 是什么

JavaScript 最初主要在浏览器运行。Node.js 提供了浏览器之外的 JavaScript 运行环境，使 JavaScript 能够：

- 启动服务器。
- 读写文件。
- 连接数据库。
- 调用操作系统能力。
- 编写命令行工具。

Node.js 不是编程语言，JavaScript 才是语言。Node.js 更像运行 JavaScript 的“发动机和工具箱”。

### 5.2 同步与异步

同步像在窗口排队：前一个人没办完，后一个人不能办理。

异步像餐厅点餐：下单后厨师做菜，你不需要站在厨房门口等待，可以先做别的事。

```ts
import { readFile } from "node:fs/promises";

async function loadConfig() {
  try {
    // await 会暂停当前函数，但不会阻塞整个 Node.js 进程。
    const text = await readFile("./config.json", "utf-8");
    const config = JSON.parse(text);
    return config;
  } catch (error) {
    console.error("读取配置失败", error);
    throw error;
  }
}
```

### 5.3 Promise

Promise 可以理解为“取餐号码牌”：

- `pending`：正在制作。
- `fulfilled`：制作成功，可以取餐。
- `rejected`：制作失败。

```ts
function wait(ms: number): Promise<void> {
  return new Promise((resolve) => {
    // 时间到后调用 resolve，表示任务成功完成。
    setTimeout(resolve, ms);
  });
}

async function demo() {
  console.log("开始等待");
  await wait(1000);
  console.log("一秒后继续");
}
```

### 5.4 事件循环

Node.js 的 JavaScript 主线程通常一次执行一段代码，但文件、网络等 I/O 可以交给系统处理。任务完成后，回调会进入队列，等待主线程空闲。

需要注意：

- I/O 密集任务适合 Node.js。
- 大量纯 CPU 计算会长时间占用主线程。
- 图片压缩、大模型计算等重任务可放入 Worker、独立服务或任务队列。

错误示例：

```ts
// 这段循环会长期占用主线程，期间其他请求可能无法及时处理。
for (let i = 0; i < 10_000_000_000; i++) {
  // 执行大量 CPU 计算
}
```

### 5.5 并行请求

```ts
async function loadDashboard(userId: number) {
  // 三个任务互不依赖，可以同时开始。
  const [user, notes, categories] = await Promise.all([
    getUser(userId),
    getNotes(userId),
    getCategories(userId),
  ]);

  return { user, notes, categories };
}
```

不要把互不依赖的请求全部写成连续 `await`，否则会产生不必要的等待。

---

## 6. HTTP、HTTPS 与 REST API

### 6.1 HTTP 请求结构

一个 HTTP 请求主要包括：

- 请求方法
- URL
- 请求头
- 请求体

```http
POST /api/notes HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token

{
  "title": "后端学习",
  "content": "学习 HTTP 请求结构"
}
```

### 6.2 常见请求方法

| 方法 | 常见含义 | 示例 |
|---|---|---|
| GET | 查询资源 | `GET /api/notes` |
| POST | 创建资源 | `POST /api/notes` |
| PUT | 整体替换资源 | `PUT /api/notes/10` |
| PATCH | 部分修改资源 | `PATCH /api/notes/10` |
| DELETE | 删除资源 | `DELETE /api/notes/10` |

### 6.3 PUT 和 PATCH

假设用户数据为：

```json
{
  "nickname": "demo_user",
  "avatar": "a.png",
  "bio": "Developer"
}
```

`PUT` 更强调提交完整的新状态；`PATCH` 只提交要修改的字段：

```json
{
  "bio": "React Native Developer"
}
```

实际项目中团队可能采用不同约定，重点是接口文档必须清楚。

### 6.4 状态码

| 状态码 | 含义 | 场景 |
|---|---|---|
| 200 | 成功 | 查询、更新成功 |
| 201 | 已创建 | 新建笔记成功 |
| 204 | 成功但无响应体 | 删除成功 |
| 400 | 请求格式错误 | JSON 错误、参数缺失 |
| 401 | 未认证 | 没有登录或 Token 无效 |
| 403 | 无权限 | 已登录但不能操作 |
| 404 | 资源不存在 | 笔记 ID 不存在 |
| 409 | 资源冲突 | 邮箱已注册 |
| 422 | 数据校验失败 | 字段格式不合法 |
| 429 | 请求过多 | 触发限流 |
| 500 | 服务内部错误 | 未处理异常 |

### 6.5 REST API 设计

推荐围绕“资源”设计路径：

```text
GET    /api/notes
GET    /api/notes/:id
POST   /api/notes
PATCH  /api/notes/:id
DELETE /api/notes/:id
```

不推荐把所有动作直接写进路径：

```text
/getAllNotes
/createNewNote
/deleteNoteById
```

REST 不是 HTTP 唯一选择。其他常见方案包括：

- GraphQL：客户端选择需要的字段。
- gRPC：高效的服务间通信。
- WebSocket：双向实时通信。
- Server-Sent Events：服务器持续向客户端推送。

### 6.6 HTTPS

HTTP 明文传输像寄透明信封；HTTPS 像先建立加密通道，再传输内容。

HTTPS 主要提供：

- 加密：别人难以直接读取内容。
- 完整性：内容被篡改时可以发现。
- 身份验证：证书帮助确认服务器身份。

但 HTTPS 不会自动解决：

- 弱密码
- SQL 注入
- Token 被应用日志泄漏
- 服务器代码漏洞
- 权限判断错误

---

## 7. 用 Express 写出第一个后端服务

### 7.1 初始化项目

```bash
mkdir express-notes-api
cd express-notes-api
npm init -y
npm install express
npm install -D typescript tsx @types/node @types/express
npx tsc --init
```

### 7.2 最小服务

```ts
import express from "express";

const app = express();
const port = 3000;

// 让 Express 自动解析 application/json 请求体。
app.use(express.json());

app.get("/health", (_req, res) => {
  // 健康检查接口通常用于确认服务是否正常存活。
  res.status(200).json({
    status: "ok",
    timestamp: new Date().toISOString(),
  });
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

### 7.3 内存版 CRUD

```ts
import express from "express";

interface Note {
  id: number;
  title: string;
  content: string;
}

const app = express();
app.use(express.json());

// 暂时把数据放在内存中，服务重启后会消失。
const notes: Note[] = [];
let nextId = 1;

app.get("/api/notes", (_req, res) => {
  res.json({ success: true, data: notes });
});

app.post("/api/notes", (req, res) => {
  const { title, content } = req.body;

  // 后端永远不能默认相信客户端传入的数据。
  if (typeof title !== "string" || title.trim() === "") {
    return res.status(422).json({
      success: false,
      message: "title 不能为空",
    });
  }

  const newNote: Note = {
    id: nextId++,
    title: title.trim(),
    content: typeof content === "string" ? content : "",
  };

  notes.push(newNote);
  return res.status(201).json({ success: true, data: newNote });
});

app.delete("/api/notes/:id", (req, res) => {
  const noteId = Number(req.params.id);
  const index = notes.findIndex((note) => note.id === noteId);

  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: "笔记不存在",
    });
  }

  notes.splice(index, 1);
  return res.status(204).send();
});

app.listen(3000);
```

### 7.4 中间件

中间件像进入工厂前的一道道检查门：

```text
请求
 ↓
请求日志
 ↓
限流
 ↓
身份认证
 ↓
参数校验
 ↓
业务处理
 ↓
响应
```

```ts
import type { NextFunction, Request, Response } from "express";

function requestLogger(req: Request, _res: Response, next: NextFunction) {
  const start = Date.now();

  console.log(`[REQUEST] ${req.method} ${req.path}`);

  // 调用 next 后，Express 才会继续执行后面的中间件或路由。
  next();

  console.log(`[REQUEST END] cost=${Date.now() - start}ms`);
}
```

---

## 8. 从单文件走向真实项目结构

### 8.1 为什么不能全写在 app.ts

小项目把代码写在一个文件里很快，但功能增加后会出现：

- 路由、SQL、业务逻辑混在一起。
- 修改登录功能时影响笔记功能。
- 测试困难。
- 重复代码变多。
- 新成员难以定位文件。

### 8.2 推荐的模块化结构

```text
src/
├── app.ts
├── server.ts
├── config/
│   ├── env.ts
│   └── database.ts
├── middlewares/
│   ├── auth.middleware.ts
│   ├── error.middleware.ts
│   └── validate.middleware.ts
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.repository.ts
│   │   ├── auth.routes.ts
│   │   └── auth.schema.ts
│   └── notes/
│       ├── note.controller.ts
│       ├── note.service.ts
│       ├── note.repository.ts
│       ├── note.routes.ts
│       └── note.schema.ts
├── shared/
│   ├── errors/
│   ├── logger/
│   └── utils/
└── types/
```

### 8.3 各层职责

| 层 | 负责什么 | 不应该负责什么 |
|---|---|---|
| Router | URL 与处理器映射 | 复杂业务 |
| Controller | 解析请求、组织响应 | 直接写复杂 SQL |
| Service | 业务规则 | 依赖 HTTP 细节 |
| Repository | 数据读写 | 决定用户权限 |
| Middleware | 通用请求流程 | 某个模块的核心业务 |

### 8.4 示例

Controller：

```ts
export async function createNoteController(req: Request, res: Response) {
  // Controller 只读取 HTTP 数据，并把任务交给 Service。
  const note = await noteService.create({
    userId: req.user.id,
    title: req.body.title,
    content: req.body.content,
  });

  return res.status(201).json({ success: true, data: note });
}
```

Service：

```ts
export async function create(input: CreateNoteInput) {
  // 业务规则应该集中在 Service，而不是散落在不同路由中。
  if (input.title.trim().length > 100) {
    throw new AppError("NOTE_TITLE_TOO_LONG", 422, "标题不能超过 100 个字符");
  }

  return noteRepository.create({
    ...input,
    title: input.title.trim(),
  });
}
```

Repository：

```ts
export async function create(input: CreateNoteInput) {
  const result = await db.query(
    `INSERT INTO notes (user_id, title, content)
     VALUES ($1, $2, $3)
     RETURNING id, user_id, title, content, created_at`,
    // 参数化查询可以避免把用户输入直接拼接进 SQL。
    [input.userId, input.title, input.content],
  );

  return result.rows[0];
}
```

---

## 9. 数据库与 SQL 基础

### 9.1 为什么需要数据库

内存变量在服务重启后会丢失。数据库负责：

- 持久保存数据。
- 并发读写。
- 查询和统计。
- 保证约束。
- 事务处理。
- 备份与恢复。

### 9.2 关系型与非关系型数据库

关系型数据库：

- PostgreSQL
- MySQL
- SQL Server
- SQLite

特点是表、行、列、关系和 SQL，适合结构明确、事务要求高的数据。

非关系型数据库：

- MongoDB：文档数据库。
- Redis：内存键值数据库。
- Elasticsearch/OpenSearch：搜索与分析。
- Neo4j：图数据库。

不要把“非关系型”理解为“更高级”。选型要看数据和业务。

### 9.3 基础 SQL

创建表：

```sql
CREATE TABLE notes (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL,
  title VARCHAR(100) NOT NULL,
  content TEXT NOT NULL DEFAULT '',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

插入：

```sql
INSERT INTO notes (user_id, title, content)
VALUES (1, 'SQL 学习', '今天学习 INSERT')
RETURNING *;
```

查询：

```sql
SELECT id, title, created_at
FROM notes
WHERE user_id = 1
ORDER BY created_at DESC
LIMIT 20;
```

修改：

```sql
UPDATE notes
SET title = 'PostgreSQL 学习',
    updated_at = NOW()
WHERE id = 10
  AND user_id = 1
RETURNING *;
```

删除：

```sql
DELETE FROM notes
WHERE id = 10
  AND user_id = 1;
```

在 `WHERE` 中带上 `user_id` 很重要，否则只知道笔记 ID 的用户可能修改别人的笔记。

### 9.4 JOIN

```sql
SELECT
  n.id,
  n.title,
  c.name AS category_name
FROM notes AS n
LEFT JOIN categories AS c
  ON c.id = n.category_id
WHERE n.user_id = $1;
```

可以把 JOIN 理解为按照共同字段，把两张表的相关行“拼起来”。

### 9.5 聚合

```sql
SELECT
  category_id,
  COUNT(*) AS note_count
FROM notes
WHERE user_id = $1
GROUP BY category_id
ORDER BY note_count DESC;
```

---

## 10. PostgreSQL 数据建模

### 10.1 NoteFlow 基础表

```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  nickname VARCHAR(50) NOT NULL,
  password_hash TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE categories (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(50) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (user_id, name)
);

CREATE TABLE notes (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id BIGINT REFERENCES categories(id) ON DELETE SET NULL,
  title VARCHAR(100) NOT NULL,
  content TEXT NOT NULL DEFAULT '',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 10.2 主键与外键

- 主键像每个人唯一的身份证号。
- 外键像档案中引用另一个人的身份证号。

外键能防止出现“笔记属于一个根本不存在的用户”这种数据。

### 10.3 一对一、一对多、多对多

- 一个用户有一个个人设置：一对一。
- 一个用户有多条笔记：一对多。
- 一条笔记有多个标签，一个标签属于多条笔记：多对多。

多对多需要中间表：

```sql
CREATE TABLE tags (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(30) NOT NULL
);

CREATE TABLE note_tags (
  note_id BIGINT NOT NULL REFERENCES notes(id) ON DELETE CASCADE,
  tag_id BIGINT NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (note_id, tag_id)
);
```

### 10.4 规范化与反规范化

规范化倾向于减少重复数据。例如分类名称只存一份，通过 `category_id` 引用。

反规范化会为了查询性能或历史记录，故意保存部分重复数据。例如订单中保存购买时的商品名称和价格，因为商品之后可能改名或涨价。

原则：

> 先建立清晰、正确的数据模型，再根据真实性能数据决定是否反规范化。

### 10.5 时间字段

推荐明确保存：

- `created_at`
- `updated_at`
- 软删除时的 `deleted_at`

跨时区系统通常在数据库保存带时区的时间或统一 UTC，在显示层转换为用户时区。

---

## 11. 索引、事务与并发控制

### 11.1 索引

没有索引时，数据库查找可能像从整本书第一页逐行寻找一个名字；索引像书末的关键词目录。

```sql
CREATE INDEX idx_notes_user_created_at
ON notes (user_id, created_at DESC);
```

这个索引适合：

```sql
SELECT *
FROM notes
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

索引不是越多越好：

- 索引占用空间。
- 写入时需要维护索引。
- 不符合查询方式的索引可能用不上。

使用 `EXPLAIN ANALYZE` 查看执行计划：

```sql
EXPLAIN ANALYZE
SELECT *
FROM notes
WHERE user_id = 1
ORDER BY created_at DESC
LIMIT 20;
```

### 11.2 事务

转账需要两步：

1. A 扣款。
2. B 加款。

只完成一步会造成资金错误，因此两步必须作为整体成功或整体失败。

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1
  AND balance >= 100;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

发生异常时：

```sql
ROLLBACK;
```

事务常用 ACID 描述：

- Atomicity：原子性，要么全做，要么全不做。
- Consistency：一致性，数据始终符合约束。
- Isolation：隔离性，并发事务互不产生不合理影响。
- Durability：持久性，提交后不会轻易丢失。

### 11.3 并发问题

假设库存只剩 1：

```text
请求 A 读到库存 1
请求 B 读到库存 1
A 下单并把库存改为 0
B 也下单并把库存改为 0
```

结果卖出了两件，这叫竞态条件。

更安全的 SQL：

```sql
UPDATE products
SET stock = stock - 1
WHERE id = $1
  AND stock > 0
RETURNING stock;
```

如果返回 0 行，说明库存不足。把“检查”和“修改”合并为一次原子操作，比先查再改更可靠。

### 11.4 乐观锁

```sql
UPDATE notes
SET content = $1,
    version = version + 1
WHERE id = $2
  AND version = $3;
```

如果受影响行数为 0，说明数据已经被别人修改，客户端可以提示冲突。

---

## 12. ORM、Prisma 与原生 SQL

### 12.1 ORM 是什么

ORM 把数据库中的表和代码中的对象建立映射。

原生 SQL：

```ts
await db.query(
  "SELECT * FROM notes WHERE user_id = $1 ORDER BY created_at DESC",
  [userId],
);
```

Prisma：

```ts
await prisma.note.findMany({
  where: { userId },
  orderBy: { createdAt: "desc" },
});
```

### 12.2 ORM 的优点

- 类型提示。
- 常见 CRUD 开发快。
- 迁移工具完善。
- 降低简单 SQL 的重复代码。

### 12.3 ORM 的缺点

- 复杂查询仍然需要理解 SQL。
- 不理解生成 SQL 时容易出现性能问题。
- 某些数据库特性难以完整表达。
- 错误使用可能触发 N+1 查询。

### 12.4 Prisma 模型示例

```prisma
model User {
  id           BigInt   @id @default(autoincrement())
  email        String   @unique
  nickname     String
  passwordHash String
  notes        Note[]
  createdAt    DateTime @default(now())
}

model Note {
  id        BigInt   @id @default(autoincrement())
  userId    BigInt
  title     String
  content   String   @default("")
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([userId, createdAt])
}
```

### 12.5 正确态度

不要把 ORM 当作不学 SQL 的理由。推荐：

1. 先掌握基础 SQL。
2. 用 ORM 提高日常开发效率。
3. 遇到复杂统计、性能问题时检查实际 SQL。

---

## 13. 登录、Session、JWT 与权限控制

### 13.1 Authentication 与 Authorization

- Authentication：你是谁？
- Authorization：你能做什么？

进入学校要验证学生身份，这是认证；只有老师能修改成绩，这是授权。

### 13.2 密码存储

绝对不能明文保存密码。

```ts
import bcrypt from "bcrypt";

const saltRounds = 12;

// 注册时保存哈希，而不是保存原密码。
const passwordHash = await bcrypt.hash(password, saltRounds);

// 登录时比较用户输入与数据库哈希。
const isValid = await bcrypt.compare(password, passwordHash);
```

哈希不是加密：

- 加密通常可以用密钥解密。
- 安全密码哈希设计为不可逆，并包含盐和计算成本。

### 13.3 Session

Session 模式大致流程：

1. 用户登录。
2. 服务器创建 Session。
3. Session 数据保存在服务器或 Redis。
4. 浏览器保存 Session ID Cookie。
5. 后续请求携带 Cookie。

优点：

- 服务端可以立即让 Session 失效。
- 敏感状态主要由服务端控制。

缺点：

- 多实例部署时需要共享 Session。
- 移动端和跨域场景需要正确处理 Cookie。

### 13.4 JWT

JWT 通常由三部分组成：

```text
Header.Payload.Signature
```

JWT 的 Payload 只是编码，不是加密。不要放入密码、身份证号等敏感信息。

```ts
import jwt from "jsonwebtoken";

const accessToken = jwt.sign(
  {
    sub: user.id,      // sub 表示主体，通常是用户 ID。
    role: user.role,   // 可放少量鉴权需要的信息。
  },
  process.env.JWT_SECRET!,
  {
    expiresIn: "15m",  // Access Token 应设置较短有效期。
    issuer: "noteflow-api",
  },
);
```

验证：

```ts
function authMiddleware(req: Request, _res: Response, next: NextFunction) {
  const header = req.headers.authorization;

  if (!header?.startsWith("Bearer ")) {
    return next(new AppError("UNAUTHORIZED", 401, "请先登录"));
  }

  const token = header.slice("Bearer ".length);

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = { id: String(payload.sub) };
    next();
  } catch {
    next(new AppError("INVALID_TOKEN", 401, "登录状态已失效"));
  }
}
```

### 13.5 Access Token 与 Refresh Token

常见方案：

- Access Token：有效期短，用于访问 API。
- Refresh Token：有效期长，只用于换取新的 Access Token。

Refresh Token 应该支持：

- 数据库存储哈希。
- 主动撤销。
- 设备识别。
- 轮换。
- 异常重复使用检测。

### 13.6 资源级权限

错误代码：

```ts
// 只按 noteId 查找，没有确认笔记所有者。
await noteRepository.deleteById(noteId);
```

更安全：

```ts
// 同时匹配 noteId 和当前 userId。
const deleted = await noteRepository.deleteByIdAndUserId(noteId, req.user.id);

if (!deleted) {
  throw new AppError("NOTE_NOT_FOUND", 404, "笔记不存在");
}
```

即使用户已经登录，也不能默认他有权操作任意数据。

---

## 14. 参数校验、异常处理与统一响应

### 14.1 为什么必须校验

前端校验主要改善用户体验，后端校验才是真正的安全边界。攻击者可以绕开 App，直接调用 API。

### 14.2 使用 Zod

```ts
import { z } from "zod";

export const createNoteSchema = z.object({
  title: z
    .string()
    .trim()
    .min(1, "标题不能为空")
    .max(100, "标题不能超过 100 个字符"),
  content: z.string().max(100_000, "内容过长").default(""),
  categoryId: z.coerce.number().int().positive().optional(),
});
```

校验中间件：

```ts
function validateBody(schema: z.ZodSchema) {
  return (req: Request, _res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return next(
        new AppError(
          "VALIDATION_ERROR",
          422,
          "请求数据不合法",
          result.error.flatten(),
        ),
      );
    }

    // 使用经过解析和清洗的数据覆盖原请求体。
    req.body = result.data;
    next();
  };
}
```

### 14.3 自定义错误类

```ts
export class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly statusCode: number,
    message: string,
    public readonly details?: unknown,
  ) {
    super(message);
  }
}
```

### 14.4 全局错误处理

```ts
function errorMiddleware(
  error: unknown,
  _req: Request,
  res: Response,
  _next: NextFunction,
) {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      success: false,
      error: {
        code: error.code,
        message: error.message,
        details: error.details,
      },
    });
  }

  // 未知错误写入服务端日志，但不要把堆栈直接返回用户。
  console.error(error);

  return res.status(500).json({
    success: false,
    error: {
      code: "INTERNAL_SERVER_ERROR",
      message: "服务器内部错误",
    },
  });
}
```

### 14.5 统一响应并非越统一越好

统一结构方便客户端处理，但不要为了“所有接口完全一样”破坏 HTTP 语义。例如文件下载、流式响应、204 响应不必强行套 JSON。

---

## 15. 后端安全基础

### 15.1 永远不信任客户端

客户端提交：

```json
{
  "userId": 999,
  "role": "admin"
}
```

后端不能因为请求里写了 `role: admin` 就相信。身份信息应该来自已经验证的 Session 或 Token。

### 15.2 SQL 注入

危险写法：

```ts
// 用户输入被直接拼进 SQL，可能改变 SQL 原本含义。
const sql = `SELECT * FROM users WHERE email = '${email}'`;
await db.query(sql);
```

安全写法：

```ts
// 参数化查询让数据库把 email 当成数据，而不是 SQL 代码。
await db.query("SELECT * FROM users WHERE email = $1", [email]);
```

### 15.3 CORS

CORS 是浏览器安全策略的一部分。它主要限制网页脚本跨源读取响应，不是服务器防火墙。

```ts
import cors from "cors";

app.use(
  cors({
    // 生产环境不要无条件允许所有来源。
    origin: ["https://example.com"],
    credentials: true,
  }),
);
```

React Native 原生请求通常不受浏览器 CORS 同样的限制，但后端仍要进行鉴权。

### 15.4 CSRF

如果身份凭证由浏览器自动携带的 Cookie 保存，攻击网站可能诱导浏览器向目标网站发送请求。

常见防护：

- SameSite Cookie。
- CSRF Token。
- 检查 Origin/Referer。
- 对敏感操作要求再次验证。

### 15.5 XSS

如果用户笔记支持 HTML 渲染，攻击者可能保存恶意脚本。后端和前端需要：

- 对 HTML 做安全清洗。
- 默认转义用户内容。
- 使用内容安全策略。
- 不直接执行用户输入。

### 15.6 限流

```ts
import rateLimit from "express-rate-limit";

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 统计最近 15 分钟。
  limit: 20,                // 单个来源最多尝试 20 次。
  standardHeaders: true,
});

app.use("/api/auth/login", loginLimiter);
```

分布式部署时，限流计数应放在 Redis 等共享存储中。

### 15.7 敏感配置

不要提交：

- 数据库密码
- JWT 密钥
- 云服务密钥
- SMTP 密码
- MinIO Secret Key

```env
DATABASE_URL=postgresql://user:password@postgres:5432/noteflow
JWT_SECRET=replace-with-a-long-random-secret
```

`.env` 应放进 `.gitignore`，仓库只提交 `.env.example`。

### 15.8 安全检查表

- 密码是否使用安全哈希？
- Token 是否有有效期？
- 是否验证资源所有权？
- 是否使用参数化 SQL？
- 是否限制上传文件大小和类型？
- 是否启用 HTTPS？
- 是否记录登录异常？
- 是否定期更新依赖？
- 错误响应是否泄漏堆栈和数据库结构？
- 数据库端口是否只对必要网络开放？

---

## 16. Redis 与缓存

### 16.1 Redis 是什么

Redis 常把数据放在内存中，因此读写很快。常见用途：

- 缓存
- Session
- 验证码
- 限流
- 分布式锁
- 排行榜
- 消息队列的部分场景

Redis 不是 PostgreSQL 的简单替代品。

### 16.2 Cache Aside

```text
请求数据
  ↓
先查 Redis
  ├─ 命中 → 直接返回
  └─ 未命中 → 查 PostgreSQL → 写入 Redis → 返回
```

```ts
async function getUserProfile(userId: string) {
  const key = `user:profile:${userId}`;
  const cached = await redis.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  const user = await userRepository.findById(userId);

  if (user) {
    // 设置过期时间，避免旧数据永久存在。
    await redis.set(key, JSON.stringify(user), { EX: 300 });
  }

  return user;
}
```

更新用户信息后要处理缓存：

```ts
await userRepository.update(userId, input);

// 简单可靠的方案是删除旧缓存，下次读取时重新生成。
await redis.del(`user:profile:${userId}`);
```

### 16.3 缓存问题

#### 缓存穿透

不断查询不存在的数据，缓存永远不命中。

解决思路：

- 缓存短时间的空结果。
- 参数校验。
- 布隆过滤器。

#### 缓存击穿

一个热点 Key 刚好过期，大量请求同时查询数据库。

解决思路：

- 互斥锁。
- 逻辑过期。
- 提前刷新热点数据。

#### 缓存雪崩

大量 Key 同时过期，数据库压力瞬间增大。

解决思路：

- 给过期时间加入随机值。
- 多级缓存。
- 限流和降级。

### 16.4 不要过早缓存

先确认：

1. 哪个接口真的慢？
2. 慢在数据库、网络还是代码？
3. 数据允许多久不一致？
4. 缓存失效策略是什么？

没有测量就加缓存，可能只是增加系统复杂度。

---

## 17. 文件上传、对象存储与 MinIO

### 17.1 为什么不直接把大文件放数据库

头像、附件、图片通常放在对象存储，数据库保存：

- 文件 ID
- 对象 Key
- 大小
- MIME 类型
- 所属用户
- 创建时间

对象存储可以理解为一个按 Key 取文件的巨大文件仓库。

### 17.2 上传流程

```text
客户端选择图片
     ↓
后端校验身份、类型、大小
     ↓
上传到 MinIO / S3
     ↓
数据库保存文件元数据
     ↓
返回文件访问地址或文件 ID
```

### 17.3 文件名不要直接相信

用户上传的原始文件名可能：

- 重名
- 包含特殊字符
- 伪装扩展名
- 带路径字符

推荐生成自己的对象 Key：

```ts
import { randomUUID } from "node:crypto";

function createAvatarObjectKey(userId: string, extension: string) {
  // 服务端生成不可预测且不会重名的文件名。
  return `avatars/${userId}/${randomUUID()}.${extension}`;
}
```

### 17.4 预签名 URL

大文件上传可以让后端生成短期有效的预签名 URL，客户端直接上传对象存储：

```text
客户端 → 请求上传许可 → 后端
客户端 ← 预签名 URL ← 后端
客户端 → 直接上传文件 → 对象存储
客户端 → 提交文件元数据 → 后端
```

这样可以减少后端转发大文件的压力，但仍要验证文件归属和上传结果。

### 17.5 文件安全

- 限制大小。
- 检查 MIME 类型和真实文件内容。
- 图片重新编码。
- 私有文件不要设置公开桶。
- 下载时验证权限。
- 病毒扫描。
- 预签名地址设置短有效期。

---

## 18. 定时任务、消息队列与异步处理

### 18.1 为什么需要异步任务

注册接口如果同步完成下面所有操作：

1. 创建用户。
2. 发送欢迎邮件。
3. 生成头像。
4. 写分析日志。
5. 同步第三方系统。

响应会变慢，而且任一非核心步骤失败都可能导致注册失败。

更合理：

```text
注册请求
  ↓
创建用户（核心）
  ↓
发布 user.created 消息
  ↓
立即返回成功

后台 Worker：
- 发送欢迎邮件
- 初始化默认分类
- 记录分析事件
```

### 18.2 消息队列的比喻

消息队列像快餐店的订单屏：

- 生产者：收银台，把订单放进去。
- 队列：等待制作的订单。
- 消费者：后厨，逐个处理订单。
- ACK：后厨确认订单完成。
- 重试：制作失败时重新处理。
- 死信队列：多次失败后移到专门区域检查。

### 18.3 常见工具

- RabbitMQ：传统消息队列，路由能力丰富。
- Kafka：高吞吐事件流。
- Redis Streams：基于 Redis 的流式队列。
- BullMQ：Node.js 中常用的 Redis 任务队列。

### 18.4 BullMQ 示例

生产任务：

```ts
import { Queue } from "bullmq";

const emailQueue = new Queue("email", {
  connection: { host: "redis", port: 6379 },
});

await emailQueue.add(
  "send-welcome-email",
  {
    userId: user.id,
    email: user.email,
  },
  {
    attempts: 3, // 最多尝试三次。
    backoff: {
      type: "exponential",
      delay: 1000,
    },
  },
);
```

消费任务：

```ts
import { Worker } from "bullmq";

new Worker(
  "email",
  async (job) => {
    if (job.name === "send-welcome-email") {
      await emailService.sendWelcome(job.data.email);
    }
  },
  {
    connection: { host: "redis", port: 6379 },
  },
);
```

### 18.5 幂等性

任务重试时，同一消息可能执行多次。发送优惠券、扣款等操作必须避免重复执行。

可以保存唯一业务 ID：

```sql
INSERT INTO processed_events (event_id, processed_at)
VALUES ($1, NOW())
ON CONFLICT (event_id) DO NOTHING;
```

如果插入失败，说明事件已经处理。

### 18.6 定时任务

常见场景：

- 每天备份数据库。
- 清理过期验证码。
- 每周发送统计报告。
- 定期刷新缓存。

单机可以使用 cron；多实例环境要避免每台机器都重复执行同一个任务。

---

## 19. 日志、监控与可观测性

### 19.1 三大支柱

- Logs：发生了什么。
- Metrics：系统整体表现如何。
- Traces：一次请求经过了哪些服务。

### 19.2 结构化日志

不推荐：

```ts
console.log("用户创建笔记失败");
```

更适合生产环境：

```ts
logger.error({
  event: "note.create.failed",
  requestId: req.id,
  userId: req.user.id,
  errorCode: error.code,
  // 不要记录密码、完整 Token 等敏感数据。
});
```

JSON 日志更容易被日志系统检索。

### 19.3 Request ID

一次请求可能经过 Nginx、API、数据库、消息队列。Request ID 像包裹追踪号，帮助串起所有日志。

```ts
import { randomUUID } from "node:crypto";

function requestIdMiddleware(req: Request, res: Response, next: NextFunction) {
  const requestId = req.header("x-request-id") ?? randomUUID();

  req.id = requestId;
  res.setHeader("x-request-id", requestId);
  next();
}
```

### 19.4 关键指标

建议监控：

- 请求量
- 错误率
- P50/P95/P99 延迟
- CPU 和内存
- 数据库连接数
- 慢查询数量
- Redis 命中率
- 队列积压量
- 磁盘空间

P95 延迟 800ms 表示 95% 的请求在 800ms 内完成，剩余 5% 更慢。平均值可能掩盖少量特别慢的请求。

### 19.5 健康检查

```ts
app.get("/health/live", (_req, res) => {
  // 只要进程能响应，就认为存活。
  res.json({ status: "ok" });
});

app.get("/health/ready", async (_req, res) => {
  try {
    // 就绪检查确认关键依赖可用。
    await db.query("SELECT 1");
    res.json({ status: "ready" });
  } catch {
    res.status(503).json({ status: "not_ready" });
  }
});
```

---

## 20. 后端测试

### 20.1 为什么需要测试

测试不是证明系统绝对没有错误，而是：

- 防止旧功能被新改动破坏。
- 让重构更有底气。
- 记录预期行为。
- 减少重复手工检查。

### 20.2 测试金字塔

```text
        少量 E2E 测试
      适量集成测试
    大量快速单元测试
```

### 20.3 单元测试

```ts
import { describe, expect, it } from "vitest";
import { normalizeTitle } from "./normalize-title";

describe("normalizeTitle", () => {
  it("应去除标题两侧空格", () => {
    expect(normalizeTitle("  后端学习  ")).toBe("后端学习");
  });

  it("标题为空时应抛出错误", () => {
    expect(() => normalizeTitle("   ")).toThrow("标题不能为空");
  });
});
```

### 20.4 Service 测试

```ts
it("用户不能删除别人的笔记", async () => {
  const repository = {
    findById: vi.fn().mockResolvedValue({
      id: 10,
      userId: 2,
    }),
    delete: vi.fn(),
  };

  const service = createNoteService(repository);

  await expect(
    service.deleteNote({
      noteId: 10,
      currentUserId: 1,
    }),
  ).rejects.toMatchObject({ code: "NOTE_NOT_FOUND" });

  // 没有权限时，Repository 的删除方法不应该被调用。
  expect(repository.delete).not.toHaveBeenCalled();
});
```

### 20.5 API 集成测试

```ts
import request from "supertest";

it("POST /api/notes 应创建笔记", async () => {
  const response = await request(app)
    .post("/api/notes")
    .set("Authorization", `Bearer ${testAccessToken}`)
    .send({
      title: "测试笔记",
      content: "集成测试内容",
    });

  expect(response.status).toBe(201);
  expect(response.body.data.title).toBe("测试笔记");
});
```

### 20.6 测试什么

至少覆盖：

- 正常流程。
- 参数为空或类型错误。
- 未登录。
- 没有资源权限。
- 数据不存在。
- 数据库异常。
- 重复提交。
- 并发修改。
- Token 过期。

---

## 21. Docker、Nginx 与生产部署

### 21.1 开发环境与生产环境

本地能运行不代表生产环境可靠。生产环境还需要：

- 环境变量管理
- 进程重启
- 日志
- HTTPS
- 域名
- 反向代理
- 数据备份
- 防火墙
- 监控
- 灰度与回滚

### 21.2 Docker 的比喻

传统部署像告诉别人：

> 你去买锅、买炉子、买调料，再按我的方法做。

Docker 像把应用、运行环境和依赖打包成标准集装箱，到不同服务器都用相似方式运行。

### 21.3 Node.js Dockerfile

```dockerfile
# 构建阶段：安装依赖并编译 TypeScript。
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY tsconfig.json ./
COPY src ./src
RUN npm run build

# 运行阶段：只保留生产所需内容。
FROM node:22-alpine AS runner

WORKDIR /app
ENV NODE_ENV=production

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

USER node
EXPOSE 3000

CMD ["node", "dist/server.js"]
```

> 实际使用时请根据项目支持的 Node.js 版本选择镜像，不要只因为版本号更大就直接升级生产环境。

### 21.4 Docker Compose

```yaml
services:
  api:
    build: ./apps/api
    restart: unless-stopped
    env_file:
      - .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - noteflow

  postgres:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_DB: noteflow
      POSTGRES_USER: noteflow
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      # 数据必须挂载到持久化卷，否则重建容器可能丢失。
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U noteflow -d noteflow"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - noteflow

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    networks:
      - noteflow

volumes:
  postgres_data:

networks:
  noteflow:
```

### 21.5 Nginx 反向代理

```nginx
server {
    listen 443 ssl;
    server_name api.example.com;

    # 证书路径根据实际环境配置。
    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location /api/ {
        proxy_pass http://127.0.0.1:3000;

        # 把客户端和协议信息传给后端。
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 21.6 为什么使用 Nginx

- TLS/HTTPS 终止。
- 反向代理。
- 静态文件。
- 压缩。
- 请求大小限制。
- 简单负载均衡。
- 超时控制。
- 统一入口。

### 21.7 数据库备份

```bash
#!/usr/bin/env bash
set -euo pipefail

# 使用时间戳区分不同备份。
backup_time=$(date +"%Y-%m-%d_%H-%M-%S")
backup_file="/backups/noteflow_${backup_time}.dump"

pg_dump \
  --format=custom \
  --dbname="$DATABASE_URL" \
  --file="$backup_file"

# 删除 14 天前的旧备份，避免磁盘被占满。
find /backups -type f -name "noteflow_*.dump" -mtime +14 -delete
```

备份的真正标准不是“生成了文件”，而是“定期验证能够恢复”。

### 21.8 生产端口原则

通常只对公网开放：

- 22：SSH，最好限制来源并使用密钥。
- 80：HTTP，用于跳转 HTTPS。
- 443：HTTPS。

PostgreSQL、Redis、MinIO 管理端口一般不要直接暴露公网。

---

## 22. 性能优化与高并发入门

### 22.1 性能优化顺序

推荐顺序：

1. 测量。
2. 找到瓶颈。
3. 提出假设。
4. 小范围修改。
5. 再次测量。

不要凭感觉随意优化。

### 22.2 常见瓶颈

- SQL 没有索引。
- 一次返回太多数据。
- N+1 查询。
- 数据库连接池过小或过大。
- 串行执行可并行任务。
- CPU 密集代码阻塞事件循环。
- 调用第三方接口过慢。
- 日志同步写入过多。
- 大文件经过 API 中转。

### 22.3 分页

Offset 分页：

```sql
SELECT *
FROM notes
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;
```

简单，但很深的页可能变慢，并且数据插入时可能重复或遗漏。

游标分页：

```sql
SELECT *
FROM notes
WHERE user_id = $1
  AND (created_at, id) < ($2, $3)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

适合无限滚动和大量数据。

### 22.4 N+1 查询

错误思路：

```text
查询 100 条笔记：1 次 SQL
为每条笔记查询分类：100 次 SQL
总计：101 次 SQL
```

可以使用 JOIN、批量 `IN` 查询或 ORM 的正确预加载方式。

### 22.5 数据库连接池

建立数据库连接有成本，连接池会复用连接。

```ts
import { Pool } from "pg";

export const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                     // 不要盲目设置成非常大的数。
  idleTimeoutMillis: 30_000,
  connectionTimeoutMillis: 5_000,
});
```

如果有 10 个 API 实例，每个最大 100 个连接，数据库可能面对 1000 个连接。因此连接池要结合实例数和数据库能力设置。

### 22.6 超时、重试与熔断

调用第三方服务必须设置超时：

```ts
const response = await fetch(url, {
  signal: AbortSignal.timeout(5000),
});
```

重试只适合：

- 临时网络错误。
- 明确可以安全重复的操作。
- 有退避策略。

支付、创建订单等非幂等操作不能直接无脑重试。

### 22.7 限流与降级

系统压力过大时：

- 限制低优先级请求。
- 暂时关闭昂贵的统计功能。
- 返回缓存数据。
- 排队处理。
- 对不同用户设置配额。

高并发不是“让所有请求都进来”，而是“在容量范围内保持核心服务可靠”。

### 22.8 水平扩展

```text
               ┌→ API 实例 1
客户端 → 负载均衡 ├→ API 实例 2 → PostgreSQL / Redis
               └→ API 实例 3
```

无状态 API 更容易水平扩展。Session、缓存、上传文件不能只存在某一台 API 机器的本地内存或磁盘中。

---

## 23. 单体、模块化单体与微服务

### 23.1 单体应用

所有功能部署在一个应用中。

优点：

- 开发和部署简单。
- 本地调试方便。
- 数据一致性容易处理。
- 对小团队成本低。

缺点：

- 模块边界差时容易变成“大泥球”。
- 整体发布。
- 某个模块故障可能影响全局。

### 23.2 模块化单体

仍然部署为一个应用，但代码内部保持明确模块边界：

```text
modules/
├── users/
├── auth/
├── notes/
├── categories/
├── files/
└── notifications/
```

这是很多初创项目和个人项目很合适的选择。

### 23.3 微服务

把业务拆成独立服务，例如：

- 用户服务
- 笔记服务
- 文件服务
- 通知服务
- 搜索服务

微服务会带来：

- 网络调用
- 服务发现
- 分布式事务
- 链路追踪
- 独立部署
- 消息一致性
- 更复杂的测试和运维

### 23.4 什么时候不要微服务

如果：

- 团队只有一两个人。
- 业务边界还不稳定。
- 单体性能没有成为瓶颈。
- 没有成熟监控和自动部署。

那么微服务通常只会增加负担。

推荐 NoteFlow 先使用模块化单体。等搜索、AI、文件处理等模块真的需要独立扩缩容时，再拆服务。

### 23.5 分布式系统基本现实

网络调用可能：

- 超时。
- 重复。
- 乱序。
- 部分成功。
- 返回结果丢失。

因此分布式系统需要更多：

- 幂等
- 重试
- 超时
- 熔断
- 补偿
- 最终一致性
- 可观测性

---

## 24. Java、Python、Go 后端快速认识

这一章用于建立多元视角，不要求现在同时深入三种语言。

### 24.1 Java + Spring Boot

```java
@RestController
@RequestMapping("/api/notes")
public class NoteController {

    private final NoteService noteService;

    // 推荐使用构造器注入依赖，方便测试和维护。
    public NoteController(NoteService noteService) {
        this.noteService = noteService;
    }

    @GetMapping
    public List<NoteResponse> listNotes(
            @AuthenticationPrincipal UserPrincipal user) {
        // 当前用户身份来自认证系统，不从请求参数中相信 userId。
        return noteService.listByUserId(user.getId());
    }
}
```

Spring Boot 常见优势：

- 依赖注入体系成熟。
- 数据库、认证、监控等生态完整。
- 编译期类型检查强。
- 大型团队规范丰富。

需要学习：

- Java 语法与面向对象
- Maven/Gradle
- Spring IoC
- Spring MVC
- Spring Data JPA
- Spring Security
- JVM 基础

### 24.2 Python + FastAPI

```python
from fastapi import Depends, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class CreateNoteRequest(BaseModel):
    # Pydantic 会自动校验长度和数据类型。
    title: str = Field(min_length=1, max_length=100)
    content: str = Field(default="", max_length=100_000)

@app.post("/api/notes", status_code=201)
async def create_note(
    body: CreateNoteRequest,
    current_user: User = Depends(get_current_user),
):
    # Depends 用于注入当前用户等依赖。
    return await note_service.create(
        user_id=current_user.id,
        title=body.title,
        content=body.content,
    )
```

FastAPI 常见优势：

- 语法清晰。
- 类型提示与自动接口文档。
- 与 AI、数据处理生态结合方便。

适合把 NoteFlow 的 AI 摘要、OCR、向量检索等功能做成独立 Python 服务。

### 24.3 Go + Gin

```go
type CreateNoteRequest struct {
    Title   string `json:"title" binding:"required,max=100"`
    Content string `json:"content"`
}

func CreateNote(c *gin.Context) {
    var body CreateNoteRequest

    // ShouldBindJSON 会解析并校验 JSON 请求。
    if err := c.ShouldBindJSON(&body); err != nil {
        c.JSON(http.StatusUnprocessableEntity, gin.H{
            "message": "请求数据不合法",
        })
        return
    }

    userID := c.GetString("userId")
    note, err := noteService.Create(c.Request.Context(), userID, body)

    if err != nil {
        handleError(c, err)
        return
    }

    c.JSON(http.StatusCreated, note)
}
```

Go 常见优势：

- 编译成单个二进制文件。
- 启动快、部署方便。
- goroutine 并发模型易于上手。
- 云原生生态丰富。

### 24.4 用同一功能比较语言

最好的多语言学习方法不是分别背语法，而是用同一个小项目实现：

- `POST /notes`
- `GET /notes`
- PostgreSQL
- 参数校验
- 统一错误
- 测试
- Docker

然后比较：

- 类型系统。
- 异步/并发方式。
- 依赖管理。
- 项目结构。
- 启动速度。
- 开发体验。

---

## 25. NoteFlow 后端完整案例

### 25.1 目标架构

```text
React Native + Expo
        ↓ HTTPS
Nginx / Cloudflare
        ↓
Node.js API（Express 或 NestJS）
   ├── Auth
   ├── Users
   ├── Notes
   ├── Categories
   ├── Files
   └── Sync
        ↓
PostgreSQL + Redis + MinIO
```

### 25.2 第一阶段：最小可用版本

功能：

- 邮箱注册
- 登录
- 创建笔记
- 查询笔记
- 修改笔记
- 删除笔记
- 分类

先不要加入：

- AI 总结
- 多人协作
- 复杂富文本
- 微服务
- Kubernetes

### 25.3 API 设计

```text
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/refresh
POST   /api/auth/logout

GET    /api/users/me
PATCH  /api/users/me

GET    /api/notes
POST   /api/notes
GET    /api/notes/:id
PATCH  /api/notes/:id
DELETE /api/notes/:id

GET    /api/categories
POST   /api/categories
PATCH  /api/categories/:id
DELETE /api/categories/:id
```

### 25.4 创建笔记数据流

```text
CreateNoteScreen
  ↓
useCreateNote()
  ↓
noteService.create()
  ↓
POST /api/notes
  ↓
authMiddleware
  ↓
validateBody
  ↓
noteController
  ↓
noteService
  ↓
noteRepository
  ↓
PostgreSQL
```

每一层只承担自己的职责，出现问题时也更容易定位。

### 25.5 创建笔记接口示例

路由：

```ts
router.post(
  "/",
  authMiddleware,
  validateBody(createNoteSchema),
  asyncHandler(createNoteController),
);
```

Controller：

```ts
export async function createNoteController(req: Request, res: Response) {
  const note = await noteService.create({
    userId: req.user.id,
    title: req.body.title,
    content: req.body.content,
    categoryId: req.body.categoryId,
  });

  return res.status(201).json({
    success: true,
    data: note,
  });
}
```

Service：

```ts
export async function create(input: CreateNoteInput) {
  if (input.categoryId) {
    // 必须确认分类属于当前用户。
    const category = await categoryRepository.findOwnedCategory(
      input.categoryId,
      input.userId,
    );

    if (!category) {
      throw new AppError("CATEGORY_NOT_FOUND", 404, "分类不存在");
    }
  }

  return noteRepository.create(input);
}
```

### 25.6 搜索

初期可以用 PostgreSQL：

```sql
SELECT id, title, content, created_at
FROM notes
WHERE user_id = $1
  AND (
    title ILIKE '%' || $2 || '%'
    OR content ILIKE '%' || $2 || '%'
  )
ORDER BY updated_at DESC
LIMIT 50;
```

数据量增加后再学习：

- PostgreSQL Full Text Search
- `pg_trgm`
- Elasticsearch/OpenSearch
- 向量检索

不要在数据还很少时为了“高级”立即引入 Elasticsearch。

### 25.7 离线同步

移动端离线同步比普通 CRUD 更复杂。需要考虑：

- 本地生成临时 ID 还是 UUID。
- 网络恢复后按什么顺序同步。
- 同一笔记在两台设备都修改怎么办。
- 删除操作如何同步。
- 重试是否会重复创建。

可给笔记增加：

```sql
ALTER TABLE notes
ADD COLUMN version INTEGER NOT NULL DEFAULT 1,
ADD COLUMN deleted_at TIMESTAMPTZ,
ADD COLUMN client_updated_at TIMESTAMPTZ;
```

更新时使用版本号：

```sql
UPDATE notes
SET title = $1,
    content = $2,
    version = version + 1,
    updated_at = NOW()
WHERE id = $3
  AND user_id = $4
  AND version = $5
RETURNING *;
```

更新失败时返回冲突，让同步层决定：

- 服务器优先。
- 客户端优先。
- 最后写入优先。
- 保留两个版本。
- 让用户手动合并。

### 25.8 文件与头像

- 文件存入 MinIO。
- 数据库保存对象 Key。
- 私有笔记附件需要鉴权。
- 头像可以生成缩略图。
- 删除用户时设计对象清理任务。

### 25.9 Redis 用途

NoteFlow 中 Redis 可以用于：

- 邮箱验证码。
- Refresh Token 状态。
- 登录限流。
- 用户信息短期缓存。
- 异步任务队列。

不要为了“用了 Redis”而把每条笔记都缓存。先根据访问热点和性能数据决定。

### 25.10 发布前清单

- 注册登录测试完成。
- 用户只能访问自己的笔记。
- 数据库迁移可重复执行。
- 敏感配置未提交 Git。
- HTTPS 有效。
- 定期备份并测试恢复。
- 服务器只开放必要端口。
- 日志不会记录密码和完整 Token。
- 崩溃后服务能自动重启。
- API 有健康检查。
- 重要接口有限流。
- 隐私政策和账号删除流程明确。

---

## 26. 由浅入深的练习项目

### 项目 1：内存版待办 API

技术：

- TypeScript
- Express
- 数组

功能：

- 添加待办
- 查询待办
- 修改完成状态
- 删除待办

重点：

- HTTP 方法
- 路由参数
- 请求体
- 状态码

### 项目 2：带数据库的记账 API

技术：

- Express
- PostgreSQL
- SQL

功能：

- 收入和支出 CRUD
- 分类
- 月度统计
- 分页

重点：

- 数据建模
- 聚合查询
- 参数化 SQL
- 事务

### 项目 3：用户系统

功能：

- 注册
- 登录
- JWT
- Refresh Token
- 修改密码
- 忘记密码
- 登录限流

重点：

- 安全
- Token 生命周期
- 邮件服务
- 权限测试

### 项目 4：文件分享服务

功能：

- 文件上传
- 临时分享链接
- 下载权限
- 到期清理

重点：

- MinIO/S3
- 预签名 URL
- 文件校验
- 定时任务

### 项目 5：实时聊天室

技术：

- WebSocket
- Redis Pub/Sub
- PostgreSQL

重点：

- 长连接
- 在线状态
- 消息顺序
- 多实例广播

### 项目 6：NoteFlow 完整后端

重点：

- 模块化架构
- 测试
- 部署
- 缓存
- 文件
- 离线同步
- 监控

### 项目 7：多语言重写

选择 NoteFlow 的一个小模块，分别用：

- Spring Boot
- FastAPI
- Go/Gin

实现相同 API，以比较生态与思维方式。

---

## 27. 24 周学习路线

每天建议 1～2 小时。时间不足时可以把每周拉长，不要为了赶进度跳过实践。

### 第 1～2 周：语言与工具

学习：

- TypeScript 基础
- 异步编程
- Node.js 模块
- npm
- Git
- 命令行

产出：

- 命令行笔记管理工具
- 每天至少一次小提交

### 第 3～4 周：HTTP 与 Express

学习：

- HTTP 请求响应
- REST
- Express 路由和中间件
- 错误处理
- 参数校验

产出：

- 内存版待办 API
- Apifox 接口集合

### 第 5～7 周：SQL 与 PostgreSQL

学习：

- 表设计
- CRUD
- JOIN
- 聚合
- 索引
- 事务

产出：

- 记账 API
- ER 图
- 20 条手写 SQL

### 第 8～9 周：工程结构

学习：

- Controller/Service/Repository
- 配置管理
- 错误码
- 日志
- TypeScript 类型设计

产出：

- 把记账 API 从单文件重构成模块化项目

### 第 10～12 周：认证与安全

学习：

- 密码哈希
- JWT/Session
- Refresh Token
- CORS/CSRF/XSS
- SQL 注入
- 限流

产出：

- 完整用户系统
- 权限测试

### 第 13～14 周：缓存与文件

学习：

- Redis
- 缓存策略
- MinIO/S3
- 文件上传

产出：

- 验证码
- 头像上传
- 私有文件下载

### 第 15～16 周：测试

学习：

- Vitest/Jest
- Supertest
- 单元测试
- 集成测试
- 测试数据库

产出：

- 核心 Service 测试
- 登录与笔记 API 集成测试

### 第 17～18 周：Docker 与部署

学习：

- Linux
- Dockerfile
- Compose
- Nginx
- HTTPS
- 备份恢复

产出：

- 从零部署一个测试 API
- 写部署文档和恢复文档

### 第 19～20 周：性能与可靠性

学习：

- SQL 执行计划
- 分页
- 连接池
- 超时重试
- 幂等
- 限流和降级

产出：

- 一份接口性能报告
- 优化前后对比

### 第 21～22 周：异步任务与监控

学习：

- 消息队列
- 定时任务
- 日志
- 指标
- Request ID

产出：

- 邮件队列
- 失败重试
- 服务监控面板

### 第 23～24 周：完整项目

完成 NoteFlow 核心后端：

- 用户
- 笔记
- 分类
- 文件
- 测试
- Docker
- Nginx
- HTTPS
- 备份
- 文档

最后写一份复盘：

- 做了什么？
- 为什么这样设计？
- 遇到什么问题？
- 如何定位和解决？
- 如果再做一次会怎么改？

---

## 28. 后端开发固定思维

### 28.1 数据来自哪里，去向哪里

每写一个功能先画：

```text
客户端 → API → 业务规则 → 数据库/第三方服务 → 响应
```

### 28.2 不信任外部输入

外部输入包括：

- 请求体
- URL 参数
- 请求头
- Cookie
- 上传文件
- 第三方 API 返回
- 消息队列消息

全部需要校验。

### 28.3 先想失败，再想成功

创建笔记时要考虑：

- 未登录。
- 标题为空。
- 标题过长。
- 分类不属于用户。
- 数据库不可用。
- 客户端重复提交。
- 响应在网络中丢失。

### 28.4 权限必须靠服务端

前端隐藏按钮不是权限控制。攻击者可以直接请求接口。

### 28.5 数据一致性优先

性能慢可以优化，数据错了可能无法挽回。涉及多步写操作时先考虑事务和幂等。

### 28.6 可观测之后才可维护

没有日志、指标、Request ID 的服务，出错后只能猜。

### 28.7 配置与代码分离

开发、测试、生产环境不同的内容应该使用配置：

- 数据库地址
- Redis 地址
- 域名
- 日志级别
- 第三方密钥

### 28.8 先模块化，再微服务

清晰边界比服务数量更重要。

### 28.9 设计接口时考虑客户端

后端不只要“能返回数据”，还要考虑：

- 字段稳定性。
- 分页。
- 错误码。
- 版本兼容。
- 移动网络重试。
- 离线同步。

### 28.10 用证据优化

先看日志、指标、执行计划、压测结果，再决定加索引、缓存或扩容。

---

## 29. 常见故障排查清单

### 29.1 API 无法访问

按顺序检查：

1. Node.js 进程是否运行？
2. 是否监听正确端口和地址？
3. 本机 `curl 127.0.0.1:3000/health` 是否成功？
4. Docker 容器是否健康？
5. Nginx 配置是否通过 `nginx -t`？
6. Nginx 是否指向正确上游？
7. 防火墙是否开放 80/443？
8. DNS 是否指向正确 IP？
9. HTTPS 证书是否有效？
10. Cloudflare 是否能连接源站？

### 29.2 数据库连接失败

检查：

- 容器是否运行。
- 数据库名、用户名、密码。
- 主机名在 Docker 内是否应该写服务名。
- 端口是否正确。
- 用户权限。
- 数据目录版本是否兼容。
- 连接池是否耗尽。
- 防火墙或安全组。

Docker Compose 内部通常使用：

```env
DATABASE_URL=postgresql://noteflow:password@postgres:5432/noteflow
```

这里的 `postgres` 是 Compose 服务名，不是 `localhost`。

### 29.3 401 与 403

401：

- 没有 Token。
- Token 过期。
- 签名密钥不一致。
- Authorization 格式错误。

403：

- 用户已登录，但角色不够。
- 资源不属于当前用户。
- 账号被禁用。

### 29.4 CORS 错误

检查：

- 请求是否来自浏览器。
- Origin 是否在允许列表。
- 是否使用 Cookie。
- 服务端是否允许 credentials。
- 预检 OPTIONS 是否成功。
- 前端和后端协议、域名、端口是否一致。

不要通过浏览器插件关闭 CORS 来“修复”生产问题。

### 29.5 内存持续上涨

可能原因：

- 无限制增长的 Map/数组缓存。
- 未清理定时器。
- 未释放监听器。
- 一次加载大文件。
- 数据库结果过大。
- 日志积压。
- WebSocket 连接未清理。

排查：

- 观察堆内存。
- 生成 Heap Snapshot。
- 检查对象引用。
- 设置缓存上限和过期时间。
- 使用流处理大文件。

### 29.6 接口越来越慢

检查：

- P95/P99 延迟。
- 慢 SQL。
- `EXPLAIN ANALYZE`。
- 数据量变化。
- N+1 查询。
- 连接池等待。
- 第三方接口。
- CPU、内存、磁盘。
- 队列积压。

### 29.7 Docker 容器反复重启

```bash
docker compose ps
docker compose logs --tail=200 api
docker inspect <container-name>
```

常见原因：

- 环境变量缺失。
- 端口冲突。
- 启动命令错误。
- 数据库尚未就绪。
- 文件权限。
- 镜像架构不匹配。

### 29.8 排错原则

1. 复现问题。
2. 缩小范围。
3. 找到最靠近事实的日志。
4. 一次只改变一个变量。
5. 修复后再次验证。
6. 记录根因与预防措施。

不要一次修改 Nginx、Docker、DNS、代码和数据库，否则即使恢复也不知道是哪一步有效。

---

## 30. 面试题与自测题

### 基础

1. Node.js 是语言还是运行时？
2. Promise 有哪些状态？
3. `async/await` 是否会阻塞整个 Node.js 进程？
4. GET 和 POST 的常见区别是什么？
5. 401 和 403 有什么区别？
6. PUT 和 PATCH 有什么区别？
7. REST API 是什么？
8. 中间件有什么作用？

### 数据库

1. 主键和外键是什么？
2. INNER JOIN 和 LEFT JOIN 有什么区别？
3. 索引为什么能加速查询？
4. 为什么索引不能无限增加？
5. 事务的 ACID 是什么？
6. 什么是脏读、不可重复读、幻读？
7. 如何避免超卖？
8. Offset 分页和游标分页如何选择？
9. 什么是 N+1 查询？
10. ORM 能否代替 SQL 学习？

### 认证与安全

1. 密码为什么不能用普通 SHA256 直接保存？
2. JWT 的 Payload 是否加密？
3. Access Token 和 Refresh Token 有什么区别？
4. 登录成功是否意味着能删除任意笔记？
5. SQL 注入是怎样产生的？
6. CORS 是服务端权限系统吗？
7. Session 与 JWT 如何选择？
8. 如何让 Refresh Token 失效？
9. 如何防止暴力破解登录接口？
10. 文件上传有哪些风险？

### 工程与部署

1. Controller、Service、Repository 各做什么？
2. 为什么要使用环境变量？
3. Docker 镜像和容器有什么区别？
4. Nginx 反向代理是什么？
5. 为什么数据库端口不应该直接暴露公网？
6. 如何设计健康检查？
7. 备份为什么必须验证恢复？
8. 如何实现服务崩溃后自动重启？

### 进阶

1. Redis 缓存和 PostgreSQL 数据不一致怎么办？
2. 什么是缓存穿透、击穿和雪崩？
3. 为什么消息可能被重复消费？
4. 如何保证任务幂等？
5. 第三方接口超时时怎么办？
6. 为什么重试要使用指数退避？
7. 单体项目何时需要拆微服务？
8. P95 延迟比平均延迟多说明了什么？
9. 如何定位接口突然变慢？
10. 离线同步发生冲突时有哪些策略？

### 项目表达题

请尝试完整回答：

> NoteFlow 的一条笔记从 React Native 页面提交后，经过哪些模块最终保存到 PostgreSQL？其中如何保证用户只能修改自己的笔记？服务部署到服务器后又如何通过 Nginx 和 HTTPS 对外提供接口？

一个合格回答应包含：

- 页面/Hook/Service。
- HTTP API。
- Nginx。
- Express 路由。
- 鉴权中间件。
- 参数校验。
- Controller/Service/Repository。
- 参数化 SQL。
- `noteId + userId` 所有权判断。
- PostgreSQL。
- 状态码和错误响应。

---

## 31. 常用术语表

| 术语 | 简单解释 |
|---|---|
| API | 系统之间约定的调用方式 |
| Endpoint | 一个具体接口地址 |
| Runtime | 程序运行环境 |
| Middleware | 请求流程中的通用处理环节 |
| Controller | 接收请求并组织响应 |
| Service | 执行业务规则 |
| Repository | 封装数据读写 |
| ORM | 对象与数据库表之间的映射工具 |
| Migration | 可追踪的数据库结构变更 |
| Schema | 数据结构或数据库结构定义 |
| Authentication | 身份认证，确认你是谁 |
| Authorization | 权限授权，确认你能做什么 |
| Hash | 单向摘要计算 |
| Salt | 密码哈希中加入的随机数据 |
| Token | 代表身份或许可的凭证 |
| Cache | 为加快读取而保存的副本 |
| Queue | 等待后台处理的任务队列 |
| Idempotency | 同一操作执行多次仍得到等效结果 |
| Transaction | 作为整体成功或失败的一组操作 |
| Index | 加快数据库查找的数据结构 |
| Connection Pool | 可重复使用的数据库连接集合 |
| Reverse Proxy | 代替客户端把请求转发给后端服务 |
| Load Balancer | 把请求分配给多个服务实例 |
| Container | 隔离运行应用的标准化环境 |
| CI | 自动检查、测试和构建代码 |
| CD | 自动交付或部署代码 |
| Observability | 通过日志、指标、链路了解系统状态 |
| Latency | 请求耗时 |
| Throughput | 单位时间处理量 |
| Race Condition | 并发执行顺序导致的不确定错误 |
| Dead Letter Queue | 保存多次处理失败消息的队列 |
| Monolith | 作为一个整体开发部署的应用 |
| Microservice | 独立开发和部署的业务服务 |
| Eventual Consistency | 数据经过一段时间后达到一致 |

---

## 32. 下一步行动清单

### 现在就做

- [ ] 安装 Node.js、PostgreSQL、Git。
- [ ] 创建一个 TypeScript + Express 项目。
- [ ] 写 `/health` 接口。
- [ ] 写内存版 Notes CRUD。
- [ ] 用 Apifox 测试每个接口。
- [ ] 把数据迁移到 PostgreSQL。
- [ ] 为所有 SQL 使用参数化查询。

### 然后完成

- [ ] 拆分 Router、Controller、Service、Repository。
- [ ] 使用 Zod 做参数校验。
- [ ] 增加统一错误处理。
- [ ] 实现注册、登录、JWT。
- [ ] 验证用户只能操作自己的笔记。
- [ ] 为核心业务写测试。

### 再进入生产阶段

- [ ] 编写 Dockerfile。
- [ ] 使用 Docker Compose 管理 API、PostgreSQL、Redis、MinIO。
- [ ] 配置 Nginx 和 HTTPS。
- [ ] 设置日志、健康检查和自动重启。
- [ ] 设置数据库自动备份。
- [ ] 完成一次真实恢复演练。
- [ ] 为登录和验证码接口限流。

### 进入进阶阶段

- [ ] 分析慢 SQL 和执行计划。
- [ ] 学习游标分页。
- [ ] 为真实热点增加 Redis 缓存。
- [ ] 使用队列处理邮件和后台任务。
- [ ] 增加指标监控。
- [ ] 设计 NoteFlow 离线同步与冲突解决。
- [ ] 用 FastAPI 实现一个 AI 摘要服务。
- [ ] 用 Spring Boot 或 Go 重写一个小模块进行对比。

---

## 结语

后端能力不是记住框架 API，而是逐渐形成一种可靠性思维：

```text
输入是否可信？
身份是否真实？
用户是否有权？
数据是否一致？
失败后能否恢复？
并发时是否正确？
上线后能否观察？
出现问题能否回滚？
```

如果你能围绕这些问题设计和解释 NoteFlow，即使以后从 Express 换到 NestJS、Spring Boot、FastAPI 或 Go，核心能力依然可以迁移。

最好的学习路线不是“等全部学完再做项目”，而是：

> 学一个概念 → 写一个例子 → 放进项目 → 故意测试错误 → 总结原因 → 再进入下一层。

从一个可靠的 `/health` 接口开始，最后把整个系统部署上线。后端就是这样一层一层建立起来的。
