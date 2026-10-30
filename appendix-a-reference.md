# 附录 A 速查表

[← 上一章](11-internals.md) · [目录](README.md)

本附录汇总常用命令、参数、设置和路径。Pi 迭代较快，安装版本的权威参考始终是 `pi --help` 和 `pi <command> --help`。

## A.1 命令行选项

```sh
pi [options] [--] [@files...] [messages...]
```

| 选项 | 作用 |
|---|---|
| `-p`, `--print` | 运行提示词，输出最终文本后退出 |
| `--mode text\|json\|rpc` | 选择输出接口；`text` 不强制一次运行 |
| `--model <pattern>` | 按 ID、模糊名称或 `provider/id` 选择模型；可带 `:<thinking>` |
| `--provider <name>` | 限定 `--model` 的提供商；必须与 `--model` 同用 |
| `--api-key <key>` | 本次运行使用的 API Key，不持久化 |
| `--thinking <level>` | `off`、`minimal`、`low`、`medium`、`high`、`xhigh`、`max` |
| `--models <patterns>` | 逗号分隔的模型范围，用于启动与 `Ctrl+P` 循环 |
| `--list-models [search]` | 列出模型后退出 |
| `-c`, `--continue` | 继续当前目录最近的会话 |
| `-r`, `--resume` | 打开会话选择器 |
| `--session <path\|id>` | 按文件路径、精确 ID 或部分 ID 打开会话 |
| `--session-id <id>` | 打开精确的项目会话 ID，不存在则创建 |
| `--fork <path\|id>` | 从已有会话分叉出新会话 |
| `--session-dir <dir>` | 覆盖会话存储与查找目录 |
| `--no-session` | 使用不持久化的内存会话 |
| `-n`, `--name <name>` | 设置会话显示名 |
| `-t`, `--tools <list>` | 工具允许列表；纯名称替换默认集，`+name`/`-name` 增减 |
| `-xt`, `--exclude-tools <list>` | 禁用指定工具名或通配模式，MCP 工具也适用 |
| `-nbt`, `--no-builtin-tools` | 禁用内置工具，保留扩展/自定义工具 |
| `-nt`, `--no-tools` | 禁用所有工具 |
| `-e`, `--extension <path>` | 加载扩展文件、目录或 `builtin:<name>`，可重复 |
| `-ne`, `--no-extensions` | 禁用已发现、已配置和内置扩展；显式 `-e` 仍生效 |
| `--no-mcp` | 本次运行禁用内置 MCP 支持 |
| `--skill <path>` | 加载技能，可重复 |
| `-ns`, `--no-skills` | 禁用技能发现与加载 |
| `--prompt-template <path>` | 加载提示词模板，可重复 |
| `-np`, `--no-prompt-templates` | 禁用模板发现与加载 |
| `--theme <path>` | 加载主题，可重复 |
| `--use-theme <name[/name]>` | 本次运行的初始主题 |
| `--no-themes` | 禁用主题发现与加载 |
| `-nc`, `--no-context-files` | 禁用 `AGENTS.md`/`CLAUDE.md` 发现 |
| `--system-prompt <text\|path>` | 替换默认系统提示词 |
| `--append-system-prompt <text\|path>` | 追加系统提示词，可重复 |
| `--tui-mode <mode>` | `fullscreen`（默认）或 `regular` |
| `-a`, `--approve` | 本次运行信任项目资源 |
| `-na`, `--no-approve` | 本次运行忽略受信任控制的项目资源 |
| `--offline` | 禁用自动网络活动，等同 `PI_OFFLINE=1` |
| `--export <input> [output]` | 把会话文件导出为 HTML 后退出 |
| `--verbose` | 显示详细启动信息 |
| `-h`, `--help` | 显示帮助 |
| `-v`, `--version` | 显示版本 |

约束与组合：

- `--provider` 必须与 `--model` 同时使用；`--api-key` 要求已通过 `--model` 或 `--models` 选定模型。
- `--fork` 不能与 `--session`、`--continue`、`--resume`、`--no-session` 同用。
- `--session-id` 不能与 `--session`、`--continue`、`--resume` 同用；可与 `--fork` 组合指定新 ID。
- 会话 ID 必须以字母或数字开头和结尾，可包含字母、数字、`.`、`_`、`-`。
- `--tools` 的纯名称列表会替换默认选择；只含 `+name`/`-name` 的列表在默认选择上增减；两者不能混用。`--exclude-tools` 最后生效，也作用于 MCP 工具。
- RPC 模式拒绝 `@file` 参数；JSON 和 RPC 模式的 stdout 只用于协议记录。

## A.2 子命令

| 命令 | 作用 |
|---|---|
| `pi install <source>` | 安装 Pi 包 |
| `pi list` | 列出已配置的 Pi 包 |
| `pi remove <source>` | 移除包及设置条目；`uninstall` 是别名 |
| `pi update` | 更新 Pi |
| `pi update --extensions` | 更新所有已安装包 |
| `pi update <source>` | 更新指定包 |
| `pi update --models` | 刷新模型目录 |
| `pi update --all` | 更新 Pi 与所有包 |
| `pi config` | 启用/禁用包资源与内置扩展 |
| `pi auth check` | 检查提供商凭据状态 |
| `pi auth print-api-key` | 输出解析后的 API Key |
| `pi auth print-bearer-token` | 输出 OAuth bearer token |
| `pi mcp add/remove/list/login/logout` | 管理 MCP 服务器 |

`install`、`remove`、`uninstall`、`config` 支持 `--local`/`-l`，作用于项目 `.pi/settings.json`。

## A.3 斜杠命令

| 命令 | 作用 |
|---|---|
| `/settings` | 打开设置 |
| `/model [provider/model]` | 选择模型 |
| `/thinking [level]` | 设置思考等级 |
| `/scoped-models` | 配置模型循环范围 |
| `/login [provider]`、`/logout` | 管理提供商认证 |
| `/llama` | 管理 llama.cpp 路由器模型 |
| `/new`、`/resume` | 新建、切换会话 |
| `/name [name]` | 设置或查看会话名 |
| `/session` | 显示会话文件、ID、消息数、用量和费用 |
| `/tree` | 在会话树中导航 |
| `/fork`、`/clone` | 分叉、克隆会话 |
| `/compact [instructions]` | 压缩上下文 |
| `/import <path>` | 导入并继续 JSONL 会话 |
| `/copy` | 复制最后一条助手消息 |
| `/export [path]` | 导出 HTML 或 JSONL |
| `/share` | 上传会话并获取链接 |
| `/bug [description]` | 准备私有问题报告 |
| `/trust` | 保存项目信任决定 |
| `/mcp` | 管理 MCP 服务器 |
| `/reload` | 重新加载快捷键、扩展、技能、模板、主题、上下文文件 |
| `/hotkeys` | 显示当前快捷键 |
| `/changelog` | 显示变更记录 |
| `/quit` | 退出 |

扩展可以注册命令；提示词模板以文件名作为命令；技能以 `/skill:name` 提供。

## A.4 常用设置

| 键 | 默认值 | 说明 |
|---|---|---|
| `defaultProvider` | 自动 | 启动提供商 |
| `defaultModel` | 自动 | 启动模型 |
| `defaultThinkingLevel` | `"medium"` | 启动思考等级 |
| `enabledModels` | 全部 | 模型循环范围 |
| `hideThinkingBlock` | `false` | 隐藏思考块 |
| `defaultTools` | `read`、`bash`、`edit`、`write` | 启动工具集 |
| `codemode.mode` | `"on"` | `on` 或 `only` |
| `codemode.inlineBudget` | `3000` | 工具声明估算 token 预算 |
| `sessionDir` | agent 会话目录 | 会话存储目录 |
| `sessionShareDestination` | `"auto"` | `auto`、`gist`、`radius` |
| `compaction.enabled` | `true` | 自动压缩 |
| `compaction.reserveTokens` | `16384` | 触发压缩的剩余 token |
| `compaction.keepRecentTokens` | `20000` | 保留不总结的近期 token |
| `branchSummary.skipPrompt` | `false` | 跳过分支摘要询问 |
| `theme` | `"system"` | 主题名或 `light/dark` |
| `tuiMode` | `"fullscreen"` | 终端界面模式 |
| `quietStartup` | `false` | 隐藏启动信息 |
| `defaultProjectTrust` | `"ask"` | 只能写在 agent 目录 |
| `shellPath` | 平台默认 | 自定义 shell |
| `shellCommandPrefix` | 无 | 每条命令前置 |
| `retry.enabled` | `true` | 智能体级自动重试 |
| `retry.maxRetries` | `3` | 最大重试次数 |
| `retry.baseDelayMs` | `2000` | 初始退避延迟 |
| `transport` | `"auto"` | `auto`、`sse`、`websocket`、`websocket-cached` |
| `httpProxy` | 无 | 只能写在 agent 目录 |
| `steeringMode` | `"one-at-a-time"` | 转向消息投递方式 |
| `followUpMode` | `"one-at-a-time"` | 后续消息投递方式 |
| `externalEditor` | `$VISUAL`/`$EDITOR` | 外部编辑器命令 |
| `doubleEscapeAction` | `"tree"` | 空编辑器双击 Esc 的动作 |
| `terminal.showImages` | `true` | 是否显示内联图片 |
| `images.autoResize` | `true` | 发送前把图片缩到最大 2000×2000 |
| `markdown.mermaid` | `"streaming"` | Mermaid 渲染模式 |

设置文件可加 `"$schema": "https://pi.dev/schemas/settings.schema.json"`。

## A.5 环境变量

| 变量 | 作用 |
|---|---|
| `PI_CODING_AGENT_DIR` | agent 目录，默认 `~/.pi/agent` |
| `PI_CODING_AGENT_SESSION_DIR` | 会话目录，低于 `--session-dir` |
| `PI_PACKAGE_DIR` | 覆盖包目录，供 Nix/Guix 使用 |
| `PI_OFFLINE` | 禁用自动网络活动 |
| `PI_SKIP_VERSION_CHECK` | 禁用版本检查 |
| `PI_TELEMETRY` | 覆盖安装/更新遥测 |
| `PI_CACHE_RETENTION` | `long` 启用更长提示缓存 |
| `PI_SHARE_VIEWER_URL` | 覆盖 `/share` 的查看地址 |
| `PI_CODEMODE_MAX_WORKERS` | Codemode worker 上限，默认 2 |
| `PI_CLEAR_ON_SHRINK` | `1` 在内容缩短时清理空行 |
| `PI_HARDWARE_CURSOR` | `1` 显示硬件光标 |
| `PI_CLIPBOARD_OSC52` | `1` 同时通过 OSC 52 发送复制内容 |
| `PI_HYPERLINKS` | `1`、`0`、`auto` |
| `PI_IMAGE_PROTOCOL` | `kitty`、`iterm2`、`none`、`auto` |
| `PI_TRUE_COLOR` | `1`、`0`、`auto` |
| `PI_TUI_ESC_TIMEOUT` | 单独 ESC 的等待毫秒数 |
| `VISUAL`、`EDITOR` | 外部编辑器回退 |
| `HTTP_PROXY`、`HTTPS_PROXY` | 出站 HTTP 代理 |

模型调用的 `bash`/`powershell` 工具额外收到：

| 变量 | 内容 |
|---|---|
| `PI_SESSION_ID` | 当前会话 ID |
| `PI_SESSION_FILE` | 会话 JSONL 绝对路径；内存会话不设 |
| `PI_PROVIDER` | 当前提供商 |
| `PI_MODEL` | 当前模型 ID |
| `PI_REASONING_LEVEL` | 当前推理等级 |

## A.6 常用快捷键

| 按键 | 作用 |
|---|---|
| `Enter` | 发送 |
| `Shift+Enter` | 换行 |
| `Ctrl+G` | 外部编辑器 |
| `Tab` | 路径补全 |
| `Ctrl+O` | 展开/折叠工具输出 |
| `Ctrl+T` | 显示/隐藏思考块 |
| `Ctrl+L` | 模型选择器 |
| `Ctrl+P` | 循环模型 |
| `Shift+Tab` | 循环思考等级 |
| `Esc` | 中止当前运行 |
| `Alt+Enter` | 排队后续消息（Windows/WSL：`Ctrl+Q`） |
| `Alt+Up` | 取回排队消息（Windows/WSL：`Alt+Q`） |
| `Ctrl+X` | 复制最后一条助手消息 |
| `Ctrl+C` | 第一次清空编辑器，第二次退出 |
| `Ctrl+D` | 空编辑器时退出 |

完整列表见 `/hotkeys` 和官方 Keybindings 页面。

## A.7 文件与路径

| 路径 | 内容 |
|---|---|
| `~/.pi/agent/settings.json` | 用户设置 |
| `~/.pi/agent/keybindings.json` | 快捷键 |
| `~/.pi/agent/mcp.json` | 用户级 MCP 服务器 |
| `~/.pi/agent/models.json` | 兼容端点与模型覆盖 |
| `~/.pi/agent/auth.json` | 凭据 |
| `~/.pi/agent/trust.json` | 项目信任决定 |
| `~/.pi/agent/sessions/` | 会话文件 |
| `~/.pi/agent/extensions/` | 用户扩展 |
| `~/.pi/agent/skills/` | 用户技能 |
| `~/.pi/agent/prompts/` | 用户提示词模板 |
| `~/.pi/agent/themes/` | 用户主题 |
| `.pi/settings.json` | 项目设置 |
| `.pi/mcp.json` | 项目 MCP 服务器 |
| `.pi/extensions/`、`.pi/skills/`、`.pi/prompts/`、`.pi/themes/` | 项目资源 |
| `.pi/SYSTEM.md`、`.pi/APPEND_SYSTEM.md` | 项目系统提示词 |
| `AGENTS.md`、`CLAUDE.md`、`AGENTS.override.md` | 上下文文件 |

## A.8 常用命令组合

只读审查，不允许改文件：

```bash
pi --tools read,grep,find,ls -p "审查 src/ 的错误处理，只报告问题"
```

结构化运行并提取工具调用：

```bash
pi --mode json "运行测试并总结失败项" 2>/dev/null \
  | jq -r 'select(.type == "tool_execution_start") | .toolName' \
  | sort | uniq -c
```

继续指定会话：

```bash
pi --session <id-or-path>
pi --continue
```

临时试用一个包，不写入设置：

```bash
pi -e npm:@example/pi-tools
```

最小运行（只读工具，仍不是沙箱）：

```bash
pi --no-session --no-extensions --tools read,grep,find,ls -p "回答关于仓库的问题"
```

导出会话：

```bash
pi --export ~/.pi/agent/sessions/--path--/session.jsonl report.html
```

## A.9 运行模式与退出

| 模式 | 选择方式 | 进程何时退出 | 成功判断 |
|---|---|---|---|
| 交互 | 默认（TTY） | `/quit`、Ctrl+D、两次 Ctrl+C | 不适用 |
| 打印 | `-p`/`--print`，或非 TTY | 提示词处理完 | 退出码 0 |
| JSON | `--mode json` | 提示词处理完 | 事件中的 `stopReason`、`agent_settled.aborted` |
| RPC | `--mode rpc` | 关闭 stdin 或收到退出请求 | 响应 `success` 与事件状态 |
| SDK | `createAgentSession()` | 由宿主决定；不再使用时调用 `dispose()` | `prompt()` 完成加最终消息状态 |

注意：

- 打印模式在助手回复 `error`、`aborted` 或输出被 `length` 截断时返回非零。
- JSON 模式中，助手层面的失败不一定导致非零退出码；要检查事件。
- RPC 的 `prompt` 成功响应只表示命令被接受；完成要等 `agent_settled`。
- SDK 的 `prompt()` resolve 表示运行结束，不保证任务成功。

## A.10 官方文档索引

| 主题 | 地址 |
|---|---|
| 总览 | <https://pi.dev/docs/latest> |
| 快速开始 | <https://pi.dev/docs/latest/quickstart> |
| 工作原理 | <https://pi.dev/docs/latest/how-pi-works> |
| 终端使用 | <https://pi.dev/docs/latest/usage> |
| 命令行 | <https://pi.dev/docs/latest/cli> |
| 模型 | <https://pi.dev/docs/latest/models> |
| 提供商 | <https://pi.dev/docs/latest/providers> |
| 会话 | <https://pi.dev/docs/latest/sessions> |
| 会话格式 | <https://pi.dev/docs/latest/session-format> |
| 压缩 | <https://pi.dev/docs/latest/compaction> |
| 配置 | <https://pi.dev/docs/latest/configuration> |
| 设置 | <https://pi.dev/docs/latest/settings> |
| 安全 | <https://pi.dev/docs/latest/security> |
| 环境变量 | <https://pi.dev/docs/latest/environment-variables> |
| 提示词模板 | <https://pi.dev/docs/latest/prompt-templates> |
| 技能 | <https://pi.dev/docs/latest/skills> |
| 主题 | <https://pi.dev/docs/latest/themes> |
| Pi 包 | <https://pi.dev/docs/latest/packages> |
| MCP | <https://pi.dev/docs/latest/mcp> |
| Codemode | <https://pi.dev/docs/latest/codemode> |
| 扩展 | <https://pi.dev/docs/latest/extensions> |
| CLI 集成 | <https://pi.dev/docs/latest/cli-integration> |
| JSON 事件流 | <https://pi.dev/docs/latest/json> |
| RPC | <https://pi.dev/docs/latest/rpc> |
| SDK | <https://pi.dev/docs/latest/sdk> |
| 快捷键 | <https://pi.dev/docs/latest/keybindings> |
| 终端配置 | <https://pi.dev/docs/latest/terminal-setup> |
| 隔离 | <https://pi.dev/docs/latest/containerization> |
| 自定义提供商 | <https://pi.dev/docs/latest/custom-provider> |
| 虚拟模型 | <https://pi.dev/docs/latest/virtual-models> |

---

[目录](README.md) · [下一章 →](appendix-b-glossary.md)
