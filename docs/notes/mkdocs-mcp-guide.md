---
icon: material/robot
description: 說明如何使用 mkdocs-mcp 外掛將知識庫轉為 Model Context Protocol (MCP) 伺服器，讓 Claude Desktop、Cursor 與 Antigravity 等 AI 助理能即時檢索與理解本地技術文件。
tags:
  - MCP
  - AI
  - MkDocs
  - 工具
  - 外掛
---

# 使用 mkdocs-mcp 整合 AI 助理指南 :material-robot:

> 藉由 **Model Context Protocol (MCP)** 開放標準，運用 `mkdocs-mcp` 外掛將此知識庫轉化為可由 AI 代理人（Claude Desktop、Antigravity IDE、Cursor 等）即時讀取與問答的結構化資源庫。

---

## 1. 什麼是 Model Context Protocol (MCP)？

**Model Context Protocol (MCP)** 是由 Anthropic 所主導發起的開放標準協定，旨在為大型語言模型 (LLM) 及其客戶端建立一套標準化接口，以安全、即時且可控的方式存取外部資料來源（Resources）、工具（Tools）與 Prompt 模板。

傳統讓 AI 閱讀專案文件的痛點：

- **知識過期**：通用模型無法獲取本地剛更新的私有筆記與架構決策。
- **重型 RAG 維護成本高**：需要自建向量資料庫（Vector DB）、切分 Chunk、維護 Embedding 與檢索管線。
- **MCP 的優勢**：直接將 MkDocs 專案以標準 MCP Resource 格式（`docs://{filepath}`）掛載至 AI 助理，AI 客戶端可隨時調用最新、結構完整的原始 Markdown 檔案。

### 架構與運作流程圖

```mermaid
flowchart LR
    subgraph Repo["本地知識庫專案"]
        Docs[Markdown 筆記 docs/*.md] --> Plugin[mkdocs-mcp 外掛]
        Plugin --> FastMCP[FastMCP Server<br/>(stdio / sse)]
    end

    subgraph Client["AI 客戶端 (MCP Clients)"]
        FastMCP <-->|MCP 協定 docs://*| Claude[Claude Desktop]
        FastMCP <-->|MCP 協定 docs://*| Agy[Antigravity IDE]
        FastMCP <-->|MCP 協定 docs://*| Cursor[Cursor / Cline]
    end

    subgraph Query["使用者互動"]
        Claude --> Answer[問答與架構諮詢<br/>產出程式碼 / 檢查清單]
        Agy --> Answer
        Cursor --> Answer
    end
```

---

## 2. 外掛配置與核心參數 (`mkdocs.yml`)

本知識庫已在 `mkdocs.yml` 啟用 `mcp` 外掛：

```yaml
plugins:
  - mcp:
      naming_style: src_file   # 資源命名方式：src_file, dst_url, title
      prefer_markdown: true    # 優先提供原始 Markdown (若為 false 則為渲染後 HTML 轉純文字)
      combine_all_pages: false # 是否彙整為單一 all 資源
      combine_by_folder: false # 是否依目錄層級自動合併子頁面資源
```

### 參數說明

| 參數 | 預設值 | 可選值 | 說明 |
| :--- | :--- | :--- | :--- |
| `naming_style` | `src_file` | `src_file`, `dst_url`, `title` | 控制暴露給 MCP 資源時的標題格式。推薦 `src_file`（以相對路徑命名，例如 `notes/workflow.md`，便於 AI 精確對應檔案）。 |
| `prefer_markdown` | `true` | `true`, `false` | 為 `true` 時直接傳送原始 Markdown 原始碼；為 `false` 時會將 Material 渲染後的 HTML 轉回純文字。 |
| `combine_all_pages` | `false` | `true`, `false` | 額外產生一個 URI 為 `docs://.` 的聚合資源，將全站所有頁面合併在單一內容中。 |
| `combine_by_folder` | `false` | `true`, `false` | 依資料夾層級產生聚合資源（例如 `docs://getting-started`、`docs://notes`）。 |

---

## 3. 本地啟動 MCP 服務

`mkdocs-mcp` 提供了獨立 CLI 指令 `mkdocs-mcp build`，會在背後執行 MkDocs 建置並啟動 FastMCP 伺服器：

### 啟動步驟

```bash
# 1. 啟用專案虛擬環境
source .venv/bin/activate

# 2. 啟動 MCP 伺服器 (預設為 stdio 模式，適合與 本地 AI 客戶端連線)
mkdocs-mcp build

# （選用）指定 SSE 模式 (適合跨行程或網路通訊)
mkdocs-mcp build --mcp-transport sse
```

> [!NOTE]
> `mkdocs-mcp build` 支援傳遞大部分標準 MkDocs 建置選項，例如指定專案路徑：
>
> ```bash
> mkdocs-mcp build --mkdocs-project-path /Users/luke.chen/Documents/github/windcold-kb
> ```

---

## 4. 各大 AI 客戶端整合範例

### A. Claude Desktop

編輯 Claude Desktop 設定檔：

- **macOS**：`~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**：`%APPDATA%\Claude\claude_desktop_config.json`

在 `mcpServers` 區塊加入以下內容（請替換為您本地的實際絕對路徑）：

```json
{
  "mcpServers": {
    "windcold-kb": {
      "command": "/Users/luke.chen/Documents/github/windcold-kb/.venv/bin/mkdocs-mcp",
      "args": ["build"],
      "cwd": "/Users/luke.chen/Documents/github/windcold-kb"
    }
  }
}
```

設定存檔後，完全重啟 Claude Desktop。在對話視窗右下方看到工具圖示（Hammer / Plug）出現 `windcold-kb` 即代表連線成功。

### B. Cursor

1. 開啟 Cursor，進入 **Settings** ➔ **Features** ➔ **MCP**。
2. 點選 **+ Add New MCP Server**：
   - **Name**：`windcold-kb`
   - **Type**：`command`
   - **Command**：`/Users/luke.chen/Documents/github/windcold-kb/.venv/bin/mkdocs-mcp build`
3. 點選儲存，狀態顯示綠燈即可在 Composer 或 Chat 中直接查詢。

### C. Antigravity IDE / Cline / Roo Code

在專案或全域的 MCP 設定檔（例如 `mcp_config.json`）中設定：

```json
{
  "mcpServers": {
    "windcold-kb": {
      "command": "/Users/luke.chen/Documents/github/windcold-kb/.venv/bin/mkdocs-mcp",
      "args": [
        "build",
        "--mkdocs-project-path",
        "/Users/luke.chen/Documents/github/windcold-kb"
      ]
    }
  }
}
```

---

## 5. 實際使用範例 (Prompt Examples)

當 MCP 伺服器成功連線後，AI 助理可直接調取 `docs://` 資源回答專案相關問題：

### 範例一：依據知識庫部署方案產出檢查清單

> **User Prompt**：
> 「請讀取知識庫中的 `notes/azure-static-web-apps.md`，幫我整理一份在生產環境正式上線 Azure SWA 前的安全性與 CI/CD 檢查清單。」
>
> **AI 回應**：AI 會直接讀取該文檔內容，精準列出包含 `skip_app_build: true`、`AZURE_STATIC_WEB_APPS_API_TOKEN`、`staticwebapp.config.json` 安全標頭與 Entra ID 驗證等具體實踐。

### 範例二：查詢全站技術棧與外掛

> **User Prompt**：
> 「根據知識庫設定文件，目前此專案啟用了哪些 MkDocs 外掛？各自的用途是什麼？」
>
> **AI 回應**：AI 會調閱 `getting-started/configuration.md`，輸出包含 `git-latest-changes`、`tags`、`mcp` 等 13 項外掛之詳細用途。

---

## 6. 常見問題與除錯要點 (Troubleshooting)

### Q1: 安裝 `mkdocs-mcp` 時提示 `Requires-Python >=3.12`？

**原因**：`mkdocs-mcp`（包含底層依賴的 `mcp` 核心套件）需運行於 Python 3.12 或更高版本。
**解法**：確認本地虛擬環境與 CI/CD 執行環境的 Python 版本設定為 `>= 3.12`。本專案之 CI/CD（`ci.yml` 與 `deploy.yml`）已全數配置為 Python 3.12。

### Q2: 修改 Markdown 文件後，AI 客戶端內容未更新？

**原因**：MCP stdio 伺服器在 `mkdocs-mcp build` 啟動時會快取當前建置的頁面資源清單。
**解法**：若有新增或大幅修改文件，重啟 AI 客戶端或重新執行 `mkdocs-mcp build` 即可重新抓取最新頁面。
