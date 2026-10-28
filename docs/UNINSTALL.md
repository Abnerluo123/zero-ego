# 卸载 zero-ego（MCP 连接器）

本项目已被 [dsh-ego-browser-cli](https://github.com/Abnerluo123/dsh-ego-browser-cli) 取代。
官方 `ego-browser` CLI 现在由那个插件直接从 DSH 主进程运行，不再需要 MCP 连接器和 Go 二进制。

下面给出完整的卸载步骤。**推荐顺序：先装新插件，再卸旧连接器**，这样中间不会没有浏览器可用。

---

## 一、先确认「装了什么」

用它的安装器装过的话，机器上通常有这三样东西：

| 位置 | 内容 |
|---|---|
| `<DSH profile>/cordis.patch.yml` | 一条 `mcp-ego-lite` 的 MCP 配置 |
| 连接器目录 | `zero-ego-<版本>.exe`（Windows 默认在 `%LOCALAPPDATA%\ego-dsh-link`） |
| `~/.agents/skills/ego-browser/` | 可能被换成了 MCP 版 skill |

先看一眼配置里那条：

```powershell
# Windows
Select-String -Path "$env:USERPROFILE\.dsh\profiles\desktop\cordis.patch.yml" -Pattern "mcp-ego-lite" -Context 2,12
```

```bash
# macOS / Linux
grep -n -A12 'mcp-ego-lite' "$HOME/.dsh/profiles/desktop/cordis.patch.yml"
```

profile 名可能不是 `desktop`（比如 `web`、`tui`），按你自己的来。

---

## 二、删掉 MCP 配置行

在 `cordis.patch.yml` 里找到这一段并**整段删除**（从 `- insert:` 到 `failOnStartupError` 那行）：

```yaml
- insert:
    - id: mcp-ego-lite
      name: "@deepseek-ai/dsh-mcp-client"
      config:
        serverName: ego_lite
        transport: stdio
        command: 'C:/Users/<你>/AppData/Local/ego-dsh-link/zero-ego-1.7.3.exe'
        cwd: 'C:/Users/<你>/AppData/Local/ego-dsh-link'
        env:
          EGO_BROWSER_CLI: 'C:/Users/<你>/.local/share/ego/active_version_dir/ego-browser.exe'
        toolCallTimeoutMs: 360000
        failOnStartupError: false
```

**注意两点：**

1. **删整个 `- insert:` 块**，不要只删里面的 `- id:` 行——留下空的 `insert:` 会让 profile 解析失败。
2. 如果那个 `insert:` 块里还有别的条目，只删 `- id: mcp-ego-lite` 那一条，保留其他。

改完先自己校一下 YAML 能否解析（在 profile 目录里跑）：

```bash
cd "$HOME/.dsh/profiles/desktop"
node -e "console.log(require('js-yaml').load(require('fs').readFileSync('cordis.patch.yml','utf8')).length + ' rows OK')"
```

**改之前先备份**：`cp cordis.patch.yml cordis.patch.yml.bak`。

---

## 三、删除连接器可执行文件

Windows（默认位置）：

```powershell
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\ego-dsh-link"
```

macOS / Linux（安装器把二进制放在你指定的目录，通常是 `~/.local/share/zero-ego` 或当前目录）：

```bash
rm -rf ~/.local/share/zero-ego
```

如果提示文件被占用，说明还有 DSH 进程在跑：**先退出 DSH**，再删，然后重启。

> 删掉后当前会话里已经加载的 `mcp__ego_lite__*` 工具还会存在到重启为止，但那只是内存里的进程，不影响删除。

---

## 四、处理被替换的 skill

本项目的安装器会**用 MCP 版替换官方的 `ego-browser` skill**（把原文另存为 `SKILL.official.md`）。卸掉连接器后应当恢复官方版，否则模型还会按 MCP 的指引找 `ego_lite_*` 工具。

```powershell
# Windows
$d = "$env:USERPROFILE\.agents\skills\ego-browser"
if (Test-Path "$d\SKILL.official.md") {
    Move-Item -Force "$d\SKILL.official.md" "$d\SKILL.md"
    Write-Host "已恢复官方 skill"
}
```

```bash
# macOS / Linux
d="$HOME/.agents/skills/ego-browser"
[ -f "$d/SKILL.official.md" ] && mv -f "$d/SKILL.official.md" "$d/SKILL.md" && echo "已恢复官方 skill"
```

**如果找不到 `SKILL.official.md`**，说明当初没保留原文。这时最省事的做法是装新版 Ego Lite 时让 onboarding 重新铺设 skill，或者直接删掉这个目录，让新插件与官方 CLI 自己工作：

```bash
rm -rf ~/.agents/skills/ego-browser
```

---

## 五、重启 DSH

配置只在启动时加载。重启后：

- `mcp__ego_lite__*` 系列工具消失
- 若已装新插件，`ego_browser` 工具出现，且 shell 里的 `ego-browser` 命令也能直接用了

---

## 六、装新方案（如果还没装）

```bash
git clone https://github.com/Abnerluo123/dsh-ego-browser-cli.git
cd dsh-ego-browser-cli
pwsh -File scripts/install.ps1      # Windows
bash scripts/install.sh             # macOS / Linux
```

然后重启。细节见该仓库 README。

---

## 验证是否卸干净

```bash
# 1. 配置里不应再有 mcp-ego-lite
grep -c 'mcp-ego-lite' "$HOME/.dsh/profiles/desktop/cordis.patch.yml"   # 期望 0

# 2. 连接器目录不应存在
ls "$HOME/AppData/Local/ego-dsh-link" 2>/dev/null || echo "已删除"

# 3. 重启后，工具列表里不应再出现 mcp__ego_lite__*
```

---

## 常见问题

**删完配置后 DSH 起不来**
YAML 被改坏了。用备份还原：`cp cordis.patch.yml.bak cordis.patch.yml`，然后重新按第二步操作，注意整块删除。

**删文件时提示「正在使用」**
还有 DSH / 连接器进程在跑。退出 DSH 再删。

**重启后仍然有 `mcp__ego_lite__*` 工具**
说明配置没删对，或者你改的不是当前正在用的 profile。用启动日志确认加载的是哪个 profile。

**想回退到本项目**
按原 README 的安装步骤重装即可；release 页面仍保留历史版本。
