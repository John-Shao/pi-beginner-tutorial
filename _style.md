# 《Pi 入门教程》写作规范（内部文件，不属于正文）

本文件规定教程的定位、事实来源、结构与用语。修改正文前先读本文件；规范未覆盖的问题，按“先查官方文档、再查源码、拿不准就删”的顺序处理。

## 1. 教程定位

- 读者：第一次接触 Pi 的开发者与技术人员。会使用命令行，了解 Git、API Key、大模型 API 的基本概念，但不了解 Pi。
- 目标：读完后能独立安装、配置、日常使用 Pi；能用提示词模板、技能、Pi 包和 MCP 定制工作流；能编写简单扩展；能通过打印、JSON、RPC 或 SDK 把 Pi 接进其他程序；并具备阅读 Pi 源码的入口知识。
- 语言：简体中文。术语首次出现时给出英文原文，例如“会话（session）”。
- 版本基线：`@earendil-works/pi-coding-agent` 1.4.0，源码 commit `f09d01b8`（2026-10-30）。
- 语气：教学口吻，直接、具体、少形容词。不用“让我们一起”式口水话，不堆感叹号，不写营销文案。

## 2. 事实来源与优先级

1. 官方文档源码：`pi/packages/coding-agent/docs/*.md`，与 <https://pi.dev/docs/latest> 同源。正文引用在线地址 `https://pi.dev/docs/latest/<slug>`，slug 为文档文件名去掉 `.md`（`index.md` 对应 `/docs/latest`）。
2. 源码：`pi/packages/**`。用于解释原理、补充文档未写明的行为、给出“源码位置”。
3. 文档与代码冲突时以代码为准，正文只写实际行为，并在此文件中记录差异。
4. 不确定的细节：删掉，或写成“由实现决定，见源码 xxx”。禁止编造参数、默认值、文件路径和 API 签名。

## 3. 章节文件与编号

| 文件 | 标题 |
|---|---|
| `00-preface.md` | 前言：如何使用本教程 |
| `01-what-is-pi.md` | 第 1 章 认识 Pi |
| `02-install.md` | 第 2 章 安装、登录与完成第一个任务 |
| `03-terminal.md` | 第 3 章 终端交互：高效驱动 Pi |
| `04-models.md` | 第 4 章 模型、提供商与思考等级 |
| `05-sessions.md` | 第 5 章 会话、分支与上下文管理 |
| `06-configuration.md` | 第 6 章 配置、项目约定与安全边界 |
| `07-customize.md` | 第 7 章 用资源定制 Pi |
| `08-mcp-codemode.md` | 第 8 章 MCP 与 Codemode：接入外部工具 |
| `09-extensions.md` | 第 9 章 开发扩展 |
| `10-automation.md` | 第 10 章 自动化与嵌入 |
| `11-internals.md` | 第 11 章 源码导读与进阶路线 |
| `appendix-a-reference.md` | 附录 A 速查表 |
| `appendix-b-glossary.md` | 附录 B 术语表 |
| `appendix-c-troubleshooting.md` | 附录 C 故障排查 |

正文第一行是一级标题，格式严格为 `# 第 N 章 标题`，前面不加 HTML、徽章或引言。附录为 `# 附录 A 速查表`。

## 4. 每章结构模板

```markdown
# 第 N 章 标题

[← 上一章](00-preface.md) · [目录](README.md)

> **本章你将学会**：一句话列出 3～5 项能力。
>
> **对应官方文档**：[文档名](https://pi.dev/docs/latest/sessions) · [另一篇](https://pi.dev/docs/latest/cli)

引言 2～4 段：这一章解决什么问题，为什么重要，和前后章节的关系。

## N.1 小节标题
正文……

## N.x 小结
3～6 条要点，每条一句话。

## 练习
1. 3～6 个可立即动手验证的练习，每个练习给出验收标准。
2. …

## 延伸阅读
- [官方文档：sessions](https://pi.dev/docs/latest/sessions)
- 源码：`packages/coding-agent/src/…`

---

[目录](README.md) · [下一章 →](02-install.md)
```

- 小节编号与章号一致（第 5 章为 `## 5.1`）。最多到三级标题（`###`）；需要更细的层次用加粗词或列表。
- 每章正文目标 2500～4500 个汉字（代码与表格不计）。宁可写透一个点，不要罗列十个点。
- 章末“小结 / 练习 / 延伸阅读”为必需项。
- 每页必须有顶部“上一章/目录”和底部“目录/下一章”导航；前言与最后一页按特殊情况处理。

## 5. 术语与译名

| 英文 | 中文（首次出现写法） | 说明 |
|---|---|---|
| Pi | Pi | 不翻译 |
| agent | 智能体 | “AI agent” → 智能体 |
| agent harness | 智能体运行框架 | 只在第 1 章解释 |
| session | 会话（session） | 后文可简称会话 |
| branch | 分支（branch） | 与 Git 分支区分时写“会话分支” |
| context | 上下文（context） | |
| prompt | 提示词 | “prompt template” → 提示词模板 |
| system prompt | 系统提示词 | |
| tool call | 工具调用 | |
| tool | 工具 | |
| provider | 提供商 | 模型服务商 |
| model | 模型 | |
| thinking level | 思考等级 | |
| compaction | 压缩（compaction） | 会话上下文压缩 |
| skill | 技能（skill） | |
| extension | 扩展（extension） | |
| theme | 主题 | |
| Pi package | Pi 包 | 不写“软件包”以免与 npm 包混淆 |
| print mode | 打印模式 | |
| JSON mode | JSON 模式 | |
| RPC mode | RPC 模式 | |
| turn | 回合 | 一次模型请求加其工具执行 |
| project trust | 项目信任 | |
| slash command | 斜杠命令 | |
| keybinding | 快捷键 | |
| MCP server | MCP 服务器 | |
| codemode | Codemode | 保留英文，首字母大写 |
| SDK | SDK | |
| token | token | 不译为“令牌” |
| TUI | 终端界面（TUI） | |

- 命令、文件名、配置键、环境变量、代码标识符保留英文并用反引号。
- 中文标点用全角；中英文之间加空格，例如“使用 `pi` 命令启动”。
- 不用 emoji、颜文字；正文不出现“我们”。

## 6. 代码、命令与表格

- 代码块语言标记：`bash`/`sh`（shell）、`powershell`、`text`（交互输入或输出）、`json`、`jsonl`、`typescript`、`js`、`python`、`markdown`。
- 每条命令必须能直接复制执行；交互界面里的输入用 `text` 块，并在前后说明“在 Pi 的编辑器里输入”。
- 输出示例标注“示例输出”，不得让人误以为是固定输出。
- 表格用于对照关系（参数、模式、机制选型），不用表格代替叙述性解释。
- 路径只能来自文档或源码。仓库相对路径写作 `packages/coding-agent/src/...`，用户目录写作 `~/.pi/agent/`。
- 结构、时序和决策关系优先用 Mermaid 图表表达，节点文字保持简短，图前用一句话说明它回答什么问题。
- 终端截图只使用官方文档中的素材，存入 `assets/`，图下注明来源和“界面可能随版本变化”；不要用装饰性图片暗示不存在的功能。

## 7. 引用与“源码位置”

- 每章至少 2 处“源码位置”提示，格式：

  > 源码位置：`packages/coding-agent/src/core/agent-session.ts`（会话生命周期）、`packages/agent/src/agent-loop.ts`（智能体循环）。

- 关键结论后附文档链接，格式：`详见[会话与上下文](https://pi.dev/docs/latest/sessions)。`
- 不复制大段文档原文（连续超过 5 行必须改写）；不引用 GitHub blob 链接，在线文档只用 `https://pi.dev/docs/latest/<slug>`。

## 8. 交叉引用

- 引用其他章写“见第 5 章”；引用小节写“见 5.3 节”。
- 每页 H1 之后加顶部导航：`[← 上一章](00-preface.md) · [目录](README.md)`；前言使用 `[← 目录](README.md)`。
- 每页末尾加底部导航：`[目录](README.md) · [下一章 →](02-install.md)`；最后一页使用 `[返回目录](README.md)`。
- 正文不链接其他章文件路径，只用文字引用；顶部和底部导航是唯一例外。
- 前言与第 1 章给出全文地图，其余各章不重复整体架构。

## 9. 写作红线

禁止：

1. 编造 CLI 参数、默认值、环境变量、配置键、文件路径、API 签名。
2. 把“计划中”的能力写成已发布功能。
3. 正文出现 TODO、待补、“此处省略”、占位符。
4. 大段复制官方文档。
5. 使用本仓库之外的第三方教程内容。
6. 修改 `pi/` 仓库中的源码。
7. 正文使用 emoji 或营销腔。

必须：

1. 每个新机制先一句话定义，再给例子，最后解释为什么这样设计。
2. 命令行示例与 `docs/cli.md` 或 `packages/coding-agent/src/cli/args.ts` 核对参数名。
3. 扩展 API 签名与 `docs/extensions.md` 或 `src/core/extensions/types.ts` 一致。
4. 完稿后通读：读者能否只靠本章完成任务？前后是否矛盾？

## 10. 版本差异记录

写作过程中未发现文档与代码矛盾。若后续发现，记录在此处并在正文中只写实际行为。
