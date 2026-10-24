# LLM-Wiki 原始素材進料區 (Raw Sources Ingestion)

本目錄為 **LLM-Wiki 架構** 的第一層進料區（Tier 1: Raw Sources）。

---

## 1. 目錄定位與功能

- **存放對象**：未經整理的原始資料、技術規格手冊、會議紀錄、網頁截取、架構草稿、PDF 筆記或零散問題排查 Log。
- **免格式包袱**：此目錄受 `.markdownlintignore` 排除，任何草稿或貼入的原始內容皆不會阻礙 CI/CD 建置。
- **編譯目標**：AI Agent（如 Antigravity、Claude Code、Cursor）將此目錄作為輸入來源，定期或依指令將其萃取、蒸餾、結構化並交叉索引至 `docs/notes/` 中。

---

## 2. 建議子目錄結構

```text
sources/
├── drafts/       # 待整理的個人技術隨手筆記或靈感
├── references/   # 外部官方文件、開源專案技術規格或架構截圖
├── meetings/     # 架構討論會議記錄與決策要點
└── logs/         # 線上故障排查過程與原始錯誤訊息
```

---

## 3. LLM 編譯指令範例

當您在此目錄存放新素材後，可直接對 AI Agent 說：

> 「請讀取 `sources/drafts/xxx.md`，依照 `WIKI_SCHEMA.md` 的規範，將其編譯並整合進 `docs/notes/`，同時更新交叉連結與知識圖譜索引。」
