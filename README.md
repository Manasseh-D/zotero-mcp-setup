# Zotero MCP Setup — 改进版

**MCP 服务器:** [54yyyu/zotero-mcp](https://github.com/54yyyu/zotero-mcp) (MIT, Python, 3500+ ⭐)  
**上游:** [ehawkin/zotero-mcp-setup](https://github.com/ehawkin/zotero-mcp-setup) | 全部 4 个脚本已改进。CLI 接口不变，向上兼容。

---

## TL;DR

1. 下载对应脚本：
   - **Windows：** [install-zotero-mcp.ps1](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.ps1)（右键 → "另存为..."）
   - **Mac：** [install-zotero-mcp.sh](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.sh)
2. 安装 [Zotero 8](https://www.zotero.org/download/) 和 [Claude Desktop](https://claude.ai/download)（如尚未安装）
3. Zotero：**Edit > Settings > Advanced** > 勾选 **"Allow other applications to communicate with Zotero"**
4. 运行脚本：
   - **Windows：** 右键 → "Run with PowerShell"
   - **Mac：** Terminal 中 `bash ~/Downloads/install-zotero-mcp.sh`
5. 脚本引导完成，重启 Claude Desktop

---

## 准备工作

1. **Zotero 8+** — [zotero.org/download](https://www.zotero.org/download/)
2. **Claude Desktop**（桌面版，非网页版）— [claude.ai/download](https://claude.ai/download)
3. **脚本** — [Windows](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.ps1) | [Mac](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.sh)

Zotero 设置：
- **Windows：** Edit > Settings > Advanced > 勾选 **"Allow other applications to communicate with Zotero"**
- **Mac：** Settings > Advanced > 同上

---

## 运行

### Windows

右键 `install-zotero-mcp.ps1` → **"Run with PowerShell"**。若无此选项：

```
powershell -ExecutionPolicy Bypass -File "$HOME\Downloads\install-zotero-mcp.ps1"
```

**不要以管理员身份运行。**

### Mac

```
bash ~/Downloads/install-zotero-mcp.sh
```

### 过程

1. 选择 Default（推荐）或 Advanced
2. 可选输入 Zotero API Key（[zotero.org/settings/keys](https://www.zotero.org/settings/keys)），按 Enter 跳过则仅读取
3. 自动安装依赖、构建搜索索引（~5–15 min）
4. 完成后重启 Claude Desktop

---

## 改进点

三类改进已应用到全部 4 个脚本：

| # | 项 | 类型 | 涉及文件 |
|---|----|------|---------|
| 1 | Zotero 多层路径检测 | 增强 | `.ps1` `.sh` `.py` ×2 |
| 2 | 数据目录检测 (`prefs.js`) | 新增 | `.ps1` `.sh` `.py` ×2 |
| 3 | `ZOTERO_DATA_DIR` 环境变量 | 新增 | `.ps1` `.sh` `.py` (GUI) |

详细改动参见 [CHANGELOG_improvements.md](CHANGELOG_improvements.md)。

---

### 1. 多层路径检测

原版各脚本仅检查固定路径。

```
所有脚本统一策略：
  优先级 1  进程检测    pgrep / Get-Process / tasklist           运行时 100% 命中
  优先级 2  注册表      Uninstall 键 (Windows only)               自定义安装路径
  优先级 3  路径扫描    /Applications / ProgramFiles / LOCALAPPDATA  兜底
```

| 脚本 | 方法 |
|------|------|
| `install-zotero-mcp.ps1` | `Get-Process` → 注册表 → 固定路径 |
| `install-zotero-mcp.sh` | `pgrep -x Zotero` → `/Applications` + `~/Applications` |
| `zotero-mcp-diagnostic.py` | `tasklist`/`pgrep` → WMIC → 注册表 (Win) / 路径扫描 (Mac) |
| `zotero-mcp-installer.py` | `pgrep` → `os.path.isdir` (Mac) / `tasklist` → 固定路径 (Win) |

**数据目录检测（附带）：** prefs.js → `extensions.zotero.dataDir` → 默认 `~/Zotero`

---

### 2. HTTP 连通性检测（仅 `.ps1`）

PS 5.1 `Invoke-WebRequest` 默认行为与 Zotero 内嵌 HTTP 服务器不兼容 → 误报"无法连接"。

→ 原生 `HttpWebRequest`：

```powershell
$req = [System.Net.HttpWebRequest]::Create("http://127.0.0.1:23119/...")
$req.ServicePoint.Expect100Continue = $false
$req.KeepAlive = $false
$req.ProtocolVersion = [System.Net.HttpVersion]::Version11
```

---

### 3. ZOTERO_DATA_DIR 环境变量

所有安装脚本在构建 Claude Desktop 配置时注入：

```powershell  # .ps1
if ($ZoteroDataDir) { $envObj["ZOTERO_DATA_DIR"] = $ZoteroDataDir }
```
```bash     # .sh
if [[ -n "$ZOTERO_DATA_DIR" ]]; then _env_add "ZOTERO_DATA_DIR" "$ZOTERO_DATA_DIR"; fi
```
```python   # .py (GUI)
def _build_zotero_env_vars(self, ..., data_dir=""):
    if data_dir: env_vars["ZOTERO_DATA_DIR"] = data_dir
```

诊断脚本同步新增 `ZOTERO_DATA_DIR` 验证。

---

## 覆盖场景

- Zotero 自定义安装路径 / 便携版 / 用户级 Applications (`~/Applications`)
- 数据目录迁移至其他磁盘
- Win10 旧版 / Server 2016 的 PS 5.1
- Zotero 运行正常但本地 API 连接报错

---

## 兼容性

默认行为不变，仍支持 `-eugene` `--diagnose` 及所有可选开关。诊断脚本和 GUI 安装器接口不变。

---

## 手动更新搜索索引

```powershell
$HOME\.local\bin\zotero-mcp.exe update-db --fulltext          # 增量
$HOME\.local\bin\zotero-mcp.exe update-db --fulltext --force-rebuild  # 重建
```

Mac：
```bash
~/.local/bin/zotero-mcp update-db --fulltext
~/.local/bin/zotero-mcp update-db --fulltext --force-rebuild
```

**不要使用 `sudo` 或 "Run as Administrator"。**

---

## 项目链接

- **Zotero MCP：** [github.com/54yyyu/zotero-mcp](https://github.com/54yyyu/zotero-mcp)
- **安装器上游：** [github.com/ehawkin/zotero-mcp-setup](https://github.com/ehawkin/zotero-mcp-setup)
- ehawkin 团队贡献：11 个新工具、20+ bug 修复、339 项自动化测试（PRs [#165](https://github.com/54yyyu/zotero-mcp/pull/165), [#170](https://github.com/54yyyu/zotero-mcp/pull/170), [#174](https://github.com/54yyyu/zotero-mcp/pull/174)）

---

## 关于

Copyright © 2026 Silver Apps LLC.

本安装器配置 [zotero-mcp](https://github.com/54yyyu/zotero-mcp) by 54yyyu (MIT License)。与 Zotero 项目或 Anthropic 无关。安装器不收集数据，配置修改前自动备份。
