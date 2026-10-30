# 附录 C 故障排查

[← 上一章](appendix-b-glossary.md) · [目录](README.md)

按现象组织。先做最小化验证，再逐项排除。Pi 的诊断信息通常写在 stderr 或 agent 目录的日志中，不在 stdout。

```mermaid
flowchart LR
    A["版本与启动"] --> B["模型与认证"]
    B --> C["终端交互"]
    C --> D["工具与会话"]
    D --> E["MCP 与 Codemode"]
    E --> F["包、扩展与分享"]
```

大多数问题可以在前面的层解决；只有前面的层都正常时，才继续往后查。

## C.1 安装与启动

**`pi: command not found`**

1. 确认安装方式：安装脚本、npm 全局安装或 Nix。
2. npm 全局安装后，检查 `npm prefix -g` 对应的 bin 目录是否在 `PATH` 中。
3. 重新打开终端，让新的 `PATH` 生效。
4. 运行 `pi --version` 验证。

**提示 Node.js 版本过低**

Pi 要求 Node.js 22.19 或更高。运行 `node --version` 核对。使用安装脚本时，脚本可以在需要时安装 Node；使用 npm 或从源码运行时，需要自己升级 Node。

**启动后直接进入打印模式，没有终端界面**

当 stdin 或 stdout 被重定向时，Pi 默认使用打印模式。检查是否把输出管道给了 `less`、`tee` 或日志文件，或是否在非交互环境（CI、某些 IDE 任务）中运行。交互使用必须在真实终端中直接运行 `pi`。

**项目资源没有加载**

1. 确认项目中有 `.pi/settings.json`、`.pi/extensions/` 等受信任资源。
2. 启动时是否拒绝了信任询问？运行 `/trust` 保存决定，或用 `--approve` 单次信任。
3. 自动化模式下没有交互询问：设置 `defaultProjectTrust` 或显式传 `--approve`/`--no-approve`。
4. 确认启动目录是项目根目录；Pi 从工作目录发现资源。

**启动时报扩展加载失败**

1. 用 `pi -ne` 禁用所有扩展，确认问题是否由扩展引起。
2. 用 `pi -e ./path.ts` 单独加载可疑扩展。
3. 检查扩展工厂是否在启动阶段抛错、导入路径是否存在、依赖是否安装。
4. 查看启动诊断和 stderr；必要时运行 `/debug`。

## C.2 模型与认证

**`/model` 中没有想要的模型**

1. 运行 `/login` 确认该提供商已认证。
2. 确认环境变量是在启动 Pi 的同一个 shell 中设置的。
3. 运行 `pi auth check --provider <name> --json` 查看凭据状态。
4. 自定义模型可能已从 `models.json` 加载，但没有可用凭据时不会出现在选择器中。
5. 刷新目录：`pi update --models`；离线环境检查缓存。

**一个 shell 能用，另一个 shell 不能用**

多数情况是 API Key 来自环境变量，而不是 `auth.json`。环境变量只在启动 Pi 的进程中存在。要么在每个 shell 中设置，要么用 `/login` 保存凭据。

**远程机器上登录时浏览器打开在别处**

OAuth 回调可能无法到达远程进程。按提示把最终重定向 URL 或授权码粘贴回 Pi。部分提供商支持无头流程，具体见 Providers 页面。

**兼容端点拒绝请求**

1. 检查 `models.json` 中的 `api` 是否与服务实际协议一致，例如 `openai-completions`。
2. 检查 `baseUrl` 是否包含正确的版本路径。
3. 确认 `apiKey` 能解析；占位密钥只适用于不校验密钥的服务。
4. 打开 `/model` 重新加载文件，或重启 Pi。
5. 兼容性开关只在确认端点行为差异时启用，不要仅凭“声称兼容”就打开。

**请求频繁超时或被限流**

1. 查看提供商状态和配额。
2. 调整 `retry.maxRetries`、`retry.baseDelayMs`，但不要把提供商级重试设得太高，否则会延迟 Pi 处理配额错误。
3. 切换模型或提供商。
4. 检查 `HTTP_PROXY`/`HTTPS_PROXY` 和 `httpIdleTimeoutMs`。

**本地模型不出现**

1. 确认 llama.cpp 路由器正在运行，`llama serve` 没有带 `--model`（带模型会进入单模型模式）。
2. 运行 `/login llama.cpp` 检查 URL，默认 `http://127.0.0.1:8080`。
3. 使用 `--no-models-autoload` 时，先运行 `/llama` 加载模型。
4. 检查 `LLAMA_BASE_URL` 和可选的 `LLAMA_API_KEY` 是否在启动 Pi 的进程中。

## C.3 终端交互

**`Shift+Enter` 直接发送，不能换行**

终端没有上报扩展键信息。先查官方 Terminal Setup 中对应终端的配置；多数终端可以用 `Ctrl+J` 作为换行替代。

**`Alt+Enter` 不能排队后续消息**

Windows Terminal 会占用部分 Alt 组合；Windows 和 WSL 上默认使用 `Ctrl+Q`。其他终端可能需要在配置中转发 Alt+Enter。运行 `/hotkeys` 确认当前绑定。

**输入法候选框位置不对**

在 WSL、WezTerm 或某些 IDE 终端中，可以让 Pi 使用硬件光标：

```bash
export PI_HARDWARE_CURSOR=1
pi
```

或在设置中把 `showHardwareCursor` 设为 `true`。

**全屏滚动很慢**

iTerm2 在全屏模式下会把滚轮事件交给 Pi。可以调整 iTerm2 的 “Trackpad scrolls fast?” 设置，或改用常规模式：

```bash
pi --tui-mode regular
```

**快捷键无效**

1. 运行 `/hotkeys` 查看实际绑定。
2. 检查 `~/.pi/agent/keybindings.json` 是否覆盖了该动作。
3. 某些 `super` 组合依赖 Kitty 键盘协议，终端不支持时无法工作。
4. 修改后运行 `/reload`。

**颜色不对或图片不显示**

1. 检查终端是否支持真彩色和内联图片。
2. 用 `PI_TRUE_COLOR`、`PI_IMAGE_PROTOCOL` 覆盖自动检测。
3. 在设置中检查 `terminal.images`、`terminal.trueColor`。
4. 主题只影响颜色，不影响图片协议。

## C.4 工具与会话

**工具调用被阻止**

1. 检查是否安装了权限门或安全扩展，例如 `confirm-destructive.ts` 一类的扩展。
2. 运行 `pi -ne` 排除扩展因素。
3. 查看阻止原因；扩展返回的 `reason` 通常会显示在工具结果中。

**文件修改冲突或丢失**

1. 确认自定义工具是否使用 `withFileMutationQueue()`。
2. 同一回合的并行工具调用可能操作同一个文件；逐文件队列是避免覆盖的机制。
3. 用 Git 或快照恢复，并让 Pi 重新读取文件后再修改。

**上下文增长过快，频繁压缩**

1. 用 `/session` 查看用量，确认是哪个模型和会话。
2. 减少一次性读取的大文件；用 `read` 的 `offset`/`limit`，或让 Codemode 先过滤输出。
3. 调整 `compaction.keepRecentTokens` 和 `reserveTokens`，但不要把保留量设得超过模型窗口。
4. 开始新会话，或把不相关的工作分开。
5. 检查 `AGENTS.md` 和技能描述是否过长。

**压缩失败**

压缩需要一次模型调用。提供商不可用、认证失败或请求超限都会导致失败。修复提供商问题后运行 `/compact` 重试。关闭自动压缩不会禁用 `/compact`。

**会话无法继续**

1. 确认启动时没有用 `--no-session`。
2. 确认工作目录与创建会话时相同；会话按目录分组。
3. 检查 `--session-dir`、`PI_CODING_AGENT_SESSION_DIR` 或 `sessionDir` 是否改变了存储位置。
4. 用 `/resume` 打开选择器，或 `pi --session <path|id>` 直接打开。

**`pi --continue` 打开的不是预期的会话**

`--continue` 选择当前目录最近的会话。用 `/resume` 选择其他会话，或用 `--session` 指定路径或 ID。

**工具输出被截断**

不同工具的截断方式不同：

- `bash` 默认保留最后 2000 行或 50KB（先到者为准），并在截断时给出完整输出的临时文件路径。
- `read` 超过行数限制时提示用 `offset`/`limit` 继续读取。
- MCP 的文本工具结果超过 20KB 时去掉中间部分，完整文本保存在结果指出的临时文件中。
- Codemode 脚本拿到的是完整工具结果，可以自己过滤后再输出。

需要完整内容时，用 `read` 分段读取，或让 Codemode 先过滤再返回需要的部分。

## C.5 MCP 与 Codemode

**MCP 服务器连不上**

1. 运行 `pi mcp list`，它会连接所有启用的服务器并报告错误。
2. 检查 `command` 和 `args`：`command` 是一个可执行文件，不是 shell 字符串。
3. 检查 `env`、`headers` 中的变量是否能在 Pi 进程中解析。
4. 确认项目 `.pi/mcp.json` 只在信任项目后读取。
5. 服务只提供 SSE 时无法连接；改用 streamable HTTP 端点。
6. 查看 `~/.pi/agent/mcp.log` 的服务器日志和 stderr 尾部。

**MCP 工具没有出现在模型工具列表**

这通常是暴露方式导致的：

- 默认 `codemode` 暴露不会直接声明工具，需要通过 Codemode 脚本调用；
- `deferred` 暴露需要 `tool_search` 加载；
- 检查 `toolExposure` 是否把工具设为 `hidden`；
- 检查 `--tools` 是否以 `mcp__` 条目过滤掉了工具；
- 运行 `/mcp` 查看实际状态和有效暴露方式。

**Codemode 不可用**

1. 没有 MCP 服务器时，需要手动启用：`"defaultTools": ["+codemode"]` 或 `pi --tools +codemode`。
2. 有 `codemode` 暴露的 MCP 服务器连接时会自动启用。
3. 检查 `codemode.mode` 和 `codemode.inlineBudget` 是否符合预期。
4. 脚本输出超限或调用次数过多会导致失败；把大结果写到文件或分批处理。
5. 多个并发会话受 `PI_CODEMODE_MAX_WORKERS` 限制，默认 2；必要时调高。

**Codemode 脚本找不到工具**

用 `searchTools("关键词")`、`describeTool(name)` 或 `ALL_TOOLS` 查找未声明的工具。MCP 工具名是 `mcp__<server>__<tool>`，脚本中不合法的字符会变成 `_`。

## C.6 包、扩展与分享

**Pi 包安装失败**

1. 检查源格式：`npm:@scope/name@version`、`git:host/repo@ref`、本地路径。
2. 确认 npm 能访问 registry；企业环境检查 `npmCommand`、代理和 `HTTP_PROXY`。
3. 启动时安装失败通常只警告并跳过该包；`pi install` 和 `pi update --extensions` 会以非零状态退出并报告错误。
4. 用 `pi -e <source>` 观察临时加载是否成功。

**包更新后没有变化**

运行 `pi update --extensions` 或 `pi update <source>`。固定版本的 npm 规格、git tag 和 commit 不会自动移动；需要更新规格或使用可移动的 ref。修改后运行 `/reload` 或重启。

**扩展改了但没有生效**

自动发现的扩展需要 `/reload`；用 `-e` 加载的扩展需要重启或重新加载。检查文件是否在 `extensions/` 的发现深度内，或是否在 `package.json` 的 `pi.extensions` 中声明。

**`/share` 失败**

1. 默认在配置了 Radius 认证时使用 Radius，否则使用 GitHub gist。
2. gist 需要 GitHub CLI 已登录：运行 `gh auth status`。
3. 用 `sessionShareDestination` 强制目的地；`radius` 不会回退到 gist。
4. 检查网络和 `PI_SHARE_VIEWER_URL`。

**导出或分享后发现泄露了敏感信息**

立即删除分享产物或 gist，轮换泄露的凭据。会话可能包含提示词、工具参数、命令输出、文件内容和扩展消息。之后在导出前先用 `/session` 和文本搜索审查内容。

## C.7 性能与资源

**响应很慢**

先区分是模型慢还是工具慢：

1. JSON 模式或 `/debug` 查看 `tool_execution_end.durationMs`，确认耗时在工具还是模型。
2. 降低思考等级或换更快的模型。
3. 检查上下文是否过大；长上下文会增加每回合的请求时间。
4. 检查网络、代理和提供商的当前状态。
5. 检查是否有扩展在流式事件上做慢工作。

**内存或磁盘占用高**

- 会话文件会保留全部分支和原始条目；长期使用后可以清理不再需要的会话。
- Codemode 每个 worker 的脚本最多使用 256 MB，默认全局最多 2 个 worker；并发会话多时注意 `PI_CODEMODE_MAX_WORKERS`。
- 大文件读取、图片和工具输出会进入上下文或临时文件；避免一次性读取整个大仓库。
- 扩展如果在 `session_start` 里加载大索引，检查它的内存占用。

**更新失败**

1. 检查安装方式：安装脚本、npm、Nix 的更新命令不同。
2. 权限问题：全局安装目录是否可写。
3. 网络问题：npm registry、代理、离线模式。
4. 锁文件或正在运行的 Pi 进程占用文件；退出后重试。
5. Nix 安装不能用 `pi update` 更新 Pi 本身。

**文档与安装版本不一致**

运行 `pi --version` 确认版本；用 `pi update` 更新；从源码运行时确认 `./pi-test.sh` 使用的是仓库构建而不是全局安装。官方文档对应最新发布，若行为不同，以当前版本的 `pi --help` 和源码为准。

## C.8 诊断信息收集

需要报告问题时，收集：

```bash
pi --version
pi --help
pi auth check --provider <name> --json
pi mcp list
```

以及在 Pi 中运行 `/session`、`/debug` 的输出。`/bug` 可以准备私有报告，可选择是否包含会话记录。分享日志前先审查，`pi-debug.log` 和会话文件可能包含敏感内容。

---

[返回目录](README.md)
