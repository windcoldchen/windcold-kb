# 歡迎來到 Windcold 知識庫 :material-book-open-page-variant:

> 基於 **LLM-Wiki + Material for MkDocs** 架構打造的個人技術與架構數位花園。

本知識庫採用 **Andrej Karpathy** 所倡導的「持續編譯式知識庫（Compiling Stateful Wiki）」理念，結合 AI 代理人持續進行知識蒸餾、交叉鏈接與版本維護，並透過 **GitHub Actions** 進行嚴格語法檢核與自動化發布至 **GitHub Pages**。

---

## :material-compass-outline: 快速導覽

<div class="grid cards" markdown>

-   :material-brain:{ .lg .middle } __LLM-Wiki 智慧架構__

    ---

    深入了解本專案的三層知識生命週期（Raw Ingestion ➔ LLM Compiler ➔ MkDocs / MCP / llms.txt）。

    [:octicons-arrow-right-24: 前往架構指南](notes/llm-wiki-architecture.md)

-   :material-rocket-launch:{ .lg .middle } __快速上手__

    ---

    了解知識庫的建置流程、設定方式以及如何透過 GitHub Pages 發布。

    [:octicons-arrow-right-24: 前往設定指南](getting-started/configuration.md)

-   :material-format-paint:{ .lg .middle } __語法與元件示範__

    ---

    探索 Material for MkDocs 的豐富功能：摺疊面板、程式碼分頁、提示框與流程圖。

    [:octicons-arrow-right-24: 查看語法指南](getting-started/syntax-cheatsheet.md)

-   :material-notebook-edit-outline:{ .lg .middle } __技術筆記__

    ---

    涵蓋後端開發、系統架構、DevOps、可觀測性與 SRE 工程最佳實踐。

    [:octicons-arrow-right-24: 探索技術筆記](notes/example.md)

-   :material-history:{ .lg .middle } __最新異動__

    ---

    瀏覽全站文件的 Git 最新提交修訂歷程與變更紀錄。

    [:octicons-arrow-right-24: 查看異動紀錄](changelog.md)

-   :material-information-outline:{ .lg .middle } __關於知識庫__

    ---

    關於此知識庫的緣起、維護原則與聯絡方式。

    [:octicons-arrow-right-24: 了解更多](about.md)

</div>

---

## :material-history: 最近文件異動

以下為知識庫近期修訂之文件（由 `git-latest-changes` 自動彙整）：

{{ latest_changes }}

---

## :material-lightning-bolt: 核心特色與發布流程

```mermaid
flowchart LR
    A["原始素材 (sources/)"] --> B["AI Agent 編譯器"]
    B --> C["結構化 Markdown (docs/)"]
    C --> D{"CI 語法檢查<br/>mkdocs build --strict"}
    D -->|"通過"| E["GitHub Pages 自動發布"]
    D -->|"通過"| F["llms.txt & MCP 線上服務"]
```

- **全域即時搜尋**：支援關鍵字即時提示、語意高亮與分享連結。
- **深色 / 淺色模式切換**：隨系統設定自動切換，亦可手動一鍵點選。
- **程式碼語法支援**：行號顯示、行標記、複製按鈕以及代碼註解。
- **品質守護 CI/CD**：開立 PR 自動執行嚴格語法建置檢查，通過後自動部署上線。

---

## :material-source-branch: 本地即時預覽

在本地終端機中執行以下指令即可啟動即時預覽伺服器：

=== "使用虛擬環境 (Virtualenv)"

    ```bash
    # 啟動現有虛擬環境
    source .venv/bin/activate

    # 啟動本地開發伺服器
    mkdocs serve
    ```

=== "新環境建立"

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt
    mkdocs serve
    ```

伺服器啟動後，開啟瀏覽器前往 [http://127.0.0.1:8000](http://127.0.0.1:8000) 即可同步查看編輯效果（支援 Hot Reloading）。
