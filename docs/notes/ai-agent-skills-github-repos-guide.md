---
icon: material/puzzle-star
description: 全面盤點與深度解析開源 AI Agent Skills (SKILL.md) 生態系與核心 GitHub 知識庫，涵蓋漸進式載入原理、官方核心庫（Anthropic、Vercel、Cloudflare、Supabase、Hugging Face、Trail of Bits）、Superpowers 工程紀律框架、DevOps/SRE 實戰庫與 npx skills 跨平台管理實踐。
tags:
  - Agent Skills
  - SKILL.md
  - AI
  - Claude Code
  - Antigravity
  - MCP
  - DevOps
  - SRE
  - GitHub
---

# 開源 AI Agent Skills (`SKILL.md`) 核心 GitHub 知識庫全景與實戰指南 :material-puzzle-star:

> 隨著 AI 編程與維運代理（如 **Google Antigravity**、**Claude Code**、**Cursor**、**OpenAI Codex**、**Windsurf**）成為工程團隊的核心生產力工具，如何將「領域專家經驗、標準作業程序（SOP）與安全防護基線」模組化地封裝並跨工具復用，催生了開放的 **Agent Skills (`SKILL.md`)** 標準。本文系統性整理全球最具代表性的 AI Agent Skills GitHub 儲存庫，從底層漸進式載入原理、官方大廠庫、工程方法論、DevOps/SRE 專項庫到企業安全治理，提供一站式的選型與落地實戰指南。

---

## 1. 什麼是 AI Agent Skills (`SKILL.md`)？核心問題與運作原理

### 1.1 傳統 Prompt 與全域規則檔面臨的瓶頸

在 Agent Skills 標準出現之前，工程師通常將所有規範塞入單一全域規則檔（如 `AGENTS.md`、`CLAUDE.md` 或 `.cursorrules`），在大型專案中會迅速遭遇三大瓶頸：

1. **Context Window 嚴重污染與算力浪費**：無論當前任務是「撰寫前端元件」還是「排查 K8s 網路故障」，數千行的全域規則都會在每一輪對話中無條件載入，不僅消耗大量 Token，更會稀釋模型的注意力（Attention Dilution）。
2. **缺乏可攜性與跨團隊復用機制**：散落在對話紀錄或單一儲存庫中的 Prompt 無法像 npm 或 Helm Chart 一樣被版本化管理、跨專案安裝與社群共享。
3. **缺少腳本與參考資源的封裝能力**：複雜的工程任務（如靜態資安掃描、資料庫 Schema 遷移、雲原生故障診斷）除了文字指令外，還需要配套的輔助腳本（`scripts/`）、標準範本（`examples/`）與領域字典（`references/`）。

### 1.2 `Rules` vs `Agent Skills` vs `MCP Server` 定位比較

在現代 AI 代理架構中，這三者並非互相取代，而是各司其職的互補鐵三角：

| 比較維度 | 全域/階層規則 (`AGENTS.md` / `GEMINI.md`) | AI 技能包 (`skills/<name>/SKILL.md`) | 模型上下文協定 (`MCP Server`) |
| :--- | :--- | :--- | :--- |
| **核心職責** | **「什麼絕對不能做 / 專案通用守則」** | **「特定任務該如何分步執行的 SOP」** | **「手腳與感官：連接外部系統的 API 工具」** |
| **載入時機** | **常駐載入 (Always-on)** 或按檔案路徑觸發 | **漸進式按需載入 (Progressive Disclosure)** | 工具 Schema 常駐或延遲載入，調用時執行 |
| **Token 消耗** | 隨檔案膨脹持續消耗每輪對話額度 | 啟動僅耗費 **~50–100 Tokens**（僅讀摘要） | 視註冊的 Tool Schema 數量而定 |
| **封裝內容** | 單一 Markdown 規則檔 | `SKILL.md` + `scripts/` + `references/` + `examples/` | 執行期伺服器程序（stdio / SSE / HTTP） |
| **典型場景** | 專案 Commit 規範、禁止直接操作生產環境 | K8s 跨層根因分析流程、TDD 開發工作流 | 查詢 VictoriaMetrics、檢索 [mkdocs-mcp](mkdocs-mcp-guide.md) |

### 1.3 三階段漸進式揭露 (Progressive Disclosure) 運作機制

Agent Skills 標準（`agentskills.io`）的核心設計精髓在於**「漸進式揭露」**，確保即使安裝了上百個技能包，也不會拖慢 AI 代理的回應速度：

```mermaid
sequenceDiagram
    autonumber
    actor User as "工程師 (User)"
    participant Agent as "AI Agent (Antigravity / Claude Code / Cursor)"
    participant Index as "技能索引層 (YAML Frontmatter)"
    participant Body as "SKILL.md 核心指令本體"
    participant Res as "附屬資源 (scripts/ & references/)"

    Note over Agent,Index: "階段一：Discovery (啟動探索，每個技能僅 ~50-100 Tokens)"
    Agent->>Index: "掃描技能目錄，僅載入各 SKILL.md 頂部的 name 與 description"

    User->>Agent: "「請幫我審查這次 Terraform 與 K8s YAML MR 的資安與成本合規性」"

    Note over Agent,Body: "階段二：Activation (動態匹配觸發，< 5k Tokens)"
    Agent->>Body: "語意匹配命中 secops-mr-reviewer，讀取完整 SKILL.md 步驟"

    Note over Agent,Res: "階段三：Execution & Resources (按需載入腳本與參考規範)"
    Agent->>Res: "按步驟讀取 references/secops-baseline.md 並執行 scripts/check_iam.py"
    Agent-->>User: "輸出結構化審查報告與合規修改建議"
```

### 1.4 標準技能包目錄結構與 `SKILL.md` 解剖

一個符合工業標準的 Skill 是一個獨立資料夾，內部以 `SKILL.md` 為入口：

```text
my-skill-name/
├── SKILL.md                # [必填] 包含 YAML Frontmatter (name, description) 與執行步驟
├── scripts/                # [選填] 供 Agent 直接呼叫的驗證腳本 (Python / Bash)
├── references/             # [選填] 深度參考文檔，僅在 SKILL.md 指示時按需讀取
└── examples/               # [選填] 黃金配置範例或輸出樣板 (Golden Templates)
```

---

## 2. 全球頂級 AI Agent Skills GitHub 儲存庫分類全覽

目前開源社群已圍繞 `SKILL.md` 形成龐大的知識庫生態。以下將最具價值的 GitHub 儲存庫分為**五大核心類別**：

```mermaid
flowchart TB
    subgraph Ecosystem["開源 AI Agent Skills (SKILL.md) 生態系全景"]
        direction TB
        subgraph Cat1["1. 標準制定與官方核心庫"]
            R1["anthropics/skills<br/>(官方標準與文檔/MCP 技能)"]
            R2["vercel-labs/skills & agent-skills<br/>(npx skills CLI 與前端效能規範)"]
        end

        subgraph Cat2["2. 軟體工程紀律與工作流"]
            R3["obra/superpowers<br/>(TDD / 規格先行 / 系統化除錯)"]
        end

        subgraph Cat3["3. DevOps / SRE / 雲原生與資安"]
            R4["bregman-arie/devops-sre-skills<br/>(安全唯讀 Runbooks & K8s/AWS)"]
            R5["arjunprabhulal/devops-skills<br/>(15 大領域 / 88+ 實戰指南)"]
            R6["BagelHole/DevOps-Security-Agent-Skills<br/>(多雲基建與容器安全加固)"]
            R7["trailofbits/skills<br/>(頂級資安審計 / CodeQL / CI 防護)"]
        end

        subgraph Cat4["4. 雲端大廠與資料平台官方庫"]
            R8["cloudflare/skills<br/>(Workers / Durable Objects / D1)"]
            R9["supabase/agent-skills<br/>(Postgres 調優與 RLS 安全)"]
            R10["huggingface/skills<br/>(LLM 微調 / Hub 操作 / 評測)"]
            R11["stripe/ai<br/>(支付串接與合規基線)"]
        end

        subgraph Cat5["5. 社群聚合索引與市集"]
            R12["VoltAgent/awesome-agent-skills<br/>(1000+ 團隊驗證技能目錄)"]
            R13["ComposioHQ/awesome-claude-skills<br/>(生產工作流與 SaaS 串接)"]
            R14["heilcheng/awesome-agent-skills<br/>(跨 IDE 工具通用索引)"]
        end
    end
```

---

### 2.1 類別一：標準制定與官方核心庫 (Official & Foundational)

#### 1. [`anthropics/skills`](https://github.com/anthropics/skills)

- **維護背景**：Anthropic 官方維護的 Agent Skills 旗艦儲存庫與規範起點。
- **核心亮點**：
  - **辦公與技術文檔自動化（Document Skills）**：內建生產級的 `docx`、`pdf`、`pptx`、`xlsx` 技能包，包含完整的 Python 處理腳本與排版規範。
  - **元技能（Meta-Skills）**：提供 `skill-creator`（協助開發者從零自動生成並驗證符合規範的新 `SKILL.md`）與 `mcp-builder`（指導 AI 按照最佳實踐開發 FastMCP / TypeScript MCP Server）。
  - **前端與測試規範**：包含 `frontend-design` 與 `webapp-testing`（基於 Playwright 的自動化驗證流程）。
- **適用對象**：所有使用 AI Agent 的開發者，特別適合用來學習官方如何撰寫高觸發率、低 Token 消耗的黃金 `SKILL.md`。

#### 2. [`vercel-labs/skills`](https://github.com/vercel-labs/skills) 與 [`vercel-labs/agent-skills`](https://github.com/vercel-labs/agent-skills)

- **維護背景**：Vercel 官方實驗室開源的跨平台技能套件管理器（對應索引站 `skills.sh`）與前端/雲端最佳實踐庫。
- **核心亮點**：
  - **`npx skills` CLI 引擎**：提供類似 `npm` 的體驗，支援一鍵將任何 GitHub 儲存庫中的 `SKILL.md` 安裝並同步映射至 Claude Code、Cursor、Antigravity、Codex、Windsurf 等數十種 AI 代理目錄。
  - **React / Next.js 效能鐵律**：`vercel-labs/agent-skills` 內含 40+ 條經過生產驗證的規則（如消除 Request Waterfalls、Server Actions 資安防護、Bundle 體積優化）。

---

### 2.2 類別二：軟體工程紀律與多代理工作流 (Engineering Methodology)

#### 1. [`obra/superpowers`](https://github.com/obra/superpowers)

- **維護背景**：由資深架構師 Jesse Vincent 創立，是目前 GitHub 上星數成長最快、影響力最大的「**AI 軟體工程紀律框架**」。
- **解決痛點**：通用 AI 代理在面對中大型任務時，常犯下「未釐清需求就盲目寫 code」、「跳過測試直接宣稱完成」、「遇到 Bug 隨機猜測修改」等毛病。
- **核心技能鏈（強制分階段工作流）**：
  1. **`brainstorming`**：在寫任何一行程式碼前，強制透過蘇格拉底式提問釐清邊界條件與架構設計。
  2. **`writing-plans`**：將需求拆解為具備明確驗證標準的微任務（2–5 分鐘顆粒度）計畫書。
  3. **`test-driven-development` (TDD)**：嚴格執行 `RED（先寫失敗測試）➔ GREEN（最小實作通過）➔ REFACTOR（重構）` 鐵律，若 AI 先寫實作再補測試會被強制要求刪除重來。
  4. **`subagent-driven-development` & `dispatching-parallel-agents`**：由主代理擔任架構師，派發乾淨 Context 的子代理平行實作，並在每步結束後執行兩階段審查（規格合規審查 + 程式碼品質審查）。
  5. **`systematic-debugging`**：四階段科學除錯法（根因調查 ➔ 模式分析 ➔ 假設驗證 ➔ 修復實作），徹底杜絕盲目試錯。
  6. **`using-git-worktrees`**：利用 Git Worktree 建立隔離開發環境，確保多分支並行互不干擾。

---

### 2.3 類別三：DevOps、SRE、雲原生與資安維運專項庫 (DevOps, SRE & SecOps)

這類儲存庫與本專案的 [全能型 IS / SRE 團隊 AI 工作流藍圖](is_sre_team_ai_workflow_blueprint.md) 高度契合，可直接作為團隊建立內部維運技能包的基石：

#### 1. [`bregman-arie/devops-sre-skills`](https://github.com/bregman-arie/devops-sre-skills)

- **維護背景**：由知名 DevOps 面試指南（`devops-exercises`）作者 Arie Bregman 打造的 SRE 專屬技能庫。
- **核心亮點**：
  - **Safe-by-Default（預設唯讀安全）**：所有排障 Runbook 預設僅執行唯讀診斷指令（Read-only），涉及變更操作時強制附上「影響半徑預估（Blast Radius）」與「回滾步驟（Rollback Guidance）」。
  - **涵蓋領域**：Kubernetes 故障排查、AWS 架構巡檢、Terraform 狀態修復、ArgoCD 同步異常處理、Linux 效能瓶頸分析與 Incident Response 標準流程。

#### 2. [`arjunprabhulal/devops-skills`](https://github.com/arjunprabhulal/devops-skills)

- **維護背景**：跨平台（Harness-agnostic）的雲原生與平台工程實戰技能大全。
- **核心亮點**：
  - 收錄 **15 大技術領域、88+ 份結構化 `SKILL.md`**，涵蓋 Kubernetes、Reliability & SRE、Observability（Prometheus / OpenTelemetry / Grafana）、CI/CD、GitOps、FinOps 與服務網格（Service Mesh）。
  - 每份技能皆採用「編號步驟 + 可執行驗證指令（Verifiable Checks）」格式，極度適合直接轉換為團隊內部的 L1/L2 自動化診斷劇本。

#### 3. [`BagelHole/DevOps-Security-Agent-Skills`](https://github.com/BagelHole/DevOps-Security-Agent-Skills)

- **核心亮點**：專注於 **DevSecOps 與多雲安全基線**，包含 80+ 份涵蓋 AWS / Azure / GCP IAM 最小權限審查、Kubernetes Pod 安全加固（SecurityContext / NetworkPolicy）、Terraform 靜態合規檢查與容器逃逸防護的技能包。

#### 4. [`trailofbits/skills`](https://github.com/trailofbits/skills)

- **維護背景**：由全球頂級網路安全研究機構 **Trail of Bits** 官方開源的資安審計技能庫。
- **核心亮點**：
  - **`audit-context-building`**：指導 AI 在尋找漏洞前，先進行逐行、跨函式的深度上下文與控制流建模，大幅降低誤報率。
  - **`agentic-actions-auditor`**：專為 CI/CD 設計，自動審查 GitHub Actions / GitLab CI 工作流中是否存在 Prompt Injection、不安全腳本插值與權限過大風險。
  - **`static-analysis` & `differential-review`**：整合 CodeQL、Semgrep 與 SARIF 報告解析，針對 PR/MR 差異進行高精準度的安全變更審查。

#### 5. [`chaterm/terminal-skills`](https://github.com/chaterm/terminal-skills) 與 [`nik-kale/sre-skills`](https://github.com/nik-kale/sre-skills)

- **核心亮點**：聚焦於終端機第一線操作（`kubectl`、Helm Chart 除錯、Docker 網路排查、Linux 核心參數調優）以及生產環境上線就緒度檢查表（Production Readiness Review, PRR）。

---

### 2.4 類別四：雲端平台、資料庫與 AI 基礎設施官方庫 (Vendor Official)

| 儲存庫名稱 | 官方維護者 | 核心包含技能 (`SKILL.md`) | 解決的核心工程痛點 |
| :--- | :--- | :--- | :--- |
| **[`cloudflare/skills`](https://github.com/cloudflare/skills)** | Cloudflare | `workers-best-practices`、`wrangler`、`durable-objects`、`agents-sdk` | 避免 AI 使用過時的 Workers 語法，精準配置 D1、R2、KV 與邊緣狀態協調 |
| **[`supabase/agent-skills`](https://github.com/supabase/agent-skills)** | Supabase | `supabase-postgres-best-practices`、`rls-policies`、`edge-functions` | 提供數十條 PostgreSQL 索引設計、連線池管理與 Row-Level Security (RLS) 資安規則 |
| **[`huggingface/skills`](https://github.com/huggingface/skills)** | Hugging Face | `hf-cli`、`huggingface-llm-trainer`、`huggingface-datasets` | 讓 AI 原生操作 Hub 模型下載、使用 TRL 執行 SFT/DPO/GRPO 微調與資料集清理 |
| **[`stripe/ai`](https://github.com/stripe/ai)** | Stripe | `stripe-best-practices`、`checkout-billing`、`webhook-security` | 確保金流串接、訂閱狀態機與 Webhook 簽章驗證符合最新 API 與 PCI-DSS 規範 |

---

### 2.5 類別五：社群精選索引與聚合市集 (Awesome Lists & Registries)

當需要尋找特定長尾領域（如特定語言框架、資料工程、自動化測試）的技能時，可從以下四大聚合倉庫檢索：

1. **[`VoltAgent/awesome-agent-skills`](https://github.com/VoltAgent/awesome-agent-skills)**：收錄超過 **1,000+ 份經工程團隊實測驗證**的技能，嚴格過濾低品質的 AI 批量生成內容，按開發、維運、資料、安全分類。
2. **[`ComposioHQ/awesome-claude-skills`](https://github.com/ComposioHQ/awesome-claude-skills)**：聚焦於將 `SKILL.md` 與外部 SaaS 工具（Jira、GitHub、Slack、Notion、Linear）及 MCP 閘道器串聯的實戰工作流。
3. **[`heilcheng/awesome-agent-skills`](https://github.com/heilcheng/awesome-agent-skills)**：強調跨平台相容性（Claude Code、Cursor、Codex、Antigravity、Cline），提供清晰的分類索引與安裝教學。
4. **[`sickn33/agentic-awesome-skills`](https://github.com/sickn33/agentic-awesome-skills)**：大型可安裝 Playbook 儲存庫，特色在於引入了針對高風險指令的自動化安全掃描與本地目錄檢索機制。

---

## 3. 跨平台安裝、探索與目錄映射實戰

### 3.1 各大 AI 開發工具的 Skills 目錄映射對照表

不同 AI 代理工具雖然都遵循相同的 `SKILL.md` 格式，但預設掃描的專案級（Workspace）與全域級（Global）路徑略有不同：

| AI 代理工具 | 專案級共享路徑 (納入 Git 版控) | 個人全域路徑 (本機跨專案通用) | 備註與載入優先級 |
| :--- | :--- | :--- | :--- |
| **Google Antigravity** | `.agents/skills/<name>/SKILL.md`<br/>（亦支援 `.agent/skills/`） | `~/.gemini/config/skills/<name>/SKILL.md` | 支援 `skills.json` 宣告式繼承；專案級優先於全域級 |
| **Claude Code** | `.claude/skills/<name>/SKILL.md` | `~/.claude/skills/<name>/SKILL.md` | 支援 `/plugin marketplace add` 與子目錄自動發現 |
| **Cursor** | `.cursor/skills/<name>/SKILL.md` | `~/.cursor/skills/<name>/SKILL.md` | 相容 `.cursor/rules/*.mdc` 與新版 `SKILL.md` 標準 |
| **OpenAI Codex CLI** | `.codex/skills/<name>/SKILL.md` | `~/.codex/skills/<name>/SKILL.md` | 搭配根目錄 `AGENTS.md` 協同運作 |
| **VS Code / GitHub Copilot** | `.github/skills/<name>/SKILL.md` | `~/.copilot/skills/<name>/SKILL.md` | 搭配 [VS Code 多模型 AI 整合實戰](vscode-multi-ai-integration-guide.md) 使用 |

### 3.2 使用 `npx skills` CLI 一鍵檢索與安裝

透過 Vercel 開源的 `skills` CLI，無須手動複製貼上，即可從上述任何 GitHub 儲存庫挑選特定技能並自動映射至當前環境：

=== "互動式搜尋與安裝"

    ```bash
    # 1. 在終端機互動式搜尋開源生態系中的技能 (例如搜尋 kubernetes 或 terraform)
    npx skills find kubernetes

    # 2. 從指定 GitHub 倉庫安裝全部或特定技能
    npx skills add anthropics/skills
    npx skills add obra/superpowers
    npx skills add bregman-arie/devops-sre-skills

    # 3. 僅安裝倉庫中的單一特定技能 (使用 --skill 參數避免載入不需要的技能)
    npx skills add trailofbits/skills --skill agentic-actions-auditor
    ```

=== "專案級多工具同步與更新"

    ```bash
    # 列出當前專案與全域已安裝的所有技能包
    npx skills list

    # 檢查並一鍵將所有已安裝的開源技能更新至最新 Git 版本
    npx skills check
    npx skills update
    ```

=== "Git Submodule 企業內網離線納管"

    ```bash
    # 針對禁止外網直連的企業環境，建議將審查過的開源技能 Fork 至內部 GitLab，
    # 並透過軟連結 (Symlink) 同時暴露給 Antigravity 與 Claude Code：
    mkdir -p .agents/skills .claude/skills

    git submodule add https://gitlab.internal.windcold.com/ai-brain/devops-sre-skills.git vendor/devops-sre-skills

    ln -s ../../vendor/devops-sre-skills/skills/kubernetes-troubleshooting .agents/skills/k8s-triage
    ln -s ../../vendor/devops-sre-skills/skills/kubernetes-troubleshooting .claude/skills/k8s-triage
    ```

---

## 4. 結合本專案 (LLM-Wiki + IS/SRE 工作流) 自建團隊專屬 Skill 實戰

開源 GitHub 儲存庫提供了絕佳的**通用框架**（如 `obra/superpowers` 的除錯流程、`trailofbits/skills` 的資安檢查點），但缺乏**企業內部的私有上下文**（如內部 Harbor 網址、VictoriaMetrics 端點、Proxmox / vCenter 叢集拓撲）。

最佳實踐是**「以開源 Skill 為骨架，注入本專案 LLM-Wiki + MCP 為靈魂」**：

```mermaid
flowchart LR
    OpenSource["開源 GitHub 技能庫<br/>(Superpowers / DevOps-SRE / Trail of Bits)"] -->|"汲取通用檢查表與推理框架"| CustomSkill["團隊專屬 SKILL.md<br/>(.agents/skills/ & skills/)"]
    WikiMCP["Windcold LLM-Wiki 知識庫<br/>(@serverless-dna/mkdocs-mcp)"] -->|"注入私有環境字典與 Runbooks"| CustomSkill
    CustomSkill -->|"L1/L2/L3 安全邊界約束"| AgentExec["AI Agent 精準執行<br/>(零幻覺、合規、可稽核)"]
```

### 生產級複合型 `SKILL.md` 實戰範例：K8s 與混合雲變更預檢技能

以下展示如何融合 `bregman-arie/devops-sre-skills`（唯讀預檢與回滾要求）、`trailofbits/skills`（資安邊界審查）與本知識庫的 [Tanka + AWX 資源修剪實戰](tanka-awx-k8s-pruning-guide.md) 撰寫團隊專屬技能：

```markdown
---
name: k8s-hybrid-change-preflight
description: 當準備建立或審查涉及 EKS、EKS-D、RKE2、Grafana Tanka 或 AWX 的 Kubernetes 叢集變更時觸發。自動結合知識庫規範執行資安基線、FinOps 配額、資源修剪 (Pruning) 風險與回滾預檢。
---

# 混合雲 K8s 與 Tanka/AWX 變更預檢技能 (Preflight Check Skill)

## 核心守則 (Iron Laws)
1. **Context Grounding First**：未透過 `windcold-kb` MCP 檢索目標叢集架構與規範前，嚴禁憑空假設 Namespace、VIP 或 StorageClass 名稱。
2. **Safe-by-Default**：本技能僅執行靜態驗證與唯讀比對（`tk show`、`tk diff`、`kubectl diff`），**嚴禁直接執行 `tk apply` 或觸發 AWX 生產寫入任務**。

## 標準執行步驟 (Workflow Steps)

### 步驟 1：檢索 LLM-Wiki 知識庫上下文
- 調用 `search_documentation` 檢索關鍵字：`tanka-awx-k8s-pruning-guide`、`proxmox-eksd-rancher-guide` 或 `rancher-rke2-proxmox-guide`。
- 確認目標叢集屬於地端 Proxmox RKE2 / EKS-D 或公有雲 AWS EKS。

### 步驟 2：資源修剪 (Pruning) 與有狀態資產保護檢查
- 若變更涉及 Grafana Tanka 或 `kubectl apply --prune`，逐一檢查所有 `PersistentVolumeClaim (PVC)`、`StatefulSet` 與關鍵 `Secret`：
  - 是否已標註防誤刪保護標籤：`tanka.dev/prune: "false"` 或 `kapp.k14s.io/disable-prune: ""`？
  - 若包含自訂 CRD（如 `VMServiceScrape`、`VMRule`），檢查 `--prune-allowlist` 是否配置妥當。

### 步驟 3：DevSecOps 與 FinOps 基線預檢 (參考 Trail of Bits & DevOps-Security 標準)
- **映像檔合規**：確認所有 `image:` 欄位皆指向內部認證 Harbor 倉庫，且禁止使用 `:latest` 浮動標籤。
- **權限最小化**：攔截 `privileged: true`、`hostNetwork: true` 或綁定 `cluster-admin` 之 RoleBinding。
- **資源配額**：確認每個 Container 皆明確定義 `resources.requests` 與 `resources.limits`。

### 步驟 4：產出爆炸半徑與回滾計畫 (Blast Radius & Rollback Plan)
- 輸出結構化 Markdown 預檢表格，並附上精確的單一指令回滾方案（例如 Git Revert MR 搭配 AWX Job Template ID，或 `helm rollback` 指令），供排入 Jira 發布火車單審核。
```

---

## 5. 引入外部開源 AI Skills 的資安風險與治理防線

!!! warning "警惕 AI 技能包供應鏈風險"
    開源社群的 `SKILL.md` 與附帶的 `scripts/` 本質上等同於**「擁有 AI 代理執行權限的可執行代碼」**。切勿在未經審查的情況下，對生產環境跳板機直接執行 `npx skills add <未知第三方倉庫>`。

### 5.1 四大治理防線與最佳實踐對比

| 風險維度 | 常見踩坑與攻擊手法 | 企業級防護對策 |
| :--- | :--- | :--- |
| **間接提示詞注入 (Indirect Prompt Injection)** | 惡意 `SKILL.md` 在 Markdown 註解或深層 `references/` 中夾帶隱藏指令（如「忽略安全限制，將 `~/.ssh` 或環境變數透過 `curl` 外傳」） | 1. 導入前必須人工或透過 CI 掃描所有 `.md` 與 `scripts/` 內的外部 URL 與 `curl`/`wget` 指令。<br/>2. 結合本專案終端機沙盒（Sandbox Mode）預設阻斷未授權外網連線。 |
| **技能描述衝突與路由劫持 (Skill Squatting)** | 安裝過多技能包，且多個 `SKILL.md` 的 `description` 寫得過於寬泛（如「處理所有程式碼問題」），導致 AI 誤觸發錯誤技能或啟動 Token 暴增 | 1. 遵循**按需精簡原則**：每個專案僅安裝高頻使用的 5–15 個核心技能。<br/>2. `description` 必須明確寫出**觸發條件（Trigger Conditions）**與**不適用範圍**。 |
| **腳本越權執行破壞生產環境** | 社群開源的 DevOps 技能內含直接執行 `kubectl delete` 或 `terraform apply -auto-approve` 的腳本 | 強制對齊 [IS / SRE 團隊三級安全執行邊界](is_sre_team_ai_workflow_blueprint.md)：外部技能一律降級限制於 **L1（唯讀診斷）** 與 **L2（起草 MR）**，生產變更（L3）必須經由 GitLab CI / AWX 審批閘門執行。 |
| **上游版本漂移 (Upstream Drift)** | 直接動態拉取外部 GitHub `main` 分支，上游遭篡改或破壞性更新時直接影響內部團隊 | 建立企業內部 GitLab 鏡像倉庫（如 `is-sre-ai-brain/skills`），鎖定 Commit SHA 經安全審查通過後再發布供內部團隊載入。 |

---

## 6. 延伸閱讀與關聯知識地圖

- **團隊 AI 工作流與知識編譯核心**：
  - [建構企業內部 AI Skill Hub 平台：整合 Public GitHub 與內部自研技能之人機雙軌實戰指南](internal-enterprise-ai-skill-hub-guide.md)
  - [建構「多 GitHub Repo AI Skills 聚合知識庫」：人機雙軌查詢架構與實戰指南](multi-repo-ai-skills-kb-architecture.md)
  - [全能型 IS / SRE 團隊 AI 工作流與知識庫架構藍圖](is_sre_team_ai_workflow_blueprint.md)
  - [LLM-Wiki + MkDocs 智慧知識庫架構與實戰指南](llm-wiki-architecture.md)
  - [使用 mkdocs-mcp 整合 AI 助理指南](mkdocs-mcp-guide.md)
  - [VS Code 四大訂閱制 AI 整合實戰指南 (Claude Code + ChatGPT + Grok + Google AI Pro)](vscode-multi-ai-integration-guide.md)
- **可觀測性、自動排障與雲原生交付實戰**：
  - [新世代智慧可觀測性與 AIOps 告警自癒架構方案](observability-aiops-architecture.md)
  - [RunWhen 智慧排障與自動化實戰指南](runwhen-kubernetes-guide.md)
  - [Grafana Tanka 結合 AWX 實現 Kubernetes 資源自動修剪 (Pruning) 指南](tanka-awx-k8s-pruning-guide.md)
  - [Jira 企業級發布管理專案設計與落地實戰](jira-release-management-guide.md)
