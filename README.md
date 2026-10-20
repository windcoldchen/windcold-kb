# Windcold 知識庫 (Knowledge Base)

基於 **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** 與 **GitHub Pages** 打造的個人技術筆記與知識管理站點。

:earth_americas: **線上網址**：[https://windcoldchen.github.io/windcold-kb/](https://windcoldchen.github.io/windcold-kb/)

---

## :sparkles: 特色

- **Material Design**：現代化精美介面、繁體中文支援、深色 / 淺色主題隨心切換。
- **全域極速搜尋**：即時字詞建議、內容高亮與分享。
- **豐富語法擴充**：支援程式碼分頁、複製按鈕、程式碼註解、Admonition 提示方塊、Mermaid 圖表。
- **自動化發布**：透過 GitHub Actions 每次推送 `main` 分支時自動建置並發布至 GitHub Pages。

---

## :rocket: 本地開發指引

### 1. 建立並啟動 Python 虛擬環境

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2. 安裝相依套件

```bash
pip install -r requirements.txt
```

### 3. 啟動本地即時預覽

```bash
mkdocs serve
```

開啟瀏覽器前往 [http://127.0.0.1:8000](http://127.0.0.1:8000) 即可即時預覽。修改任何 `.md` 或 `mkdocs.yml` 會自動重新載入（Hot Reload）。

### 4. 靜態建置測試

```bash
mkdocs build --strict
```

---

## :gear: GitHub Pages 設定說明

本專案使用 GitHub 官方原生的 **GitHub Actions Pages Artifact** 機制自動部署。

首次發布時，請確認 GitHub 倉庫設定：
1. 前往 **GitHub 倉庫 Settings** -> **Pages**。
2. 在 **Build and deployment** 下方的 **Source**，切換為 **GitHub Actions**。
3. 推送更新到 `main` 分支後，GitHub Actions 將會自動執行建置與部署。

---

## :file_folder: 專案架構

```text
windcold-kb/
├── .github/workflows/
│   └── deploy.yml          # GitHub Actions 自動部署工作流
├── docs/                   # 存放 Markdown 筆記
│   ├── getting-started/    # 入門與語法指引
│   ├── notes/              # 知識筆記分類
│   ├── about.md            # 關於本站
│   └── index.md            # 首頁
├── .gitignore
├── mkdocs.yml              # MkDocs 核心設定
├── README.md               # 專案說明文件
└── requirements.txt        # 依賴清單
```