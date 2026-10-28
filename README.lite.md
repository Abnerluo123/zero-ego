# zero-ego (lite)

> [!IMPORTANT]
> **不再需要本项目。** 官方 `ego-browser` CLI 现在可以直接从 DSH 主进程运行，无需 MCP 连接器与 Go 二进制：
> **[Abnerluo123/dsh-ego-browser-cli](https://github.com/Abnerluo123/dsh-ego-browser-cli)**。
> 本说明仅保留给已安装的用户。

本包为 **Ego 原生版**：不经过 MCP 包装，直接配合官方 `ego-browser` skill 使用，功能与官方一致。

**lite 不是精简版**——它与标准包使用同一 Ego 后端，两包都**必须安装 Ego Lite**，不兼容其他浏览器。

## 安装

解压发布包后运行：

- Windows：`.\install.ps1 -ApplyProfile`
- macOS / Linux：`bash ./install.sh --apply`

安装器会备份并更新现有 MCP 配置，不会终止其他会话。

## 使用

1. 阅读已安装的 `ego-browser` skill。
2. 每次任务先 `ego_lite_start`，保存 `spaceId` 并在后续调用中续用。
3. `ego_lite_snapshot` 取 refs，用精确选择器操作，并等待预期结果。
4. 成功后、给出最终答复**之前**调用 `ego_lite_finish`，并确认 `agentControlReleased: true`。
5. 用户接管、暂停或出错时改用 `ego_lite_handoff`，不要 `finish`。

完整说明见 [README](README.md) 与 [Agent 快速指南](AGENTS.md)。
