---
name: go-wails-stack
description: 本项目固定的技术选型规范（Go 后端 + Wails v2 桌面应用 + Vue3 前端）。每当用户在本项目中要求编写、生成、搭建、脚手架化任何 Go 代码、Wails 应用、前端 Vue 组件、项目结构、依赖选择时，必须先加载并遵循此技能中列出的库和约定，禁止随意引入清单之外的替代库（如用 Element UI 代替 Element Plus、用 Vuex 代替 Pinia、用 Wails v3 代替 v2 等）。涉及"新建项目""脚手架""技术选型""该用什么库""项目结构应该怎么组织"等问题时也应触发。
---

# Go + Wails 桌面项目技术选型规范

本技能定义了本项目的固定技术栈。编写任何代码、给出任何架构建议、或搭建项目脚手架时，**必须**从下面的清单中选择对应库，不得擅自替换为其他同类库，除非用户明确要求更换。

## 使用方法

1. 收到编码/脚手架/选型相关请求时，先读完本文件全文。
2. 按「核心选型表」直接使用对应库名和用法，不要在多个同类库之间犹豫或询问用户"要不要用 X"。
3. 「补充能力清单」部分为可选能力，仅在用户需求涉及对应场景（如需要 gRPC、需要消息队列）时才引入，不要默认全部安装。
4. 如果用户要求的能力在清单中找不到对应项，先在「补充能力清单」中查找；仍找不到时，**必须**执行第十节「清单外能力选型流程」，禁止未经检索就自造轮子或凭印象指定库名。

---

## 一、项目结构

- 目录布局遵循 [golang-standards/project-layout](https://github.com/golang-standards/project-layout)（`cmd/`、`internal/`、`pkg/` 等标准划分）

---

## 二、桌面应用框架 —— 固定用 Wails v2

| 类别 | 选型 |
|---|---|
| 桌面框架 | **wailsapp/wails v2**（不是 v3——v3 仍处于 Beta，API 未锁定，本项目不采用） |

Wails v2 关键约定：
- 修改 Go 侧结构体/方法后运行 `wails generate module`（或用 `wails dev` 自动监听）重新生成前端 TS 绑定
- 托盘、菜单、文件对话框、通知等系统能力直接用 Wails 内置的 `runtime` 包，**不要**额外引入第三方库实现
- 多平台打包统一用 `wails build -platform`，构建脚本维护在项目根目录

---

## 三、后端核心框架

| 类别 | 库 | 说明 |
|---|---|---|
| Web 框架 | `gin-gonic/gin` | 仅在需要暴露 HTTP 接口时使用 |
| 配置管理 | `spf13/viper` | 多格式配置、环境变量、热更新 |
| 命令行 | `spf13/cobra` | CLI 命令组织 |
| 日志 | `rs/zerolog` | 结构化日志，不用标准库 log 或 logrus |
| ORM | `go-gorm/gorm` 或 `uptrace/bun` | 二选一；bun 更贴近原生 SQL，gorm 更主流易上手，无特殊需求默认选 gorm |
| SQLite 驱动 | [`ncruces/go-sqlite3`](https://github.com/ncruces/go-sqlite3) | 仅在使用 SQLite 时采用；基于 wasm2go、无 CGO，禁止使用依赖 CGO 的 SQLite 驱动 |
| Redis | `redis/go-redis` | |
| JSON 序列化 | `bytedance/sonic` | 高性能场景替代标准库 encoding/json |
| 参数校验 | `go-playground/validator` | 配合 struct tag |

---

## 四、可观测性

- 链路追踪 / Metrics：`open-telemetry/opentelemetry-go`
- Prometheus 指标：`prometheus/client_golang`

## 五、定时任务

- `robfig/cron/v3`

## 六、通用工具库

- `samber/lo`

## 七、稳定性治理

| 能力 | 库 |
|---|---|
| 熔断 | `sony/gobreaker` |
| 限流（令牌桶） | `juju/ratelimit` |
| 限流（滑动窗口） | `go-zero/core/limit` |

---

## 八、前端技术栈（固定组合）

| 类别 | 选型 | 说明 |
|---|---|---|
| 前端框架 | **Vue 3** | 使用 `<script setup>` 组合式 API 风格 |
| UI 组件库 | **Element Plus** | 不用 Element UI（Vue2 版）、不用 Ant Design Vue |
| 状态管理 | **Pinia** | 不用 Vuex |
| 样式方案 | **TailwindCSS** | 与 Element Plus 组合使用，Tailwind 负责布局/间距/细节样式 |
| 构建工具 | Vite | Wails v2 官方 Vue 模板默认集成 |
| 类型系统 | TypeScript | 默认启用，配合 Wails 自动生成的 Go→TS 绑定 |

编码时的具体约定：
- 新组件一律用 `<script setup lang="ts">`
- 全局状态放 Pinia store，禁止用组件间 props/emit 长链路传递跨层级状态
- 样式优先用 Tailwind 原子类；仅当 Tailwind 难以表达（复杂组件内部状态样式）时才写 scoped CSS
- 调用 Go 后端方法一律通过 Wails 自动生成的绑定（`wailsjs/go/...`），不要手写 fetch 调用本地服务

---

## 九、补充能力清单（按需引入，不要默认全装）

仅当用户的需求命中以下场景时才建议引入对应库：

| 场景 | 库 |
|---|---|
| 需要更完善的错误堆栈 | `cockroachdb/errors` |
| 需要编译期依赖注入 | `google/wire` |
| 单元测试断言 | `stretchr/testify` |
| Mock 生成 | `uber-go/mock` |
| 集成测试（需要真实 DB/中间件） | `testcontainers-go` |
| 需要暴露 HTTP 接口文档 | `swaggo/swag`（配合 gin） |
| 需要 JWT 鉴权 | `golang-jwt/jwt/v5` |
| 需要 gRPC 通信 | `google.golang.org/grpc` + `google.golang.org/protobuf` |
| 需要消息队列（Kafka） | `segmentio/kafka-go` |
| 需要消息队列（RabbitMQ） | `rabbitmq/amqp091-go` |
| 需要分布式 ID | `bwmarrin/snowflake` |
| 需要分布式锁（基于 Redis） | `go-redsync/redsync` |
| 需要调用第三方 HTTP 接口 | `go-resty/resty` |
| CI 代码质量检查 | `golangci-lint` |
| 优雅关闭 | 原生 `context` + `signal.NotifyContext` |
| 需要独立部署为容器服务 | Dockerfile（多阶段构建） |

---

## 十、清单外能力选型流程（强制）

触发：所需能力在第二～九节清单（含第九节补充能力清单）中均无对应项，例如采集 CPU / 内存 / 磁盘占用、解析 Office 文档、操作系统级能力。

1. **先查内置**：Wails `runtime` 包或 Go 标准库已能覆盖 → 不引入第三方库。
2. **必须搜索**：先用 GitHub 搜索 + 搜索引擎检索该领域的成熟开源实现，禁止跳过此步直接写代码或凭印象指定库名。检索词模板：`<能力> go library github`、`<能力> awesome-go`、`<能力> golang 推荐`。
3. **量化筛选**：至少比较 2 个候选，按下表取 star 多且被反复推荐者。

   | 判据 | 门槛 |
   |---|---|
   | Star 数 | 优先 ≥ 1k；同类候选中取最高 |
   | 维护活跃度 | 近 12 个月有提交或发版 |
   | 社区推荐度 | 被 awesome-go、官方文档或 ≥2 个独立技术来源推荐 |
   | 兼容性 | Go 版本匹配、License 可商用、不与现有选型重复 |

   示例：硬件 / 系统信息采集 → `shirou/gopsutil`（高 star、持续维护、社区广泛推荐）。
4. **CHECKPOINT**：落地前输出候选对比（仓库全名 / star 数 / 最近提交 / 推荐来源 / 选择理由），并标注「清单外新增项」。
5. **STOP**：最高 star < 500、近 2 年无维护、或仅单一来源提及 → 列出候选等待用户确认，不得擅自引入。

失败分支（三段式 fallback）：
- 搜索无结果 → 用标准库或 Wails 内置能力实现最小闭环，并告知用户该功能无成熟第三方库。
- 候选均不达标 → 列出 Top 3 及量化数据，STOP 等用户决策。
- 与清单已有库功能重叠 → 回退使用清单库，不新增依赖。

输出规格：引入时必须写出 `仓库全名 | star 数 | 选择理由 | 清单外新增项` 四要素并告知用户，禁止静默引入。

---

## 十一、禁止事项速查

编码或建议时如果出现以下情况，视为违反本规范，需要改正：

- ❌ 使用 Wails v3 而不是 v2
- ❌ 使用 SQLite 时采用依赖 CGO 的驱动；统一使用 `ncruces/go-sqlite3`
- ❌ 用 Element UI 而不是 Element Plus
- ❌ 用 Vuex 而不是 Pinia
- ❌ 用 React 而不是 Vue3
- ❌ 未经用户要求就默认引入「补充能力清单」中的库
- ❌ 前端直接 fetch 调用后端，而不是走 Wails 生成的绑定
- 禁止：清单未覆盖能力时未经 GitHub / 搜索引擎检索就自造轮子或凭印象指定库 → 改走第十节流程
- 禁止：引入 star 低、长期无维护、无社区推荐的冷门仓库 → 改选第十节量化筛选达标的仓库
- 禁止：静默引入清单外新库（未标注「清单外新增项」及选择理由）→ 补齐四要素并告知用户
