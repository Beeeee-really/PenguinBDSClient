# HuHoBotPenguin-Llama — LeviLamina (LLSE Node.js 后端)

QQ 开放平台官方机器人与 Minecraft 基岩版服务器之间的聊天 / 命令桥接插件，**内置 LLM AI 助理**（OpenAI 兼容接口，无需额外部署 AstrBot）。由 [HuHoBot/PenguinClient](https://github.com/HuHoBot/PenguinClient)（Java 版）移植。

> 本版名为 **Llama**（代指大模型），是在普通版 `HuHoBotPenguin-LLSE` 基础上额外集成 AI 对话能力。不需要 AI 请用普通版。

## AI 能力（相比普通版新增）

- **群内 AI 对话**：未命中命令的普通消息 → 发给配置的 LLM → 回复回群，**多轮上下文**（按群保留最近 N 条，重启不丢）
- **AI 工具调用（function calling）**：AI 可自动调用内置工具：`查询在线玩家` / `查询白名单` / `执行命令`（高危仅 `ai.admin-openids`）
- **自定义 Skill（🧩）**：在 WebUI 定义自己的 AI 技能，AI 调用后执行你配置的控制台命令（见下文）
- **零额外依赖**：内置 HTTP/HTTPS 客户端，零 npm 依赖

## 核心配置

| 配置 | 默认 | 说明 |
|---|---|---|
| `ai.enabled` | `false` | AI 总开关；`false` 时行为与普通版完全一致 |
| `ai.base-url` | 空 | OpenAI 兼容接口地址，如 `https://api.openai.com/v1` 或本地 `http://127.0.0.1:11434/v1`（Ollama） |
| `ai.api-key` | 空 | API 密钥（本地无鉴权可留空） |
| `ai.model` | `gpt-4o-mini` | 模型名（需支持 function calling，如 DeepSeek / Qwen / GPT 系列） |
| `ai.system-prompt` | 服务器管理助理 | 系统提示词（插件会自动附加工具使用引导） |
| `ai.context-limit` | `10` | 每群保留最近 N 条上下文 |
| `ai.admin-openids` | `[]` | **AI 执行控制台命令 / 管理员 Skill 的 OpenID 白名单**（高危，配你的 QQ OpenID） |
| `ai.skills` | `[]` | 自定义 Skill 列表（见下文，也可在 WebUI 配置） |
| `ai.max-tokens` / `ai.temperature` / `ai.timeout` | 1000 / 0.7 / 15000 | 采样与超时参数 |
| `admin.mode` / `admin.openids` | `both` / `[]` | 群命令管理员判定（与 AI 执行命令用的 `ai.admin-openids` 不同） |

## AI 使用示例

```json
"ai": {
  "enabled": true,
  "base-url": "https://api.openai.com/v1",
  "api-key": "sk-xxx",
  "model": "gpt-4o-mini",
  "context-limit": 10,
  "admin-openids": ["你的QQOpenID"]
}
```

群内 @机器人 说"查在线""现在谁在玩""查一下白名单"等，AI 自动调用对应工具；要 AI 执行控制台命令（如 `执行命令 list`），需把**你的 QQ OpenID 填进 `ai.admin-openids`**。

## 自定义 Skill

在 WebUI「🧩 Skill 管理」页，或直接改 `ai.skills` 配置，定义 AI 可调用的自定义技能：

```json
"ai": { "skills": [
  { "key": "server_status", "name": "服务器状态", "desc": "查看服务器内存/性能", "command": "mem", "permission": 0 },
  { "key": "kick", "name": "踢出玩家", "desc": "将玩家踢出服务器", "command": "kick {0} 你被移除了", "permission": 1 }
]}
```

| 字段 | 说明 |
|---|---|
| `key` | Skill 名（AI 工具名，唯一，自动注册为 `skill__<key>`） |
| `name` | 显示名（可选） |
| `desc` | 给 AI 看的描述，说明这个技能做什么 |
| `command` | 控制台命令模板；`{0}`/`{1}`/`{params}` 为参数占位符（由 AI 按顺序提供），支持 `{group}`/`{user}` |
| `permission` | `0` 所有人可调；`1` 仅 `ai.admin-openids` 管理员可调 |

示例：定义 `kick`（`kick {0} 你被移除了`）后，群里说"把 Steve 踢了"，AI 会调用 `skill__kick` 执行 `kick Steve 你被移除了`（`permission:1` 时还需你的 OpenID 在 `ai.admin-openids`）。

## WebUI 管理面板

开启后访问 `http://127.0.0.1:8088`：**深色侧边栏 + 多页面**管理后台，无需改配置文件或进服。

| 配置 | 默认 | 说明 |
|---|---|---|
| `webui.enabled` | `false` | 是否启用 WebUI（Node http，lse-nodejs 可用） |
| `webui.host` | `127.0.0.1` | 监听地址：`127.0.0.1`=仅本机访问；`0.0.0.0`=监听所有网卡，**外网可访问**（务必配 `webui.password`） |
| `webui.port` | `8088` | 监听端口 |
| `webui.username` / `webui.password` | `admin` / 空 | 登录凭据；**不填密码则无登录校验**（仅本机可安全访问） |

页面功能：
- **📈 总览**：配置版本、可用工具数、快捷开关（AI / Markdown 一键切换）
- **🛠️ AI 工具**：展示 AI 可用工具与权限
- **🧩 Skill 管理**：增删改自定义 Skill（key/名称/权限/描述/命令模板），保存自动生效
- **💬 AI 对话**：发消息测试 AI（支持工具调用）
- **⚙️ 配置**：分组表单编辑全部配置，**保存后自动热重载生效**
- 移动端自动折叠为汉堡菜单

> 仅绑定 `127.0.0.1`；如需外网访问，请在宿主机/反向代理层自行加登录保护。

## 其余功能（同普通版）

- **双向转发**：游戏 `#消息` ↔ QQ 群；20+ 群指令（查在线/白名单/管理员/认证等）
- **📋 Markdown 卡片**：查在线 / 查白名单 / `motd` 命令（`motd.use-markdown` 总开关）
- **MOTD 状态图 + 自定义模板**：`Markdown/online.md` 可编辑、`motd.api`/`motd.text` 模板
- **敏感词审核**：正则 + 本地词库 + OpenAI 二审
- **控制台命令**：`huhobot reload` / `huhobot info`

## 部署

1. 安装 LSE Node 引擎：`lip install github.com/LiteLDev/LegacyScriptEngine`
2. 放入 `plugins/HuHoBotPenguin-LLSE-Llama/`
3. 填 `bot.app-id` / `bot.secret`；AI 版额外填 `ai.*`
4. 重启服务器 → 控制台 `QQ 机器人已连接`

## License

[MIT](LICENSE) © 2026 HuHoBot
