---
name: go-web-stack
description: 本项目固定的 Web 技术选型规范（Go 后端 + Vue3 前端 + go:embed 单二进制部署）。每当用户在本项目中要求编写、生成、搭建、脚手架化任何 Go Web 后端代码、Vue3 前端组件、API、项目结构、构建流程、依赖选择时，必须先加载并遵循此技能中列出的库和约定，禁止随意引入清单之外的替代库（如用 React 代替 Vue3、用 Vuex 代替 Pinia、用 Element UI 代替 Element Plus、用 Wails 代替标准 Web 等）。涉及“新建项目”“脚手架”“技术选型”“该用什么库”“项目结构应该怎么组织”“如何构建部署”等问题时也应触发。
---

# Go Web 项目技术选型规范

本技能定义本项目固定的 Web 技术栈、项目结构、前后端通信方式、配置规范和构建部署方式。

核心架构：

> **Vue3 + TypeScript + Vite 前端，Go + Gin 后端，Vue 构建产物通过 go:embed 嵌入 Go，最终只生成一个可执行文件。**

编写任何代码、给出任何架构建议、搭建项目脚手架或选择依赖时，**必须遵循本文件约定**。

除非用户明确要求，不得擅自替换核心技术栈。

当所需能力在本文件清单中找不到对应项时，**必须**执行第四十一节「清单外能力选型流程」，先检索现成成熟仓库并按 star 数、维护活跃度、社区推荐度筛选，禁止未经检索就自造轮子或凭印象指定库名。

---

# 一、核心架构

项目采用：

```text
Vue3 Web
    ↓
Vite Build
    ↓
project/server/web/
    ↓
Go embed
    ↓
Go Binary
```

运行时：

```text
Browser
   │
   │ HTTP / HTTPS
   ▼
┌──────────────────────────────┐
│         Go Server            │
│                              │
│  Gin                         │
│   ├── /api/v1/*              │
│   │       ↓                  │
│   │    Handler               │
│   │       ↓                  │
│   │    Service               │
│   │       ↓                  │
│   │    Repository            │
│   │                          │
│   └── /*                     │
│           ↓                  │
│      Embedded Vue            │
└──────────────────────────────┘
```

生产环境原则：

```text
一个 Go 可执行文件
+
一个 YAML 配置文件
+
外部依赖的数据目录 / 数据库
```

不要求：

```text
Node.js
Nginx
独立前端服务器
Wails
Docker 才能运行
```

---

# 二、项目目录结构

项目固定采用：

```text
project/
├── web/                         # Vue3 前端源码
│   ├── src/
│   │   ├── api/
│   │   ├── assets/
│   │   ├── components/
│   │   ├── layouts/
│   │   ├── router/
│   │   ├── stores/
│   │   ├── types/
│   │   ├── utils/
│   │   ├── views/
│   │   ├── App.vue
│   │   └── main.ts
│   │
│   ├── public/
│   ├── package.json
│   ├── pnpm-lock.yaml
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── index.html
│
├── server/                      # Go 后端
│   ├── web/                     # Vue build 输出目录，由 web 构建生成
│   │   ├── assets/
│   │   └── index.html
│   │
│   ├── cmd/
│   │   └── server/
│   │       └── main.go
│   │
│   ├── internal/
│   │   ├── config/
│   │   ├── handler/
│   │   ├── middleware/
│   │   ├── model/
│   │   ├── repository/
│   │   ├── router/
│   │   ├── service/
│   │   └── web/
│   │
│   ├── pkg/
│   ├── configs/
│   │   └── config.yaml
│   ├── go.mod
│   └── go.sum
│
├── scripts/
│   ├── build.ps1
│   └── build.sh
│
├── .gitignore
├── Makefile
└── README.md
```

其中：

```text
project/web
```

只保存前端源码。

构建后的文件：

```text
project/server/web
```

由前端自动生成。

**不要把前端 build 产物放在 `project/web/dist` 作为最终 Go 构建输入。**

---

# 三、前端构建输出规范

Vue 使用 Vite 构建。

默认：

```bash
cd web
pnpm build
```

必须将输出目录设置为：

```text
../server/web
```

Vite：

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  build: {
    outDir: '../server/web',
    emptyOutDir: true,
  },
})
```

因此：

```text
project/
├── web/
│   └── src/
│
└── server/
    └── web/
        ├── index.html
        └── assets/
```

执行：

```text
pnpm build
```

以后：

```text
web/src
   ↓
Vite
   ↓
server/web
```

---

# 四、Go Embed

Go 后端必须使用标准库：

```go
embed
```

嵌入前端构建产物。

推荐：

```text
server/internal/web/embed.go
```

示例：

```go
package web

import (
	"embed"
	"io/fs"
)

//go:embed all:web
var Files embed.FS

func FS() fs.FS {
	fsys, err := fs.Sub(Files, "web")
	if err != nil {
		panic(err)
	}

	return fsys
}
```

如果 Go 版本或项目环境不适合 `all:` 写法，可以使用：

```go
//go:embed web/*
var Files embed.FS
```

但必须保证最终嵌入的是：

```text
server/web/
```

---

# 五、最终构建产物

最终：

```bash
cd server
go build -o ../bin/app ./cmd/server
```

生成：

```text
project/
└── bin/
    └── app
```

Windows：

```text
app.exe
```

Linux：

```text
app
```

最终运行：

```bash
./app
```

或者：

```powershell
.\app.exe
```

访问：

```text
http://127.0.0.1:8080
```

不需要启动：

```text
npm
vite
nginx
wails
```

---

# 六、前后端通信

Web 前端与 Go 后端统一使用：

```text
HTTP / HTTPS
REST API
```

固定：

```text
Vue
 ↓
Axios
 ↓
Gin
 ↓
Service
 ↓
Repository
```

API 统一：

```text
/api/v1/
```

例如：

```text
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
DELETE /api/v1/users/:id
```

禁止：

```text
Wails Binding
Wails Runtime
桌面 IPC
```

---

# 七、后端核心技术栈

| 类别     | 固定选型                        |
| ------ | --------------------------- |
| 编程语言   | **Go**                      |
| Web 框架 | **gin-gonic/gin**           |
| 配置管理   | **spf13/viper**             |
| CLI    | **spf13/cobra**             |
| 日志     | **rs/zerolog**              |
| ORM    | **go-gorm/gorm**            |
| Redis  | **redis/go-redis**          |
| JSON   | **bytedance/sonic**         |
| 参数校验   | **go-playground/validator** |

---

# 八、Web 框架

固定使用：

```text
github.com/gin-gonic/gin
```

Gin 负责：

* HTTP Server
* REST API
* 路由
* Middleware
* 参数绑定
* HTTP Response
* 错误处理
* 静态文件服务

基本调用关系：

```text
Router
   ↓
Middleware
   ↓
Handler
   ↓
Service
   ↓
Repository
   ↓
Database
```

禁止在 Handler 中堆积复杂业务逻辑。

---

# 九、Handler

Handler 只负责：

```text
HTTP Request
    ↓
参数解析
    ↓
参数校验
    ↓
调用 Service
    ↓
HTTP Response
```

禁止：

```text
Handler
 ├── SQL
 ├── Redis
 ├── 复杂业务判断
 ├── 文件业务
 └── 第三方 API 业务
```

复杂业务必须进入 Service。

---

# 十、Service

Service 负责核心业务逻辑。

例如：

```text
internal/
├── handler/
│   └── user.go
│
├── service/
│   └── user.go
│
└── repository/
    └── user.go
```

调用关系：

```text
HTTP
 ↓
Handler
 ↓
Service
 ↓
Repository
 ↓
Database
```

Service 不应该依赖 Gin Context 的具体 HTTP 行为。

---

# 十一、Repository

Repository 负责数据访问。

禁止 Handler 直接：

```go
db.Where(...)
db.Create(...)
db.Delete(...)
```

统一：

```text
Service
   ↓
Repository
   ↓
GORM
```

这样方便：

* 单元测试
* Mock
* 数据库替换
* 业务解耦

---

# 十二、统一 YAML 配置

**项目配置统一使用 YAML。**

这是本项目的固定规范。

默认：

```text
server/configs/config.yaml
```

示例：

```yaml
server:
  host: 0.0.0.0
  port: 8080
  mode: release

database:
  driver: mysql
  host: 127.0.0.1
  port: 3306
  database: app
  username: root
  password: ""
  max_open_conns: 100
  max_idle_conns: 10

redis:
  enabled: false
  host: 127.0.0.1
  port: 6379
  password: ""
  db: 0

log:
  level: info
  format: json

jwt:
  secret: ""
  expire: 24h
```

配置文件：

```text
config.yaml
```

作为唯一正式配置格式。

禁止默认同时维护：

```text
config.json
config.toml
config.xml
```

---

# 十三、Viper 配置管理

固定：

```text
github.com/spf13/viper
```

Viper 负责读取：

```text
YAML
```

以及必要的环境变量覆盖。

配置结构使用 Go Struct：

```go
type Config struct {
	Server   ServerConfig   `mapstructure:"server"`
	Database DatabaseConfig `mapstructure:"database"`
	Redis    RedisConfig    `mapstructure:"redis"`
	Log      LogConfig      `mapstructure:"log"`
}
```

推荐：

```text
internal/config/
├── config.go
└── types.go
```

配置加载流程：

```text
config.yaml
     ↓
   Viper
     ↓
 Config Struct
     ↓
Application
```

---

# 十四、配置文件位置

默认读取：

```text
./configs/config.yaml
```

也可以通过 CLI 指定：

```bash
./app --config ./configs/config.yaml
```

生产环境：

```bash
./app --config /etc/myapp/config.yaml
```

**配置文件不嵌入 Go 二进制。**

原因：

> 前端属于程序本身，可以 embed；运行配置属于部署环境，应保持外置。

因此最终：

```text
app
config.yaml
```

是合理的生产部署方式。

---

# 十五、环境变量

环境变量只作为 YAML 配置的覆盖机制。

例如：

```text
APP_SERVER_PORT=9000
```

可以覆盖：

```yaml
server:
  port: 8080
```

但是：

> **正式配置规范仍然以 YAML 为主。**

不要把整个项目改造成 `.env` 驱动。

禁止默认使用：

```text
.env
.env.production
.env.local
```

作为主要配置体系。

---

# 十六、日志

固定：

```text
github.com/rs/zerolog
```

禁止：

```go
log.Println()
```

禁止默认引入：

```text
logrus
zap
```

日志至少包含：

```text
time
level
message
request_id
```

推荐 JSON：

```json
{
  "level": "info",
  "time": "2026-10-05T20:00:00+08:00",
  "message": "server started",
  "request_id": "xxx"
}
```

---

# 十七、数据库

默认：

```text
github.com/go-gorm/gorm
```

数据库：

```text
MySQL
PostgreSQL
```

根据项目实际需求选择。

架构：

```text
Handler
   ↓
Service
   ↓
Repository
   ↓
GORM
   ↓
Database
```

数据库 Model：

```text
internal/model/
```

Repository：

```text
internal/repository/
```

禁止把数据库 Model 无条件直接作为 API Response。

区分：

```text
Request DTO
Response DTO
Database Model
```

---

# 十八、Redis

固定：

```text
github.com/redis/go-redis
```

Redis 仅在真正需要时启用。

用途：

* Cache
* Session
* Distributed Lock
* Rate Limit
* Temporary Data
* Task State

不要因为技术栈中有 Redis，就默认部署 Redis。

---

# 十九、JSON

高性能场景：

```text
github.com/bytedance/sonic
```

特殊兼容场景可以使用：

```text
encoding/json
```

不要为了追求“统一”而强制所有代码使用 Sonic。

---

# 二十、参数校验

固定：

```text
github.com/go-playground/validator
```

例如：

```go
type CreateUserRequest struct {
	Username string `json:"username" validate:"required,min=3,max=32"`
	Email    string `json:"email" validate:"required,email"`
}
```

参数校验应该发生在：

```text
Handler / Request DTO
```

---

# 二十一、API Response

统一：

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

错误：

```json
{
  "code": 40001,
  "message": "参数错误",
  "data": null
}
```

HTTP Status Code 与业务 Code 同时使用。

---

# 二十二、认证与权限

需要 JWT 时使用：

```text
github.com/golang-jwt/jwt/v5
```

认证：

```text
Browser
   ↓
Login
   ↓
JWT
   ↓
Authorization: Bearer xxx
   ↓
Gin Middleware
   ↓
Handler
```

权限复杂时：

```text
User
 ↓
Role
 ↓
Permission
```

不要在 Handler 中硬编码大量权限逻辑。

---

# 二十三、前端技术栈

固定：

| 类别   | 选型               |
| ---- | ---------------- |
| 前端框架 | **Vue 3**        |
| 语言   | **TypeScript**   |
| 构建工具 | **Vite**         |
| UI   | **Element Plus** |
| 状态管理 | **Pinia**        |
| CSS  | **TailwindCSS**  |
| HTTP | **Axios**        |
| 路由   | **Vue Router**   |

禁止：

```text
React
Vue 2
Vuex
Element UI
Ant Design Vue
```

除非用户明确要求更换。

---

# 二十四、Vue3 编码规范

所有新组件：

```vue
<script setup lang="ts">
</script>

<template>
</template>

<style scoped>
</style>
```

使用：

```text
Composition API
<script setup>
TypeScript
```

禁止新代码使用 Options API。

---

# 二十五、Element Plus

固定：

```text
element-plus
```

负责：

```text
Button
Form
Table
Dialog
Drawer
Select
DatePicker
Pagination
Message
Notification
```

禁止为了简单组件再引入其他 UI 框架。

---

# 二十六、Pinia

固定：

```text
pinia
```

目录：

```text
web/src/stores/
├── user.ts
├── app.ts
└── settings.ts
```

适合：

```text
用户信息
Token
权限
全局配置
主题
应用状态
```

禁止 Vuex。

组件局部状态不要强行放进 Pinia。

---

# 二十七、TailwindCSS

固定：

```text
TailwindCSS
```

主要用于：

```text
布局
间距
尺寸
响应式
Flex
Grid
字体
```

Element Plus 负责复杂业务组件。

例如：

```vue
<div class="flex items-center justify-between gap-4 p-4">
```

复杂组件内部状态难以用 Tailwind 表达时，可以使用：

```vue
<style scoped>
</style>
```

---

# 二十八、Axios

前端 HTTP 请求统一使用：

```text
Axios
```

目录：

```text
web/src/api/
├── request.ts
├── auth.ts
├── user.ts
└── system.ts
```

推荐：

```text
View
 ↓
API
 ↓
Axios
 ↓
Gin API
```

禁止在 Vue Component 中到处直接调用 Axios。

---

# 二十九、Vue Router

固定：

```text
vue-router
```

目录：

```text
web/src/router/
└── index.ts
```

需要权限时使用 Router Guard。

---

# 三十、Vite 开发环境

开发环境：

```text
Browser
   ↓
Vite :5173
   ↓
Proxy /api
   ↓
Gin :8080
```

推荐 Vite 配置：

```ts
server: {
  proxy: {
    '/api': {
      target: 'http://127.0.0.1:8080',
      changeOrigin: true,
    },
  },
}
```

开发时：

```bash
cd web
pnpm dev
```

Go：

```bash
cd server
go run ./cmd/server
```

生产环境则不启动 Vite。

---

# 三十一、生产环境静态文件

生产环境由 Go 直接提供 Vue 静态资源。

流程：

```text
Browser
   ↓
Go Server
   ├── /api/*
   │      ↓
   │    REST API
   │
   └── /*
          ↓
       Embedded Vue
```

Vue SPA 必须支持：

```text
/history
/dashboard
/users
/settings
```

等前端路由刷新。

对于非 `/api/` 路由：

```text
返回 index.html
```

---

# 三十二、go:embed 静态资源规范

前端构建目录：

```text
server/web/
```

Go：

```go
//go:embed all:web
var webFS embed.FS
```

Go 服务启动时：

```text
webFS
 ↓
fs.Sub
 ↓
http.FS
 ↓
Gin
```

静态文件不得依赖外部 Node Server。

---

# 三十三、构建规范

完整生产构建：

```text
1. 安装前端依赖
2. 构建 Vue
3. 输出到 server/web
4. Go 编译
5. 生成单一二进制
```

流程：

```text
web/src
   ↓
pnpm build
   ↓
server/web
   ↓
go build
   ↓
bin/app
```

推荐脚本：

```text
scripts/
├── build.ps1
└── build.sh
```

---

# 三十四、Makefile

推荐：

```makefile
.PHONY: web-build build run clean

web-build:
	cd web && pnpm build

build: web-build
	cd server && go build -o ../bin/app ./cmd/server

run:
	cd server && go run ./cmd/server

clean:
	rm -rf server/web
	rm -rf bin
```

Windows 环境同时提供：

```text
scripts/build.ps1
```

---

# 三十五、Docker

Docker 不是运行该项目的必要条件。

项目本身必须能够：

```bash
./app
```

直接运行。

如果用户要求容器化，再提供：

```text
Dockerfile
```

采用多阶段构建：

```text
Node
 ↓
Vue Build
 ↓
Go Build
 ↓
最终镜像
```

最终镜像只包含：

```text
Go Binary
Config
必要运行目录
```

---

# 三十六、可观测性

链路追踪：

```text
go.opentelemetry.io/otel
```

Metrics：

```text
github.com/prometheus/client_golang
```

按需监控：

```text
HTTP 请求数量
HTTP 请求延迟
HTTP 错误率
Database
Redis
Go GC
Goroutine
CPU
Memory
```

不要默认安装完整可观测性生态。

---

# 三十七、定时任务

固定：

```text
github.com/robfig/cron/v3
```

用于：

```text
定时同步
数据清理
缓存刷新
统计
周期检查
```

复杂耗时任务不能直接阻塞 HTTP Request。

---

# 三十八、通用工具

固定：

```text
github.com/samber/lo
```

仅在确实能够提高代码可读性时使用。

不要为了“统一使用”而强行引入。

---

# 三十九、稳定性治理

| 能力     | 库                    |
| ------ | -------------------- |
| 熔断     | `sony/gobreaker`     |
| 令牌桶限流  | `juju/ratelimit`     |
| 滑动窗口限流 | `go-zero/core/limit` |

按需使用。

---

# 四十、补充能力清单

仅当需求命中场景时引入。

| 场景           | 库                                                       |
| ------------ | ------------------------------------------------------- |
| 更完善的错误堆栈     | `cockroachdb/errors`                                    |
| 编译期依赖注入      | `google/wire`                                           |
| 单元测试断言       | `stretchr/testify`                                      |
| Mock         | `uber-go/mock`                                          |
| 集成测试         | `testcontainers-go`                                     |
| API 文档       | `swaggo/swag`                                           |
| JWT          | `golang-jwt/jwt/v5`                                     |
| gRPC         | `google.golang.org/grpc` + `google.golang.org/protobuf` |
| Kafka        | `segmentio/kafka-go`                                    |
| RabbitMQ     | `rabbitmq/amqp091-go`                                   |
| 分布式 ID       | `bwmarrin/snowflake`                                    |
| Redis 分布式锁   | `go-redsync/redsync`                                    |
| 第三方 HTTP API | `go-resty/resty`                                        |
| CI 代码检查      | `golangci-lint`                                         |
| 优雅关闭         | 原生 `context` + `signal.NotifyContext`                   |
| 容器部署         | Dockerfile 多阶段构建                                        |

---

# 四十一、清单外能力选型流程（强制）

触发：所需能力在本文件清单（第七节后端核心技术栈、第二十三节前端技术栈、第四十节补充能力清单等）中均无对应项，例如采集服务端 CPU / 内存 / 磁盘占用、解析 Office / PDF 文档、图片处理。

1. **先查内置与已有清单**：Go 标准库（`net/http`、`embed`、`encoding/json` 等）或清单内已有库（Gin / Viper / GORM / Axios 等）能覆盖 → 不引入第三方库。
2. **必须搜索**：先用 GitHub 搜索 + 搜索引擎检索该领域的成熟开源实现，禁止跳过此步直接写代码或凭印象指定库名。检索词模板：`<能力> go library github`、`<能力> awesome-go`、`<能力> golang 推荐`；前端能力追加 `<能力> vue3 library github`。
3. **量化筛选**：至少比较 2 个候选，按下表取 star 多且被反复推荐者。

   | 判据 | 门槛 |
   |---|---|
   | Star 数 | 优先 ≥ 1k；同类候选中取最高 |
   | 维护活跃度 | 近 12 个月有提交或发版 |
   | 社区推荐度 | 被 awesome-go、官方文档或 ≥2 个独立技术来源推荐 |
   | 兼容性 | Go 版本匹配、License 可商用、不与现有选型重复 |
   | 部署约束 | 不得破坏「单 Go 二进制 + 外部 YAML」模型，不得引入运行时依赖 Node / Nginx / 独立前端服务 |

   示例：服务端硬件 / 系统信息采集 → `shirou/gopsutil`（高 star、持续维护、社区广泛推荐）。
4. **CHECKPOINT**：落地前输出候选对比（仓库全名 / star 数 / 最近提交 / 推荐来源 / 选择理由），并标注「清单外新增项」。
5. **STOP**：最高 star < 500、近 2 年无维护、或仅单一来源提及 → 列出候选等待用户确认，不得擅自引入。

失败分支（三段式 fallback）：
- 搜索无结果 → 用标准库或清单内已有能力实现最小闭环，并告知用户该功能无成熟第三方库。
- 候选均不达标 → 列出 Top 3 及量化数据，STOP 等用户决策。
- 与清单已有库功能重叠，或候选需要外部运行时 → 回退清单库 / 放弃该候选，不新增依赖。

输出规格：引入时必须写出 `仓库全名 | star 数 | 选择理由 | 清单外新增项` 四要素并告知用户，禁止静默引入。

---

# 四十二、优雅关闭

固定使用 Go 原生：

```text
context
signal.NotifyContext
```

处理：

```text
SIGTERM
SIGINT
```

关闭顺序：

```text
停止接受新请求
        ↓
等待正在处理的请求
        ↓
停止定时任务
        ↓
关闭 Redis / DB
        ↓
退出进程
```

---

# 四十三、代码质量

Go：

```text
gofmt
go vet
staticcheck
errcheck
unused
golangci-lint
```

前端：

```text
TypeScript
ESLint
Prettier
```

---

# 四十四、测试

后端默认：

```text
Go testing
```

需要时：

```text
testify
testcontainers-go
uber-go/mock
```

前端测试仅在用户明确提出需求时引入对应测试框架。

不要默认安装完整测试生态。

---

# 四十五、禁止事项速查

以下行为视为违反本规范：

* ❌ 使用 React 代替 Vue3
* ❌ 使用 Vue2
* ❌ 使用 Vuex
* ❌ 使用 Element UI
* ❌ 使用 Ant Design Vue
* ❌ 使用 Wails
* ❌ 使用 Wails v2/v3
* ❌ 使用 Wails Binding
* ❌ 使用桌面 IPC
* ❌ 将前端 build 输出到 `web/dist` 后再作为最终构建目录
* ❌ Go 构建时依赖独立 Node 服务
* ❌ 生产环境要求 Nginx 才能运行
* ❌ 生产环境要求单独启动 Vue
* ❌ Handler 中直接编写复杂业务逻辑
* ❌ Handler 中直接操作数据库
* ❌ Service 中处理 HTTP Response
* ❌ Vue Component 中大量直接调用 Axios
* ❌ 未经用户要求默认引入补充能力清单中的库
* ❌ 为简单功能随意增加第三方依赖
* ❌ 将数据库 Model 直接作为所有 API Response
* ❌ 默认使用 `.env` 作为主要配置体系
* ❌ 同时维护 YAML、JSON、TOML 多套正式配置
* ❌ 将 `config.yaml` 编译进 Go 二进制
* ❌ 为简单 CRUD 过度设计微服务
* ❌ 为简单任务默认引入消息队列
* ❌ 为简单缓存默认引入 Redis
* 禁止：清单未覆盖能力时未经 GitHub / 搜索引擎检索就自造轮子或凭印象指定库 → 改走第四十一节流程
* 禁止：引入 star 低、长期无维护、无社区推荐的冷门仓库 → 改选第四十一节量化筛选达标的仓库
* 禁止：静默引入清单外新库（未标注「清单外新增项」及选择理由）→ 补齐四要素并告知用户

---

# 四十六、默认技术栈总览

```text
┌─────────────────────────────────────────┐
│              project/web                │
│                                         │
│ Vue 3                                   │
│ TypeScript                              │
│ Vite                                    │
│ Element Plus                            │
│ Pinia                                   │
│ TailwindCSS                             │
│ Axios                                   │
│ Vue Router                              │
└──────────────────┬──────────────────────┘
                   │
                pnpm build
                   │
                   ▼
┌─────────────────────────────────────────┐
│             project/server/web          │
│                                         │
│ index.html                              │
│ assets/                                 │
└──────────────────┬──────────────────────┘
                   │
                go:embed
                   │
                   ▼
┌─────────────────────────────────────────┐
│              project/server             │
│                                         │
│ Go                                      │
│ Gin                                     │
│ Viper                                   │
│ Zerolog                                 │
│ GORM                                    │
│ Validator                               │
│ Sonic                                   │
│ go-redis                                │
└──────────────────┬──────────────────────┘
                   │
                go build
                   │
                   ▼
              ┌───────────┐
              │   app     │
              │  app.exe  │
              └───────────┘
```

配置：

```text
server/configs/config.yaml
```

运行：

```text
app
+
config.yaml
```

核心原则：

> **project/web 只负责前端源码。**

> **project/server/web 只保存前端构建产物。**

> **Vue build 必须直接输出到 project/server/web。**

> **Go 使用 go:embed 将 server/web 嵌入最终二进制。**

> **最终生产部署只需要一个 Go 可执行文件和外部 YAML 配置。**

> **配置统一使用 YAML；环境变量只能作为必要的覆盖机制，不作为主要配置格式。**

> **Vue 负责 UI，Gin 负责 HTTP API，Service 负责业务，Repository 负责数据。**

> **不使用 Wails，不引入桌面运行时。**

> **默认保持简单，只有真实需求出现时才增加 Redis、MQ、gRPC、JWT、OpenTelemetry 等能力。**

