# Zotero MCP Setup — Improved

**MCP server:** [54yyyu/zotero-mcp](https://github.com/54yyyu/zotero-mcp) (MIT, Python, 3,500+ ⭐)  
**Upstream:** [ehawkin/zotero-mcp-setup](https://github.com/ehawkin/zotero-mcp-setup) | All 4 scripts improved. CLI unchanged, fully backwards-compatible.

---

## TL;DR

1. Download:
   - **Windows:** [install-zotero-mcp.ps1](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.ps1) (right-click > "Save link as...")
   - **Mac:** [install-zotero-mcp.sh](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.sh)
2. Install [Zotero 8](https://www.zotero.org/download/) and [Claude Desktop](https://claude.ai/download) if needed
3. Zotero: **Edit > Settings > Advanced** > check **"Allow other applications to communicate with Zotero"**
4. Run:
   - **Windows:** right-click > "Run with PowerShell"
   - **Mac:** Terminal `bash ~/Downloads/install-zotero-mcp.sh`
5. Script guides you through; restart Claude Desktop when done

---

## Prerequisites

1. **Zotero 8+** — [zotero.org/download](https://www.zotero.org/download/)
2. **Claude Desktop** (desktop app, not web) — [claude.ai/download](https://claude.ai/download)
3. **Script** — [Windows](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.ps1) | [Mac](https://raw.githubusercontent.com/ehawkin/zotero-mcp-setup/main/install-zotero-mcp.sh)

Zotero setting:
- **Windows:** Edit > Settings > Advanced > check **"Allow other applications to communicate with Zotero"**
- **Mac:** Settings > Advanced > same

---

## How to run

### Windows

Right-click `install-zotero-mcp.ps1` > **"Run with PowerShell"**. If unavailable:

```
powershell -ExecutionPolicy Bypass -File "$HOME\Downloads\install-zotero-mcp.ps1"
```

**Do not run as Administrator.**

### Mac

```
bash ~/Downloads/install-zotero-mcp.sh
```

### Process

1. Pick Default (recommended) or Advanced
2. Optional: enter Zotero API Key ([zotero.org/settings/keys](https://www.zotero.org/settings/keys)); press Enter to skip (read-only)
3. Dependencies installed, search index built (~5–15 min)
4. Restart Claude Desktop

---

## Improvements

Three improvements applied across all 4 scripts:

| # | Item | Type | Files |
|---|------|------|-------|
| 1 | Multi-tier Zotero path detection | Enhanced | `.ps1` `.sh` `.py` ×2 |
| 2 | Data directory detection (`prefs.js`) | Added | `.ps1` `.sh` `.py` ×2 |
| 3 | `ZOTERO_DATA_DIR` env var | Added | `.ps1` `.sh` `.py` (GUI) |

Full details: [CHANGELOG_improvements.md](CHANGELOG_improvements.md).

---

### 1. Multi-tier path detection

Upstream checked fixed paths only.

```
Unified strategy:
  Tier 1  Process      pgrep / Get-Process / tasklist           100% when running
  Tier 2  Registry     Uninstall key (Windows only)              Custom install paths
  Tier 3  Path scan    /Applications / ProgramFiles / LOCALAPPDATA  Fallback
```

| Script | Method |
|--------|--------|
| `install-zotero-mcp.ps1` | `Get-Process` → Registry → Fixed paths |
| `install-zotero-mcp.sh` | `pgrep -x Zotero` → `/Applications` + `~/Applications` |
| `zotero-mcp-diagnostic.py` | `tasklist`/`pgrep` → WMIC → Registry (Win) / Path scan (Mac) |
| `zotero-mcp-installer.py` | `pgrep` → `os.path.isdir` (Mac) / `tasklist` → Fixed paths (Win) |

**Data directory (bonus):** prefs.js → `extensions.zotero.dataDir` → default `~/Zotero`

---

### 2. HTTP connectivity fix (`.ps1` only)

PS 5.1 `Invoke-WebRequest` defaults conflict with Zotero's embedded HTTP server → false "unable to connect".

→ raw `HttpWebRequest`:

```powershell
$req = [System.Net.HttpWebRequest]::Create("http://127.0.0.1:23119/...")
$req.ServicePoint.Expect100Continue = $false
$req.KeepAlive = $false
$req.ProtocolVersion = [System.Net.HttpVersion]::Version11
```

---

### 3. ZOTERO_DATA_DIR env var

All install scripts inject detected data directory into Claude Desktop config:

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

Diagnostic script also validates `ZOTERO_DATA_DIR` presence.

---

## Use cases

- Custom Zotero install / portable / user-level `~/Applications`
- Data directory migrated to another disk
- Older Win10 / Server 2016 with PowerShell 5.1
- Zotero running but local API falsely reported as unreachable

---

## Compatibility

Default behavior unchanged; `-eugene` `--diagnose` and all optional switches still supported. Diagnostic and GUI interfaces unchanged.

---

## Manual search index update

```powershell
$HOME\.local\bin\zotero-mcp.exe update-db --fulltext          # Incremental
$HOME\.local\bin\zotero-mcp.exe update-db --fulltext --force-rebuild  # Full rebuild
```

Mac:
```bash
~/.local/bin/zotero-mcp update-db --fulltext
~/.local/bin/zotero-mcp update-db --fulltext --force-rebuild
```

**Do not use `sudo` or "Run as Administrator".**

---

## Project links

- **Zotero MCP:** [github.com/54yyyu/zotero-mcp](https://github.com/54yyyu/zotero-mcp)
- **Upstream installer:** [github.com/ehawkin/zotero-mcp-setup](https://github.com/ehawkin/zotero-mcp-setup)
- ehawkin team contributed 11 new tools, 20+ bug fixes, 339 automated tests (PRs [#165](https://github.com/54yyyu/zotero-mcp/pull/165), [#170](https://github.com/54yyyu/zotero-mcp/pull/170), [#174](https://github.com/54yyyu/zotero-mcp/pull/174))

---

## About

Copyright © 2026 Silver Apps LLC.

Installer configures [zotero-mcp](https://github.com/54yyyu/zotero-mcp) by 54yyyu (MIT License). Not affiliated with Zotero project or Anthropic. No data collected. Config backed up automatically before changes.
