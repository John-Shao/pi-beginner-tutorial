# 附录 B 术语表

[← 上一章](appendix-a-reference.md) · [目录](README.md)

按主题分组。首次出现时建议同时写出中文和英文，后文可只用中文。

## B.1 智能体与运行

**智能体（agent）**：能调用模型和工具、根据结果继续行动的软件实体。Pi 本身是一个智能体运行框架。

**智能体运行框架（agent harness）**：连接模型、工具、上下文和会话的运行时。它决定何时请求模型、如何执行工具、如何保存历史。

**智能体循环（agent loop）**：请求模型、执行工具、把结果送回模型、重复，直到没有新工具调用或收到终止条件。实现位于 `packages/agent/src/agent-loop.ts`。

**运行（agent run）**：一次低层智能体循环从开始到结束。一次用户任务可能包含多次运行，例如自动重试、压缩后重试或排队消息触发的新运行。

**回合（turn）**：一次模型请求加上模型返回的工具调用及其执行结果。

**转向（steering）**：在 Pi 工作时插入的消息。它在当前助手回合及工具调用完成后、下一次模型请求前生效。

**后续（follow-up）**：在 Pi 工作时排队的消息。它等整个任务空闲后才处理。

**工具调用（tool call）**：模型请求执行某个工具，并给出符合该工具参数 schema 的 JSON 参数。

**工具（tool）**：模型可调用的函数，例如 `read`、`bash`、`edit`、`write`。工具有名称、描述、参数 schema 和执行函数。

**终止（terminate）**：工具结果中的提示，表示当前工具批次完成后可以跳过后续模型调用。仅当批次中所有最终结果都终止时才生效。

## B.2 会话与上下文

**会话（session）**：Pi 对一次对话的持久化记录，包含消息、工具调用与结果、模型切换、压缩等条目。

**条目（entry）**：会话文件中的一行 JSON，有 `type`、`id`、`parentId` 和时间戳。条目组成树。

**会话头（session header）**：会话文件第一行，记录版本、会话 ID、时间戳和工作目录，不属于树。

**会话树（session tree）**：由 `id`/`parentId` 连接的条目结构。回到较早节点继续会产生新分支。

**分支（branch）**：会话树中从根到某个节点的路径。与 Git 分支无关时也可写“会话分支”。

**活动分支（active branch）**：从根到当前叶子的路径，决定下一次模型请求的历史。

**叶子（leaf）**：会话树中当前所在的末端条目。切换分支就是移动叶子。

**上下文（context）**：下一次模型请求实际携带的内容，包括系统提示词、活动分支消息、工具定义和技能描述。

**上下文窗口（context window）**：模型单次请求能接受的最大 token 数，由模型目录声明。

**压缩（compaction）**：把较早的上下文总结成一条摘要，替换后续请求中的旧消息；原始条目仍保存在会话文件中。

**压缩边界（firstKeptEntryId）**：压缩条目中记录的保留起点。模型上下文从该条目开始保留，之前的消息由摘要替代。

**分支摘要（branch summary）**：离开一条会话分支时生成的摘要，用于把被放弃路径的关键信息带到新分支。

**上下文编辑（context edit）**：一种只影响未来模型上下文的追加条目，可以省略或替换较早条目的内容，不改变原始历史。

**自定义条目（custom entry）**：扩展写入的会话条目，不进入模型上下文，可用于持久化扩展状态。

**自定义消息（custom message）**：扩展注入的消息，进入模型上下文，可选择是否显示在界面中。

**标签（label）**：加在会话树节点上的书签，用于导航和筛选。

**token**：模型计费和上下文长度的基本单位。不译为“令牌”。

## B.3 模型与服务

**模型（model）**：生成回复的模型，例如某个 Claude、GPT 或 Gemini 模型。

**提供商（provider）**：Pi 访问模型的服务或账号，例如 Anthropic、OpenAI、OpenRouter、llama.cpp。

**模型目录（model catalog）**：Pi 自带的提供商、模型、能力、上下文长度和价格数据；可从 pi.dev 覆盖更新。

**思考等级（thinking level）**：控制模型推理投入的档位：`off`、`minimal`、`low`、`medium`、`high`、`xhigh`、`max`。

**虚拟模型（virtual model）**：在 `/model` 中可选、但每次请求由路由器选择物理模型的模型。

**分类器模型（classifier model）**：不聊天，只对 JSON 状态回答选择题、是/否题或评分题的模型，通常由 Codemode 调用。

**图像模型（image model）**：根据提示词和可选输入图像生成图像的模型，通常由 Codemode 调用。

**缓存预热（cache warming）**：在运行中或空闲时刷新提供商提示缓存，降低后续请求的缓存未命中成本。

**提示缓存（prompt cache）**：提供商对重复前缀的缓存机制，能降低费用和延迟。

## B.4 定制与扩展

**提示词模板（prompt template）**：带参数的 Markdown 文件，文件名成为斜杠命令，在进入智能体前展开。

**技能（skill）**：包含 `SKILL.md` 的目录，向模型提供按需加载的指令和附带文件。

**技能命令（skill command）**：`/skill:name` 形式的命令，用于强制加载某个技能。

**扩展（extension）**：加载进 Pi 进程的 TypeScript 模块，可注册事件处理器、工具、命令、快捷键、提供商等。

**Pi 包（Pi package）**：通过 npm、git 或本地路径分发的资源集合，可包含扩展、技能、提示词模板和主题。

**主题（theme）**：控制终端界面和 HTML 导出颜色的 JSON 配置。

**MCP 服务器（MCP server）**：实现 Model Context Protocol 的外部服务，向 Pi 提供工具和资源。

**工具暴露（tool exposure）**：MCP 工具到达模型的方式：`codemode`、`deferred`、`direct`、`hidden`。

**工具搜索（tool search）**：`tool_search` 工具，按相关度查找未声明的工具，并在下一次模型调用中声明匹配项。

**Codemode**：让模型编写 JavaScript 脚本调用其他工具的机制。脚本在 QuickJS 沙箱中运行，只有脚本输出到达模型。

**命名空间（namespace）**：工具分组，例如一个 MCP 服务器；Codemode 的 `describeNamespace()` 可读取其描述和工具列表。

**主机提供的包（host-provided package）**：Pi 在运行时提供给扩展的包，例如 `@earendil-works/pi-ai`、`pi-agent-core`、`pi-coding-agent`、`pi-tui`、`typebox`。扩展包应把它们声明为 `peerDependencies`。

## B.5 配置与安全

**agent 目录（agent directory）**：用户级配置目录，默认 `~/.pi/agent`，可用 `PI_CODING_AGENT_DIR` 覆盖。

**项目配置（project configuration）**：工作目录下 `.pi/` 中的设置和资源，在项目被信任后加载。

**上下文文件（context file）**：`AGENTS.md`、`CLAUDE.md` 等指令文件，从 agent 目录、工作目录和父目录发现。

**项目信任（project trust）**：Pi 是否加载项目提供的设置、扩展、技能、MCP 服务器等资源的决定。

**信任决定（trust decision）**：`yes`、`no` 或未决定；可保存到 `~/.pi/agent/trust.json`。

**沙箱（sandbox）**：限制进程可访问文件、网络和凭据的操作系统或虚拟化边界。Pi 本身不提供内置沙箱。

**提示词注入（prompt injection）**：不可信内容中的指令影响模型行为，使其执行原本不应执行的操作。

**隔离（isolation）**：把 Pi 或它的工具放进容器、虚拟机或受限环境，缩小错误或恶意操作的影响范围。

## B.6 接口

**终端界面（TUI）**：Pi 的交互模式，包含对话记录、编辑器和状态栏。

**打印模式（print mode）**：运行提示词并输出最终文本后退出，适合脚本。

**JSON 模式（JSON mode）**：把会话事件写成 JSONL 后退出，适合结构化消费。

**RPC 模式（RPC mode）**：通过 stdin/stdout 的 JSONL 命令和事件长期控制 Pi 子进程。

**SDK**：`@earendil-works/pi-coding-agent` 的 TypeScript 接口，在 Node.js 或 Bun 进程内创建和控制智能体会话。

**会话管理器（SessionManager）**：负责条目树、活动叶子和模型上下文重建的对象。

**智能体会话（AgentSession）**：coding-agent 的核心对象，持有会话、模型、工具、队列、压缩状态和扩展运行时。

**运行模式（mode）**：扩展上下文中的 `ctx.mode`：`"tui"`、`"rpc"`、`"json"`、`"print"`。

---

[目录](README.md) · [下一章 →](appendix-c-troubleshooting.md)
