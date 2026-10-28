# zero-ego

把 Ego Lite 接进 DeepSeek Harness；动作简短返回，页面树按需读取。

> [!IMPORTANT]
> ## 这个连接器不再需要了
>
> 现在有更优雅的做法：**直接让官方 `ego-browser` CLI 从 DSH 主进程运行**，不需要额外的 MCP 连接器，也不需要 Go 二进制。
>
> → **[Abnerluo123/dsh-ego-browser-cli](https://github.com/Abnerluo123/dsh-ego-browser-cli)**
>
> ### 为什么更优雅
>
> | | zero-ego（本项目） | dsh-ego-browser-cli |
> |---|---|---|
> | 组件 | 需要编译/下载 Go 单二进制，并维护 MCP 配置 | 一个 JS 插件，随 profile 加载 |
> | 浏览器 API | 由连接器重新包装成 `ego_lite_*` 工具 | **官方 CLI 原样**，无中间层 |
> | 能力覆盖 | 取决于连接器实现了哪些包装 | 官方全部能力，含 CDP、上传下载、对话框、坐标输入 |
> | 配置 | 安装器改 MCP 配置，需要选包、选版本 | 一条 install 命令，写 profile 的 `package.json` |
> | 升级 | 需要跟着 Ego Lite 与 DSH 双方更新 | 自动跟随 Ego Lite 的 junction，升级无需改配置 |
>
> ### 两者解决的是同一个问题
>
> `ego-browser` 在受限文件策略下连不上 ego daemon。本项目用「另起一个不受限的 MCP 进程」绕开它；新项目发现 DSH 的 MCP 客户端本身就是这么工作的，于是把那同一条路径直接开放给**官方** CLI——少一层包装，能力反而更完整。
>
> ### 迁移
>
> ```bash
> git clone https://github.com/Abnerluo123/dsh-ego-browser-cli.git
> cd dsh-ego-browser-cli
> pwsh -File scripts/install.ps1      # Windows
> bash scripts/install.sh             # macOS / Linux
> ```
>
> 然后重启 DeepSeek Harness。工具名从 `ego_lite_*` 变为单个 `ego_browser`。
>
> ---
>
> 下面是本项目的原有说明，**保留给已经在用的人**，不再主动维护。

[下载最新版](https://github.com/Abnerluo123/zero-ego/releases/latest) · [Agent 快速指南](AGENTS.md) · [详细参考](docs/REFERENCE.md)

## 安装：解压发布包后运行

- Windows：`.\install.ps1 -ApplyProfile`
- macOS / Linux：`bash ./install.sh --apply`

安装器备份并更新现有 MCP 配置，不杀其他会话。

| 包 | 定位 |
|---|---|
| `zero-ego-lite-<版本>.zip` | Ego 原生版，配合官方 skill，功能完整 |
| `zero-ego-<版本>.zip` | 标准发行包，当前使用同一 Ego 后端 |

**lite 不是精简版；两包都需要 Ego Lite，不兼容其他浏览器。**

## 完成任务，也要退出接管

`start → snapshot → 操作与验证 → finish → 等待回执 → 回复用户`

- 成功完成后，必须调用 `ego_lite_finish`；默认关闭 Agent 标签页。
- 仅当结果需要留给用户时，用 `keep:"all"` 或具体标签（如 `keep:"p2"`）；保留页面仍结束任务。
- 确认 `agentControlReleased:true` 才能说已退出接管；false/null 用 `control_status(spaceId)` 检查并如实报告。
- 暂停、报错或请用户操作时用 `handoff`，不要 `finish`；不要清理其他 Agent 的空间。
- MCP 无法知道模型何时结束回复，关标签或停止调用不会自动完成任务。

按需用 `snapshot` 获取 refs，截图用于视觉验证。`run` 使用 ESM；Node/CDP 拥有宿主权限，不是文件沙箱。

[工具速查](docs/AGENT_GUIDE.md) · [能力与权限边界](docs/REFERENCE.md) · [反馈记录](docs/FEEDBACK_STATUS.md) · MIT
