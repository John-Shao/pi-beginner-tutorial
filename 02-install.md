# 第 2 章 安装、登录与完成第一个任务

[← 上一章](01-what-is-pi.md) · [目录](README.md)

> **本章你将学会**：在 macOS、Linux 或 Windows 上安装 Pi；连接一个模型提供商；在项目目录中启动 Pi；用 `@` 引用文件完成一个真实任务；更新和卸载 Pi。
>
> **对应官方文档**：[Quickstart](https://pi.dev/docs/latest/quickstart) · [Use Pi in the Terminal](https://pi.dev/docs/latest/usage) · [Windows Setup](https://pi.dev/docs/latest/windows)

## 2.1 安装前确认

Pi 是一个 Node.js 程序，要求 Node.js 22.19 或更高版本。官方安装脚本会在需要时安装 Node，npm 安装方式则要求你先准备好符合要求的 Node。

在终端中检查：

```bash
node --version
```

示例输出：

```text
v24.18.0
```

还需要一个可用的模型访问方式：订阅账号、API Key、云平台凭据或本地模型。第 4 章会逐一展开，本章先用最简单的 `/login` 流程。

## 2.2 安装

从安装到完成第一个任务的顺序：

```mermaid
flowchart LR
    A["安装 Pi"] --> B["进入项目目录"]
    B --> C["启动 pi"]
    C --> D["/login 连接模型"]
    D --> E["/model 选择模型"]
    E --> F["发送任务"]
```

macOS 和 Linux 使用官方安装脚本：

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

Windows 原生环境使用 PowerShell：

```powershell
powershell -c "irm https://pi.dev/install.ps1 | iex"
```

安装脚本会固定所有依赖版本，之后用 `pi update` 更新。也可以直接从 npm 安装：

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
```

npm 方式不会固定间接依赖，升级时可能出现依赖漂移。`--ignore-scripts` 是有意的：Pi 的正常安装不需要依赖的生命周期脚本。

四种方式的选择：

| 方式 | 依赖是否固定 | 更新方式 | 适合 |
|---|---|---|---|
| 安装脚本 | 是，使用仓库生成的安装锁 | `pi update` | 大多数个人用户 |
| npm 全局安装 | 否，只固定直接依赖 | `pi update` 或重装 | 已有 Node 工具链、需要统一管理 |
| Nix flake | 由 Nix 锁定 | `nix profile upgrade pi` | 使用 Nix 管理开发环境 |
| 源码 checkout | 是，由 `package-lock.json` 固定 | `git pull` 后重装并重建 | 阅读、修改或调试 Pi |

安装脚本和 npm 安装的 Pi 都可以用 `pi update` 更新；Nix 安装由 Nix 管理，`pi update` 只更新包和模型目录。

使用 Nix 的 macOS 或 Linux 用户可以从 flake 安装：

```bash
nix profile add github:earendil-works/pi/stable
```

旧版 Nix 使用 `nix profile install`。Nix 安装从源码构建，更新要用 `nix profile upgrade pi`，`pi update` 无法更新它。

安装完成后验证：

```bash
pi --version
```

示例输出：

```text
1.4.0
```

### 从源码安装并运行

如果你要阅读、修改或调试 Pi，建议从仓库源码运行，而不是先全局安装。源码方式需要 Git、Node.js 22.19 或更高版本，以及 npm。

```bash
git clone https://github.com/earendil-works/pi.git
cd pi
npm install --ignore-scripts
npm run build
```

仓库 README 列出的常用源码命令如下：

| 命令 | 作用 |
|---|---|
| `npm install --ignore-scripts` | 安装所有依赖，但不运行生命周期脚本 |
| `npm run build` | 刷新模型数据，然后构建所有包 |
| `npm run build:offline` | 使用已有模型数据离线重建 |
| `npm run check` | 运行 lint、格式化和类型检查 |
| `./test.sh` | 运行测试；没有 API Key 时跳过依赖真实模型的测试 |
| `./pi-test.sh` | 从源码运行 Pi；可在任意目录执行 |

从源码启动 Pi：

```bash
./pi-test.sh --version
./pi-test.sh
```

在 Windows PowerShell 中使用：

```powershell
.\pi-test.ps1 --version
.\pi-test.ps1
```

`pi-test.sh` 和 `pi-test.ps1` 会直接运行 `packages/coding-agent/src/cli.ts`，并把参数传给 Pi。它们不会把 `pi` 命令安装到 `PATH`；适合开发验证，不适合当作日常全局命令。需要长期使用时，可改用安装脚本、npm 或 Nix。

更新源码安装：

```bash
git pull
npm install --ignore-scripts
npm run build
```

如果要生成独立可执行文件，可以使用仓库脚本：

```bash
./scripts/build-binaries.sh --offline-model-data --platform linux-x64 --out "$PWD/out"
```

`--platform` 可选 `darwin-arm64`、`darwin-x64`、`linux-x64`、`linux-arm64`、`windows-x64`、`windows-arm64`。GitHub Release 还提供带 `SHA256SUMS` 校验的源码归档；从归档构建时，先解压并进入对应版本目录，再运行同一脚本。

## 2.3 选择工作目录并启动

Pi 以“工作目录”为中心组织一切：它从这里发现项目配置、上下文文件和资源，也按这个目录给会话分组。工作目录还会成为内置工具的默认根目录：`read` 的相对路径、`bash` 的当前目录都以它为准。因此先进入目标项目，再启动，不要在用户主目录或仓库的父目录中启动后再用绝对路径操作：

```bash
cd /path/to/project
pi
```

如果目录中存在需要信任的项目资源（例如 `.pi/settings.json`、`.pi/extensions/`），Pi 会先询问是否信任该项目。第 6 章会解释哪些资源受信任控制，以及如何保存决定。第一次可以先用一个自己的练习目录，避免在重要仓库中误操作。

启动后界面分为三部分：

- **对话记录（transcript）**：显示你的输入、模型回复、工具调用、工具结果和错误。工具调用可以折叠或展开。
- **编辑器（editor）**：输入提示词和斜杠命令。编辑器边框的颜色或样式反映当前思考等级。
- **状态栏（footer）**：显示当前目录、会话、模型、上下文用量，以及累计 token 和费用。

<p align="center">
  <img src="assets/interactive-mode.png" alt="Pi 的终端界面：启动头部、对话记录、编辑器和状态栏" width="720">
</p>

> 官方文档中的界面截图，版本可能早于本教程基线。Pi 的界面会随版本调整，配色、字段和快捷键提示以你启动后的实际界面为准。

启动头部还会列出本次加载的指令文件与资源。第一次启动时，花一分钟检查这些信息：

- 工作目录是否正确，是否是仓库根目录；
- 加载了哪些 `AGENTS.md`/`CLAUDE.md`，有没有意外的父目录文件；
- 加载了哪些扩展、技能和 Pi 包；
- 当前模型和思考等级是否符合预期。

如果头部显示了你没预期的资源，先退出检查配置和信任状态，不要在这个状态下让 Pi 修改代码。若只想看干净界面，可以用 `/settings` 或 `quietStartup` 设置隐藏部分启动信息。

## 2.4 连接模型

在 Pi 的编辑器中输入：

```text
/login
```

选择提供商，然后按提示完成订阅登录或粘贴 API Key。凭据保存在 `~/.pi/agent/auth.json`。登录完成后，输入：

```text
/model
```

选择一个可用模型。模型选择器只显示当前有可用凭据的提供商下的模型。选定后可以用 `Ctrl+S` 把它保存为新会话的默认模型。

如果不想让 Pi 保存凭据，可以改用环境变量。例如：

```bash
export ANTHROPIC_API_KEY=sk-ant-...
pi
```

环境变量必须在启动 Pi 的进程中存在。CI 和一次性任务适合这种方式。完整提供商列表见第 4 章和官方 Providers 页面。

如果启动后没有任何可用模型，Pi 会提示你登录或配置提供商。先完成 `/login`，或在启动前设置对应环境变量，再发送任务；不要在没有可用模型的情况下反复重试。

关于凭据的三点提醒：

- `auth.json` 可能同时包含 API Key 和 OAuth token，不要提交到仓库，也不要放进共享目录。
- `/logout` 只删除本机保存的凭据，不会吊销提供商侧的 token，也不会清除环境变量或 `models.json` 中的认证。
- 在远程机器上，OAuth 回调可能打不开本地浏览器；把最终重定向 URL 或授权码粘贴回 Pi 即可完成登录。

> 源码位置：`packages/coding-agent/src/core/auth-storage.ts`（凭据读写与解析）、`packages/coding-agent/src/core/model-registry.ts`（模型与提供商注册）。

## 2.5 完成第一个任务

好的任务描述通常包含四要素：目标、范围、约束、验收。例如：

```text
目标：为 src/utils/date.ts 增加一个把日期格式化为 ISO 周字符串的函数。
范围：只修改 src/utils/date.ts 和对应的测试文件。
约束：不要改公开 API；沿用现有错误处理风格；运行 npm test -- date。
验收：新测试通过，现有测试不回归。
```

范围小、结果可检查的任务更容易验证，也更容易在出错时回退。先给 Pi 一个这样的任务。在编辑器中输入：

```text
列出这个仓库的顶层目录，说明每个目录的用途，并告诉我项目用什么命令运行测试。
```

Pi 会调用 `ls` 或 `bash` 查看目录，可能读取 `README.md`、`package.json` 或 `Makefile`，然后给出总结。你会看到每次工具调用及其结果，而不是只看到最终答案。

第二个任务练习文件引用。输入 `@` 会打开文件搜索；输入文件名的一部分并按 `Tab` 补全。例如：

```text
总结 @README.md 的安装步骤，并把缺失的前置条件补充成一个 checklist 写到 setup-checklist.md。
```

第三个任务练习修改代码：

```text
找出 src 下所有未处理的 TODO，按文件分组列出；然后只修复其中涉及注释拼写错误的部分，修改后运行测试。
```

发送前避免把密钥、token 或整段生产日志直接粘进提示词；需要提供大文件时先脱敏，或用 `@` 引用一个裁剪过的副本。会话会被保存和导出，粘贴进去的内容应按“已写入磁盘”处理。

任务完成后，按下面的顺序验收：

1. 看对话记录中最后几条工具调用，确认 Pi 实际读了哪些文件、跑了哪些命令。
2. 用 `git diff` 或编辑器检查改动，不要只看模型总结。
3. 如果 Pi 声称运行了测试，确认测试命令、退出码和输出；必要时自己再跑一遍。
4. 对不满意的改动直接继续对话，例如“把第二个文件的改动回退，只保留第一个”。
5. 如果模型没有完成，指出具体缺口，而不是重复原任务：“你没有修改 `src/auth.ts`，请只处理这个文件”。

Pi 不要求每次工具调用都确认，因此**审查改动是使用者的责任**。重要仓库建议先提交或建分支，再让 Pi 修改。发现它开始执行明显错误或危险的操作时，按 `Esc` 中止；已经排队的消息会回到编辑器。

## 2.6 退出、继续与更新

退出 Pi 使用 `/quit`，或连续按两次 `Ctrl+C`，或在空编辑器中按 `Ctrl+D`。会话默认自动保存。下次从同一目录运行：

```bash
pi --continue
```

即可继续最近的会话。运行 `pi --resume` 打开会话选择器，挑选其他会话。

> 源码位置：`packages/coding-agent/src/cli/session-picker.ts`（会话选择器）、`packages/coding-agent/src/core/session-manager.ts`（会话打开与继续）。

更新 Pi：

```bash
pi update
```

更新 Pi 和所有已安装的 Pi 包：

```bash
pi update --all
```

只更新包用 `pi update --extensions`；只刷新模型目录用 `pi update --models`。

更新后建议检查：

1. `pi --version` 是否变化；
2. 启动头部是否报告扩展或包加载失败；
3. `/changelog` 中是否有影响你工作流的变更；
4. 项目资源是否仍按预期加载。

Pi 的包和模型目录更新互不影响。`pi update` 只更新 Pi 本身，不会自动更新已安装的第三方包；需要时用 `pi update --all`。

## 2.7 卸载

npm 安装：

```bash
npm uninstall -g @earendil-works/pi-coding-agent
```

安装脚本：重新运行安装脚本并选择 **Uninstall Pi**。

Nix：

```bash
nix profile remove pi
```

三种方式都不会删除 `~/.pi/agent/` 下的配置、凭据、会话或已安装的 Pi 包。需要彻底清理时，确认没有要保留的会话后再手动删除该目录。

## 2.8 Windows 与其他终端

Windows 原生环境使用 PowerShell 安装脚本。多数功能与 macOS、Linux 一致，但快捷键有差异：

- 追加消息用 `Ctrl+Q`（对应其他平台的 `Alt+Enter`）。
- 取回排队消息用 `Alt+Q`（对应 `Alt+Up`）。
- 粘贴图片用 `Alt+V`（对应 `Ctrl+V`）。
- 原生 Windows 不支持 Unix 作业控制，`Ctrl+Z` 默认不绑定挂起。

Windows Terminal 会占用部分 Alt 快捷键；如果某个组合无效，先查官方 Terminal Setup 页面，再用 `/hotkeys` 查看当前生效的绑定。WSL 环境按 Linux 处理。

Windows 上还有几处差异：

- 原生 Windows 默认用 Git Bash 执行内置 `bash` 工具和 `!` 命令，因此通常需要安装 Git for Windows；Pi 会按 `shellPath`、常见 Git Bash 路径、`PATH` 的顺序查找 Bash。
- 可选的 `powershell` 工具让模型调用 PowerShell（优先 `pwsh.exe`，回退到 Windows PowerShell）；`!` 和 `!!` 命令仍使用 Bash。
- 路径分隔符和大小写敏感性不同于 Unix；在提示词中写路径时尽量使用工作目录内的相对路径。
- 如果安装后命令找不到，检查安装器写入的目录是否在当前用户的 `PATH` 中，并重新打开终端。

## 2.9 小结

- Pi 要求 Node.js 22.19 及以上；可用安装脚本、npm、Nix 或源码 checkout 安装。源码方式用 `npm install --ignore-scripts`、`npm run build` 和 `./pi-test.sh` 运行。
- 工作目录决定项目配置、资源发现和会话分组，启动前先 `cd` 到目标项目。
- 用 `/login` 连接提供商，用 `/model` 选择模型；CI 中可用环境变量代替存储凭据。
- 第一个任务应范围小、结果可验证；审查改动是使用者的责任。
- `pi --continue` 继续当前目录最近的会话；`pi update` 更新 Pi，`pi update --all` 同时更新已安装的包。
- 卸载不会删除 `~/.pi/agent/` 中的数据；首次使用应从小任务开始，按“检查工具调用、检查 diff、检查测试”的顺序验收。

## 练习

1. 安装 Pi 并运行 `pi --version`。验收：输出与当前安装版本一致。
2. 连接一个提供商，用 `/model` 选择模型并发送“你当前使用哪个模型”。验收：回复中的模型与状态栏显示一致。
3. 在一个练习目录中让 Pi 创建 `hello.txt`，内容为当前日期，然后读取并核对。验收：文件存在且内容正确；你能在对话记录中看到 `write` 和 `read` 的工具调用。
4. 退出后运行 `pi --continue`，继续上一个会话并让它修改 `hello.txt`。验收：修改发生在上一个会话的上下文中，`/session` 显示的会话 ID 未变。
5. 用 `pi --help` 找到 `--no-session` 并解释它的作用。验收：能说明该次运行不会持久化会话，退出后无法继续。
6. 克隆 Pi 仓库，从源码安装并运行 `./pi-test.sh --version`。验收：命令输出 Pi 版本；Windows 下可用 `.\pi-test.ps1 --version` 完成同样验证。

## 延伸阅读

- [官方文档：Quickstart](https://pi.dev/docs/latest/quickstart)
- [官方文档：Use Pi in the Terminal](https://pi.dev/docs/latest/usage)
- [官方文档：Windows Setup](https://pi.dev/docs/latest/windows)
- 源码：仓库根 `README.md` 的 Development 与 Building standalone binaries 章节
- 源码：`packages/coding-agent/src/cli/args.ts`（命令行参数）、`packages/coding-agent/src/core/auth-storage.ts`、`packages/coding-agent/src/modes/interactive/interactive-mode.ts`

---

[目录](README.md) · [下一章 →](03-terminal.md)
