# Antigravity Agent Guidelines

本文件定義 AI Agent 在此儲存庫（`windcold-kb`）協同作業時的權限、規範與標準工作流。

---

## 1. 代理權限與自動化原則 (Permissions & Workflow)

1. **全面放行 Git 與 GitHub 指令 (Always Allow All Git & GitHub Commands)**：
   - 本儲存庫對 Agent 完全授權自動執行所有 `git` 指令，包括但不限於：
     - `git status`, `git diff`, `git log`
     - `git branch`, `git checkout`, `git switch`
     - `git add`, `git commit`, `git push`, `git pull`, `git fetch`
     - `git stash`, `git merge`, `git rebase`
   - 授權執行所有 `GitHub CLI (gh)` 指令，包括：
     - `gh pr create`, `gh pr checks`, `gh pr view`, `gh pr merge`
     - `gh run list`, `gh run view`, `gh auth token`
   - 以上所有指令一律視為預設許可（Pre-approved / Always Allowed），Agent 應直接自動執行，無須向使用者發起確認或詢問。
2. **允許自動合併 Pull Request**：
   - 本儲存庫授權 Agent 在完成需求並確認 CI 綠燈後，自動執行 Squash & Merge：

     ```bash
     gh pr merge <PR_NUMBER> --squash --delete-branch
     ```

3. **標準發布流程**：
   - 開立特徵/修復分支（例如 `feat/...` 或 `fix/...`）。
   - 執行本機完整驗證（詳見第 2 節）。
   - Commit & Push 至遠端分支。
   - 使用 `gh pr create` 發起 Pull Request。
   - 監控 GitHub Actions CI 狀態（`gh pr checks <PR_NUMBER>`）。
   - CI 全部通過後執行 Squash & Merge，並同步本地 `main` 分支。

---

## 2. 代碼檢驗與建置標準 (Verification Standards)

在任何 commit 或開立 PR 前，**必須**在本機完成以下驗證：

1. **Markdown 格式檢驗**：

   ```bash
   npm run lint:md
   ```

   - 需確認 0 errors。若有格式空格問題可使用 `npm run lint:md:fix` 自動修復。

2. **MkDocs 嚴格建置檢驗**：

   ```bash
   source .venv/bin/activate
   DISABLE_MKDOCS_2_WARNING="true" MKDOCS_GIT_COMMITTERS_APIKEY=$(gh auth token) mkdocs build --strict
   ```

   - 必須在嚴格模式（`--strict`）下完全通過且無任何警告（0 warnings）。

---

## 3. Mermaid 圖表語法規範 (Mermaid Diagram Standards)

MkDocs Material 整合之 Mermaid 解析器對未引號包裹的符號較為敏感：

1. **一律以雙引號包裹文字**：
   - 節點文字（Node Text）與連線標籤（Edge Label）若包含括號 `(...)`、中括號、斜線 `/`、換行標籤 `<br/>` 等，**必須使用雙引號包裹**：

     ```mermaid
     %% 正確範例
     flowchart TD
         A["工程師提交代碼 / 開立 MR"] --> B[("Self-Hosted GitLab")]
         B --> C["GitLab Runner (Docker Executor)"]
         C -->|"驗證通過 (Pass)"| D["發布至邊緣 CDN"]
     ```

   - 避免直接書寫未引號的括號（如 `Runner[GitLab Runner (Docker Executor)]`），否則會觸發 `Parse error: got 'PS'` 導致整個圖形無法渲染。

---

## 4. 提交訊息規範 (Commit Conventions)

遵循 Conventional Commits 規範：

- `docs: 新增 ...`
- `fix(docs): 修復 ...`
- `refactor(mkdocs): 重構 ...`
- `feat: 新增 ...`
- `chore: 更新 ...`
