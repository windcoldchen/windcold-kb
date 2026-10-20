# 知識庫設定與 GitHub Pages 部署指南

本頁面詳細介紹如何維護此知識庫，以及設定 GitHub Pages 自動化發布的完整步驟。

---

## 1. 專案目錄結構

知識庫的標準檔案配置如下：

```text
windcold-kb/
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions 自動部署工作流
├── docs/                     # 存放所有 Markdown 筆記的主目錄
│   ├── index.md              # 知識庫首頁
│   ├── getting-started/      # 入門指引
│   │   ├── configuration.md
│   │   └── syntax-cheatsheet.md
│   ├── notes/                # 分類筆記
│   │   ├── example.md
│   │   └── workflow.md
│   └── about.md              # 關於頁面
├── .gitignore
├── mkdocs.yml                # MkDocs 核心設定檔
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
    - 選擇 **Read and write permissions**（或確認允許 GitHub Actions 提交與部署）。
3. 點擊 **Save**。

---

## 3. GitHub Actions 工作流程解析

我們在 `.github/workflows/deploy.yml` 定義了自動化建置流程：

```yaml
name: Deploy MkDocs to GitHub Pages

on:
  push:
    branches:
      - main  # 當推送到 main 分支時觸發

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
      - name: 檢出儲存庫
        uses: actions/checkout@v4

      - name: 設定 Python 環境
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      - name: 安裝依賴套件
        run: pip install -r requirements.txt

      - name: 建置靜態網站
        run: mkdocs build

      - name: 上傳 Pages 產出物
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./site

      - name: 部署至 GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

每次你修改文章並執行：

```bash
git add .
git commit -m "docs: 新增筆記"
git push origin main
```

GitHub Actions 就會自動觸發並在約 1~2 分鐘內完成建置與上線！

---

## 4. 本地常用指令

| 指令 | 說明 |
| :--- | :--- |
| `mkdocs serve` | 啟動本地開發伺服器（預設網址 `http://127.0.0.1:8000`），修改文件即刻更新 |
| `mkdocs build` | 建置純靜態網站至 `./site` 目錄 |
| `mkdocs build --strict` | 嚴格模式建置（若有內部連結斷裂或警告將直接拋錯，推薦部署前自我檢查） |
| `mkdocs --help` | 查看所有支援指令 |
