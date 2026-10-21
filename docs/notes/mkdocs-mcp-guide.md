---
icon: material/robot
description: 說明如何使用 mkdocs-mcp 外掛將知識庫轉為 Model Context Protocol (MCP) 伺服器，並比較「本地開發即時模式」與「遠端線上 GitHub Pages 免環境直連模式」之實作方式。
tags:
  - MCP
  - AI
  - MkDocs
  - 工具
  - 外掛
---

# 使用 mkdocs-mcp 整合 AI 助理指南 :material-robot:

> 藉由 **Model Context Protocol (MCP)** 開放標準，將此知識庫轉化為可由 AI 代理人（Claude Desktop、Antigravity IDE、Cursor 等）即時讀取與問答的結構化資源庫。本篇涵蓋**本地開發者即時模式**與**遠端線上 GitHub Pages 免環境直連模式**兩種情境。

---

## 1. 什麼是 Model Context Protocol (MCP)？

**Model Context Protocol (MCP)** 是由 Anthropic 所主導發起的開放標準協定，旨在為大型語言模型 (LLM) 及其客戶端建立一套標準化接口，以安全、即時且可控的方式存取外部資料來源（Resources）、工具（Tools）與 Prompt 模板。

傳統讓 AI 閱讀專案文件的痛點：

- **知識過期**：通用模型無法獲取本地剛更新的私有筆記與架構決策。
- **重型 RAG 維護成本高**：需要自建向量資料庫（Vector DB）、切分 Chunk、維護 Embedding 與檢索管線。
- **MCP 的優勢**：直接將 MkDocs 專案以標準 MCP Resource 格式（`docs://{filepath}`）或遠端 Search Tool 掛載至 AI 助理，AI 客戶端可隨時調用最新、結構完整的技術文件。

---

## 2. 兩種 MCP 整合情境全解析

在實際應用中，通常會根據**使用者身分**與**網路環境**區分為兩種情境：

```mermaid
flowchart TD
    subgraph ModeA["情境一：本地開發者即時模式 (mkdocs-mcp)"]
        LocalDocs[本地 docs/*.md 原始碼] --> PythonMCP[Python mkdocs-mcp CLI]
        PythonMCP -->|stdio 通訊| LocalAI[本地 AI 助理<br/>(Cursor / Claude / Antigravity)]
        LocalAI --> ResultA[可查詢未 commit 的本機最新草稿]
    end

    subgraph ModeB["情境二：遠端線上 GitHub Pages 直連模式 (@serverless-dna/mkdocs-mcp)"]
        RemoteSite[GitHub Pages 網站<br/>https://windcoldchen.github.io/windcold-kb/] --> SearchIndex[search/search_index.json]
        SearchIndex --> NodeMCP[Node npx @serverless-dna/mkdocs-mcp]
        NodeMCP -->|HTTP 抓取索引| RemoteAI[任何人的 AI 助理<br/>(零依賴、免 clone)]
        RemoteAI --> ResultB[免安裝 Python，直接查詢線上最新發布文檔]
    end
```

### 兩種模式功能與維度對比表

| 比較維度 | 情境一：本地開發即時模式 (`mkdocs-mcp`) | 情境二：遠端線上 GitHub Pages 直連 (`@serverless-dna/mkdocs-mcp`) |
| :--- | :--- | :--- |
| **技術棧** | Python (`pip install mkdocs-mcp`) | Node.js (`npx @serverless-dna/mkdocs-mcp`) |
| **資料來源** | 本地原始碼目錄 (`docs/*.md`) | 線上 GitHub Pages 靜態網站 (`search_index.json`) |
| **適用對象** | **知識庫作者 / 開發者** | **讀者 / 團隊成員 / 任何終端使用者** |
| **即時性** | **極高**（本機修改、尚未 commit 的草稿也能即時問答） | **依賴部署**（必須已 Push 並由 CI/CD 部署至 GitHub Pages） |
| **本地環境依賴** | 電腦需具備 Python >= 3.12 與本地儲存庫檔案 | **零依賴**（任何能執行 `npx` 的電腦皆可一鍵使用） |
| **協定角色** | 提供 MCP **Resource**（`docs://...` 虛擬檔案系統） | 提供 MCP **Tool**（`search_documentation` 搜尋檢索工具） |

---

## 3. 情境一：本地開發者即時模式 (`mkdocs-mcp`)

適合在本地撰寫筆記、除錯或規劃架構時使用。

### A. 外掛配置 (`mkdocs.yml`)

本知識庫已在 `mkdocs.yml` 啟用 `mcp` 外掛：

```yaml
plugins:
  - mcp:
      naming_style: src_file   # 資源命名方式：src_file, dst_url, title
      prefer_markdown: true    # 優先提供原始 Markdown (若為 false 則為渲染後 HTML 轉純文字)
      combine_all_pages: false # 是否彙整為單一 all 資源
      combine_by_folder: false # 是否依目錄層級自動合併子頁面資源
```

### B. 本地啟動指令

`mkdocs-mcp` 提供了獨立 CLI 指令 `mkdocs-mcp build`，會在背後執行 MkDocs 建置並啟動 FastMCP 伺服器：

```bash
# 1. 啟用專案虛擬環境
source .venv/bin/activate

# 2. 啟動 MCP 伺服器 (預設為 stdio 模式，適合與本地 AI 客戶端連線)
mkdocs-mcp build

# （選用）指定 SSE 模式 (適合跨行程或網路通訊)
mkdocs-mcp build --mcp-transport sse
```

### C. 本地 AI 客戶端配置範例

在客戶端設定檔（如 `claude_desktop_config.json` 或 Cursor MCP 設定）填入本地虛擬環境指令：

```json
{
  "mcpServers": {
    "windcold-kb-local": {
      "command": "/Users/luke.chen/Documents/github/windcold-kb/.venv/bin/mkdocs-mcp",
      "args": ["build"],
      "cwd": "/Users/luke.chen/Documents/github/windcold-kb"
    }
  }
}
```

---

## 4. 情境二：遠端線上 GitHub Pages 免環境直連模式

若團隊成員或其他使用者**沒有本地專案程式碼、也沒有安裝 Python**，但希望讓 AI 助理直接閱讀您已發布到 GitHub Pages 的最新技術庫，可使用此方案。

### A. 運作原理

任何透過 Material for MkDocs 部署至 GitHub Pages 的站點，皆會自動生成全站搜尋索引：

```text
https://windcoldchen.github.io/windcold-kb/search/search_index.json
```

`@serverless-dna/mkdocs-mcp` 會直接在背景透過 HTTP 下載該索引，並封裝成標準的 MCP 工具，讓 AI 能即時發動關鍵字搜尋與頁面摘要。

### B. Claude Desktop 設定方式

直接編輯 Claude Desktop 設定檔（`claude_desktop_config.json`），加入公開網站網址：

```json
{
  "mcpServers": {
    "windcold-kb-online": {
      "command": "npx",
      "args": [
        "-y",
        "@serverless-dna/mkdocs-mcp",
        "https://windcoldchen.github.io/windcold-kb/",
        "Windcold 技術知識庫 (線上版)"
      ]
    }
  }
}
```

> [!TIP]
> 儲存後完全重啟 Claude Desktop，對話框右下角的工具清單即會出現 `windcold-kb-online`，具備 `search_documentation` 與 `get_page` 兩項工具，無須配置任何本地檔案！

### C. Cursor 設定方式

1. 開啟 Cursor，進入 **Settings** ➔ **Features** ➔ **MCP**。
2. 點選 **+ Add New MCP Server**：
   - **Name**：`windcold-kb-online`
   - **Type**：`command`
   - **Command**：`npx -y @serverless-dna/mkdocs-mcp https://windcoldchen.github.io/windcold-kb/ "Windcold 技術知識庫"`
3. 點選儲存，狀態顯示綠燈即可在 Composer 或 Chat 中直接查詢線上文件。

---

## 5. 實際使用範例 (Prompt Examples)

當 MCP 伺服器成功連線後，AI 助理可直接調用知識庫內容：

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

**原因**：Python 版 `mkdocs-mcp`（包含底層依賴的 `mcp` 核心套件）需運行於 Python 3.12 或更高版本。
**解法**：確認本地虛擬環境與 CI/CD 執行環境的 Python 版本設定為 `>= 3.12`。本專案之 CI/CD（`ci.yml` 與 `deploy.yml`）已全數配置為 Python 3.12。

### Q2: 為什麼 GitHub Pages 無法直接運行 Python 版 `mkdocs-mcp`？

**原因**：GitHub Pages 是純靜態伺服器（Static Hosting），只能託管 HTML/CSS/JS 靜態檔案，沒有常駐行程（Daemon/Process）能接收來自外界的 MCP 連線。
**解法**：若要存取線上 GitHub Pages，請採用本文第 4 節的 `@serverless-dna/mkdocs-mcp` 模式，直接透過 HTTP 解析線上的 `search_index.json`。

### Q3: 本地修改 Markdown 文件後，AI 客戶端內容未更新？

**原因**：本地 MCP stdio 伺服器在 `mkdocs-mcp build` 啟動時會快取當前建置的頁面資源清單。
**解法**：若有新增或大幅修改文件，重啟 AI 客戶端或重新執行 `mkdocs-mcp build` 即可重新抓取最新頁面。
