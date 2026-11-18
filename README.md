# Go Stack Skill

一套面向 Go 项目的固定技术选型技能包。安装后，AI 编码 agent 会按统一规范生成代码，而不是每次临时挑库。

- **go-web-stack** — Go（Gin）+ Vue3 + Vite，通过 `go:embed` 打成单个可执行文件
- **go-wails-stack** — Go + Wails v2 桌面应用 + Vue3

**English:** Opinionated Go stack conventions packaged as agent skills. Install once and your coding agent follows one fixed stack — Gin + Vue3 + `go:embed` for web, Wails v2 + Vue3 for desktop — instead of improvising libraries.

## Quick Start (English)

Requires Node.js >= 22.20.

```bash
npx skills add yezihack/go-stack-skill
```

The installer detects installed agents and copies the skills into their skills directories. Install a single skill:

```bash
npx skills add yezihack/go-stack-skill --skill go-web-stack
```

Or paste this into any agent that can read the web and run commands:

> 帮我安装这个 skill：https://github.com/yezihack/go-stack-skill

**Agent instruction:** read this README's Install section first, then run the installer. If the installer is unavailable or your agent is not supported, copy `skills/<name>/` into your own skills directory. Do not clone the repository into the user's current project.

## 一行安装

对任意能读网页、能执行命令的 agent 说：

> 帮我安装这个 skill：https://github.com/yezihack/go-stack-skill

agent 读完本 README 后，应执行下面的安装命令；如果安装器不可用，则按「手工安装」把技能目录拷进自己的 skills 目录。

## 安装

### 整包装（默认）

```bash
npx skills add yezihack/go-stack-skill
```

### 只装一个技能

```bash
npx skills add yezihack/go-stack-skill --skill go-web-stack
npx skills add yezihack/go-stack-skill --skill go-wails-stack
```

### 常用参数

| 参数 | 作用 |
|---|---|
| `--all -y` | 非交互安装全部技能 |
| `--agent <name>` | 只装到指定 agent，如 `--agent claude-code`、`--agent codebuddy` |
| `-g` | 装到用户级目录，跨项目可用 |

### 支持平台

通用安装器支持 79 个 agent，你点名的这些都覆盖：Claude Code、Codex、Cursor、Gemini CLI、OpenCode、CodeBuddy、OpenClaw、Hermes Agent。

- 自动检测：`npx skills add yezihack/go-stack-skill`
- 指定平台：`npx skills add yezihack/go-stack-skill --agent <name>`
- 完整清单：安装器的 [Supported agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)

### 手工安装（WorkBuddy 等未覆盖平台）

**WorkBuddy** 不在安装器清单内，但支持标准 `SKILL.md`：

```text
用户级（跨项目）：
  Windows  C:\Users\<你>\.workbuddy\skills\
  macOS    ~/.workbuddy/skills/
  Linux    ~/.workbuddy/skills/

项目级（随仓库提交）：
  <项目根目录>/.workbuddy/skills/
```

把本仓库的 `skills/go-web-stack/` 和 `skills/go-wails-stack/` 整个目录复制进去，重启 WorkBuddy 即可。

**其他未覆盖的 agent**：把 `skills/<name>/` 整个目录复制到该 agent 的 skills 目录。多数 agent 使用 `.agents/skills/`（Codex、Cursor、Gemini CLI 等），Claude Code 使用 `.claude/skills/`，CodeBuddy 使用 `.codebuddy/skills/`。

## 更新

```bash
npx skills update
```

## 包含的技能

### go-web-stack

固定 Go Web 技术栈：Gin 后端、Vue3 + TypeScript + Vite 前端、`go:embed` 单二进制部署，并规定目录结构、分层、配置、日志、ORM、参数校验等选型。遇到清单外能力时，要求先检索成熟仓库再决定，禁止自造轮子。

### go-wails-stack

固定 Go 桌面技术栈：Wails v2（不是 v3）、Vue3 + Element Plus + Pinia + TailwindCSS，规定 Go 侧库选型、Wails 绑定用法、托盘/菜单/对话框等系统能力的实现方式，以及跨平台打包约定。

## 仓库结构

```text
skills/
├── go-web-stack/
│   └── SKILL.md
└── go-wails-stack/
    └── SKILL.md
```

每个技能只需要一个 `SKILL.md`；name 与目录名一致，frontmatter 只使用 `name` 和 `description`，以保持跨 agent 可移植。

## 反馈

问题与建议请提到 [GitHub Issues](https://github.com/yezihack/go-stack-skill/issues)。

## License

[MIT](./LICENSE)
