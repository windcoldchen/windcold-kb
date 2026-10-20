# 知識庫設定與 GitHub Pages 部署指南

本頁面詳細介紹如何維護此知識庫，包含 GitHub Actions CI/CD 自動化設定、Markdown Lint 檢查與分支保護規範。

---

## 1. 專案目錄結構

知識庫的標準檔案配置如下：

```text
windcold-kb/
├── .github/
│   └── workflows/
│       ├── ci.yml            # PR 語法與 Markdown Lint 檢查 (CI)
│       └── deploy.yml        # main 分支 Pages 自動發布 (CD)
├── docs/                     # 存放所有 Markdown 筆記的主目錄
│   ├── index.md              # 知識庫首頁
│   ├── getting-started/      # 入門指引
│   │   ├── configuration.md  # 知識庫設定與部署說明
│   │   └── syntax-cheatsheet.md # 語法元件示範
│   ├── notes/                # 分類筆記
│   │   ├── example.md        # 筆記範本
│   │   └── workflow.md       # 日常工作流
│   └── about.md              # 關於頁面
├── .gitignore
├── .markdownlint.yaml        # Markdownlint 規則設定檔
├── .markdownlint-cli2.yaml   # Markdownlint-cli2 執行與忽略路徑設定
├── mkdocs.yml                # MkDocs 核心設定檔
├── package.json              # Lint 指令設定
├── README.md
└── requirements.txt          # Python 依賴清單
```

---

## 2. GitHub Pages 設定步驟

為了讓 GitHub Actions 能自動部署知識庫，請在 GitHub Repository 進行以下一次性設定：

### 步驟 A：開啟 GitHub Pages 的 GitHub Actions 來源

1. 開啟 GitHub 倉庫頁面：[windcoldchen/windcold-kb](https://github.com/windcoldchen/windcold-kb)
2. 點擊頂部的 **Settings**（設定）標籤頁。
3. 在左側側邊欄中找到 **Pages**（位於 Code and automation 區塊下）。
4. 在 **Build and deployment** > **Source** 下拉選單中：
    - 將預設的 `Deploy from a branch` 切換為 **`GitHub Actions`**。

> [!IMPORTANT]
> 務必確認 Source 選擇的是 **GitHub Actions**。如此一來，工作流建置完成後會透過官方的 Pages Artifact 直接託管，不需要額外建立 `gh-pages` 分支，也不會有權限或衝突問題。

### 步驟 B：設定 Actions 權限（若尚未啟用）

1. 在專案的 **Settings** > **Actions** > **General**。
2. 滾動至 **Workflow permissions**：
    - 選擇 **Read and write permissions**。
3. 點擊 **Save**。

---

## 3. 分支保護機制 (Branch Ruleset)

為了確保主分支上的筆記內容品質，本專案在 `main` 啟用了 GitHub Ruleset：

1. **禁止直接 Push 到 `main`**：所有變更必須透過開立分支與發起 Pull Request (PR)。
2. **強制要求狀態檢查 (Required Status Checks)**：
   - 建立 PR 後，GitHub 會自動並行觸發：
     - `Markdown Lint Check`（格式與語法規範）
     - `MkDocs Build Check`（建置與內部連結有效性）
   - 兩項檢查成功通過（綠燈 :white_check_mark:）後，GitHub 才會開放點擊 **Merge pull request**。
3. **禁止刪除或強制覆蓋 `main`** (`deletion` 與 `non-fast-forward` 保護)。

---

## 4. GitHub Actions 工作流程解析

專案採用職責分離的雙工作流程設計：

### A. CI 語法與樣式檢查：`.github/workflows/ci.yml`

當有任何 Pull Request 指向 `main` 時自動並行觸發兩個檢查任務：

```yaml
name: CI - Syntax & Lint Check

on:
  pull_request:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  markdown-lint:
    name: Markdown Lint Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: 執行 Markdown Lint 語法檢查
        uses: DavidAnson/markdownlint-cli2-action@v19
        with:
          globs: "**/*.md"

  build-check:
    name: MkDocs Build Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"
      - run: pip install -r requirements.txt
      - name: 檢查 MkDocs 語法與站點建置 (嚴格模式)
        run: mkdocs build --strict
```

### B. CD 部署上線：`.github/workflows/deploy.yml`

僅在變更真正合併至 `main` 分支時觸發部署：

```yaml
name: Deploy MkDocs to GitHub Pages

on:
  push:
    branches:
      - main  # 當 PR 合併至 main 時觸發

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"
      - run: pip install -r requirements.txt
      - run: mkdocs build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./site
      - id: deployment
        uses: actions/deploy-pages@v4
```

---

## 5. 本地常用指令

| 指令 | 說明 |
| :--- | :--- |
| `mkdocs serve` | 啟動本地開發伺服器（預設網址 `http://127.0.0.1:8000`），修改文件即刻更新 |
| `npx markdownlint-cli2 "**/*.md"` | 執行 Markdown Lint 格式檢查 |
| `npx markdownlint-cli2 --fix "**/*.md"` | 自動修復 Markdown 格式與空格問題 |
| `mkdocs build --strict` | 嚴格模式建置（若有內部連結斷裂或警告將直接拋錯，推薦開 PR 前自我檢查） |
