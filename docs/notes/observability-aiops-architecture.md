---
icon: material/monitor-dashboard
description: 整合 Grafana、VictoriaMetrics、Alertmanager、Keep、GoAlert，並串接 Claude AI 與自研系統，打造結合 MkDocs 知識庫與 Teams 的新世代 AIOps 告警自癒解決方案。
tags:
  - Observability
  - VictoriaMetrics
  - Grafana
  - Alertmanager
  - AIOps
  - Keep
  - GoAlert
  - Pushover
  - Claude
  - MkDocs
  - Teams
---

# 新世代智慧可觀測性與 AIOps 告警自癒架構方案 :material-monitor-dashboard:

> 結合 **Grafana**、**VictoriaMetrics**、**Alertmanager** 打造高效時序監控基石，透過 **Keep**、**GoAlert** 與 **Pushover** 實現智慧告警收斂、On-Call 輪值與穿透靜音強提醒，並深度串接 **Claude AI**、**MkDocs 知識庫 (MCP)**、**自研系統** 與 **Microsoft Teams**，建構自動化故障診斷與 ChatOps 一鍵自癒閉環。

---

## 1. 方案概述與核心挑戰

### A. 傳統監控與告警機制的困境

在微服務與分散式架構日益複雜的環境中，企業傳統的監控體系常面臨以下挑戰：

1. **告警疲勞 (Alert Fatigue) 與降噪不足**：單一底層故障（例如資料庫磁碟空間耗盡）可能引發數十個微服務相依性連鎖報警，值班人員被淹沒在重複或次要的告警中。
2. **缺乏即時上下文與根因分析 (RCA)**：傳統通知僅提供冷冰冰的指標數值，排錯需人工翻查各儀表板、日誌與程式碼，無法迅速指出根本成因。
3. **知識庫 (Runbooks/SOP) 割裂**：維運手冊與 SOP 散落在內部文件系統或 Wiki，事故發生當下值班人員難以第一時間查得對應的處置指南。
4. **On-Call 輪值與升級機制不完善**：重要事故因缺乏強制的階梯升級政策 (Escalation Policy) 或電話/語音催辦，容易造成嚴重的延遲處理。
5. **故障處置無法形成自動化閉環**：排查與修復操作需跳轉多個內部系統（自研控制台、K8s CLI、資料庫後台），缺乏標準化的 ChatOps 協同與一鍵自癒能力。

### B. 新世代 AIOps 方案核心價值

本方案針對上述痛點，設計了整合 **監控 ➔ 告警 ➔ 降噪 ➔ AI 診斷 ➔ 知識庫檢索 ➔ On-Call 排班 ➔ ChatOps 協同 ➔ 自研系統自癒** 的完整技術棧：

- **極致高效的監控基石**：採用 **VictoriaMetrics** 替代傳統 Prometheus TSDB，提供高出數倍的寫入吞吐量、低至 1/5 的儲存空間與 1/7 的記憶體消耗，完整相容 PromQL (MetricsQL)。
- **智慧告警編排與 AIOps 中樞**：透過 **Keep (KeepHQ)** 作為跨平台告警匯流排，統一 Alertmanager、Grafana 與自研系統的告警資料模型，執行關聯分析與去重。
- **Claude AI 賦能與 MkDocs 知識庫連動**：在告警觸發第一時間，**Claude** 透過 **Model Context Protocol (MCP)** 自動調閱 **MkDocs 技術知識庫** 內的專屬 Runbook，並調用 VictoriaMetrics 指標趨勢，即時生成根因診斷與修復建議。
- **分級 On-Call 輪值與緊急強提醒 (GoAlert + Pushover)**：整合 **GoAlert** 處理排班與電信語音/SMS 階梯升級；同時結合 **Pushover** 的 **iOS Critical Alerts** 機制，當發生 P1 致命故障時強制穿透值班人員手機的「實體靜音開關與勿擾模式」，並以每 30 秒奪命連環催確保半夜絕不漏接。
- **Teams 互動卡片與自研系統閉環**：透過 **Microsoft Teams Adaptive Cards**，讓值班人員直接在群組內檢閱 AI 診斷摘要、查閱 SOP 連結，並直接點擊自癒按鈕回呼自研系統執行修復動作。

---

## 2. 全景系統架構與資料流向

### A. 整體架構拓撲圖

```mermaid
flowchart TD
    subgraph DataSources["1. 數據採集與業務指標層"]
        K8s["Kubernetes 集群 / Exporters"]
        InHouseApp["自研系統業務服務 (In-House Services)"]
        InHouseJob["自研定時批次 / 工作流 (Workers)"]
    end

    subgraph MetricsPlatform["2. 時序監控與規則評估層"]
        vmagent["vmagent (採集代理)"]
        VM[("VictoriaMetrics (TSDB 時序資料庫)")]
        vmalert["vmalert (規則評估引擎)"]
        Grafana["Grafana (視覺化儀表板)"]
    end

    subgraph AlertRouting["3. 告警過濾與 AIOps 中樞"]
        AM["Alertmanager (分組 / 抑制 / 靜音)"]
        Keep["Keep (KeepHQ AIOps 告警編排中樞)"]
    end

    subgraph AIAndKnowledge["4. AI 智慧診斷與 Runbook 檢索"]
        Claude["Anthropic Claude 3.5 (AI 診斷引擎)"]
        MkDocsMCP["MkDocs MCP Server (@serverless-dna/mkdocs-mcp)"]
        MkDocsKB[("MkDocs 團隊知識庫 (Runbooks / SOPs)")]
    end

    subgraph EscalationAndChatOps["5. 排班升級與協同自癒"]
        GoAlert["GoAlert (On-Call 排班 / 階梯升級)"]
        Pushover["Pushover (手機 App 緊急強提醒 / 穿透靜音)"]
        Teams["Microsoft Teams (互動式 Adaptive Cards)"]
        InHouseAPI["自研系統維運 API (一鍵自癒 / 降級開關)"]
    end

    %% 資料流連線
    K8s -->|Prometheus Metrics| vmagent
    InHouseApp -->|Push / Pull Metrics| vmagent
    InHouseJob -->|Pushgateway / API| vmagent
    vmagent -->|Remote Write| VM
    Grafana <-->|MetricsQL 查詢| VM
    vmalert -->|定期評估查詢| VM
    vmalert -->|發送觸發告警| AM

    AM -->|Webhook 送出 Raw Alerts| Keep
    InHouseApp -.->|自研自定義業務告警 Webhook| Keep

    %% Keep 工作流
    Keep -->|觸發 AIOps Workflow| Claude
    Claude <-->|MCP 檢索相關 SOP| MkDocsMCP
    MkDocsMCP <-->|讀取 search_index.json| MkDocsKB
    Claude <-->|查詢近期指標異常| VM

    Keep -->|P1/P2 高優先級事故| GoAlert
    Keep -->|"P1 致命事故 (Priority=2)"| Pushover
    Pushover -->|"iOS Critical Alerts / 穿透勿擾"| SRE["值班 SRE / 工程師 (手機/Apple Watch)"]
    GoAlert -->|"電話語音 / SMS 催辦"| SRE

    Keep -->|發布 AI 豐富化卡片| Teams
    Teams -.->|一鍵確認 / 認領| GoAlert
    Teams -->|一鍵執行自癒動作| InHouseAPI
    InHouseAPI -->|自動重啟 / 擴容 / 清除連線| InHouseApp
```

### B. 告警全生命週期步驟詳解

| 階段 | 負責組件 | 執行動作與核心邏輯 | 預期時效 |
| :--- | :--- | :--- | :--- |
| **1. 異常偵測** | 自研系統 / VM | 自研業務或基礎架構指標突破閾值（如 API 5xx 比例 > 5%），寫入 VictoriaMetrics。 | $T + 0\text{s}$ |
| **2. 規則評估** | vmalert | 依排程頻率（如每 15 秒）評估 PromQL 規則，滿足 `for: 1m` 後向 Alertmanager 發送 Alert。 | $T + 15\text{s}$ |
| **3. 初步收斂** | Alertmanager | 依據標籤（如 `cluster`, `env`, `service`）進行抑制、分組與靜音，透過 Webhook 轉交 Keep。 | $T + 20\text{s}$ |
| **4. AIOps 編排** | KeepHQ | 接收結構化告警，依據工作流引擎分派：呼叫 Claude 進行分析、準備通知頻道。 | $T + 22\text{s}$ |
| **5. AI 根因研判** | Claude + MkDocs MCP | Claude 透過 MCP 查閱 MkDocs 對應的故障排查手冊，比對 VM 近期指標，產出 150 字 RCA 與建議。 | $T + 30\text{s}$ |
| **6. 協同與呼叫** | Teams + Pushover + GoAlert | Teams 呈現豐富化卡片；Pushover 針對值班人員發動穿透靜音的強制強提醒 (每 30 秒催促)；若逾時無人認領，GoAlert 發動電話語音叫醒。 | $T + 35\text{s}$ |
| **7. 一鍵自癒** | Teams ➔ 自研系統 | 工程師在 Teams 卡片點擊「執行隔離/重啟」，WebHook 呼叫自研維運 API，系統恢復正常。 | $T + 60\text{s}$ |

---

## 3. 各組件職責定位與選型優勢

### A. VictoriaMetrics vs 傳統 Prometheus

| 評比維度 | VictoriaMetrics | 傳統 Prometheus |
| :--- | :--- | :--- |
| **記憶體消耗** | **極低**（約 Prometheus 的 $1/5 \sim 1/7$） | 較高，大規模時間序列易發生 OOM |
| **磁碟儲存效率** | **極高壓縮比**（ZSTD 演算法，減少 $70\%\sim80\%$ 空間） | 標準區塊壓縮，歷史數據佔比大 |
| **長期歷史查詢** | 原生支援單機或叢集無限橫向擴展 | 需外掛 Thanos / Cortex 等複雜組件 |
| **查詢語法** | **MetricsQL**（完全向後相容 PromQL，並提供多項延伸函數） | 標準 PromQL |
| **輕量架構** | 單一二進位檔即可運行，組態極簡 | 需要額外維護 TSDB retention 與 WAL |

### B. Keep (KeepHQ) AIOps 引擎

KeepHQ 在此架構中扮演**告警中樞 (Alert Aggregation Hub)** 與 **AIOps 編排大腦 (Workflow Orchestrator)** 的雙重關鍵角色：

- **多來源統一資料模型 (Unified Alert Data Model)**：原生對接 Prometheus Alertmanager、VictoriaMetrics、Datadog、Sentry、AWS CloudWatch 及自研系統 Webhook。Keep 將異質告警原始 JSON 映射至統一的內部資料結構（標準欄位包含 `name`、`status`、`severity`、`source`、`labels`、`annotations`、`fingerprint` 等），終結監控孤島。
- **動態關聯與告警降噪 (Topological Correlation & Deduplication)**：採用滑動時間窗口（Time-Window Bucketing）與拓撲依賴標籤，在告警湧入時自動聚合次生事件，將數十個零散告警收斂為單一「事故群組 (Incident)」，避免 On-Call 人員遭受告警風暴洗禮。
- **宣告式自動化工作流 (Declarative Workflow as Code)**：使用簡潔明瞭的 YAML 語法定義工作流觸發條件（Triggers）、執行步驟（Steps）與外派動作（Actions）。原生內建超過 50 種 Providers，包括 LLM (Anthropic、OpenAI、Ollama)、通訊平台 (Teams、Slack)、協同工具 (Jira、GitHub) 與 On-Call 系統 (GoAlert、PagerDuty)。
- **AI 根因研判與原位告警豐富化 (In-Place Alert Enrichment)**：傳統告警僅能傳遞「哪裡壞了 (What)」，缺乏「為何壞 (Why)」與「如何修 (How)」。Keep 的核心威力在於能調度跨維度資訊（時序指標、知識庫 Runbook、程式變更歷史）餵給大語言模型進行深度根因推演 (RCA)，並將 AI 診斷結果直接原位寫回 Keep Alert/Incident 資料庫的 `enrichments` 屬性中，提供 Web 控制台即時檢視與下游通知管道無縫引用。

### C. GoAlert 開源 On-Call 管理

- **靈活的輪替排班 (Rotations & Schedules)**：支援按週、按日、白夜班輪值排班，解決休假與代理人換班需求。
- **階梯式升級策略 (Escalation Policies)**：若第一線值班者於 5 分鐘內未 Acknowledge，自動升級呼叫第二線主管或主管群，並支援語音電話呼叫。
- **純開源與無授權費用**：企業完全自主掌控數據與隱私，降低昂貴的商業 SaaS (如 PagerDuty / Opsgenie) 費用。

### D. Claude + MkDocs 知識庫 (MCP 連線架構)

- **傳統痛點**：告警內容通常只有 `summary` 與 `description`，缺乏處置智慧，新進同仁常不知如何下手。
- **本方案創新**：MkDocs 透過 `@serverless-dna/mkdocs-mcp` 將企業內部積累的所有技術 SOP、Runbook、架構文檔封裝為 MCP 工具。Claude 接收到告警後，直接調用 `search_documentation` 獲取「該故障的官方標準處置流程」，並結合自研系統標籤與即時指標，組合成客製化診斷報告。

### E. Pushover 專屬行動推播與緊急強提醒

- **補足開源工具無手機 App 缺口**：GoAlert 原生未提供官方 iOS/Android App，結合 Pushover 即可獲得成熟的行動端與 Apple Watch 即時推播能力。
- **iOS Critical Alerts 穿透靜音開關**：針對 P1 致命故障發送 `priority=2`，可強制突破 iPhone 實體靜音鍵與系統「勿擾/睡眠模式」，發出高分貝警報。
- **定時奪命連環催**：支援 `retry=30`（每 30 秒重複鳴響），直到工程師在手機上點擊確認 (Acknowledge) 才會停止。
- **極致性價比**：相較於商業 SaaS 每人每月數十美元的費用，Pushover 個人端為 **$4.99 USD 一次性買斷制**（終生零月費），每月享 10,000 則免費額度。
- **專屬文件參考**：完整功能介紹、API 呼叫與費用方案請參閱 [Pushover 即時推播服務指南](pushover-guide.md)。

---

## 4. 端到端核心組態與設定實戰

### A. VictoriaMetrics 與 vmalert 規則配置

在 `vmalert` 的規則檔案中定義告警條件，標註自研系統標籤與 MkDocs 知識庫 Runbook 連結：

```yaml
# /etc/vmalert/rules/inhouse-alerts.yml
groups:
  - name: InHouseServiceAlerts
    rules:
      - alert: InHouseApiHighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[2m])) by (service, environment)
            /
            sum(rate(http_requests_total[2m])) by (service, environment)
          ) * 100 > 5
        for: 1m
        labels:
          severity: critical
          tier: backend
          service: "{{ $labels.service }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "自研系統服務 {{ $labels.service }} API 錯誤率超過 5%"
          description: "在過去 2 分鐘內，{{ $labels.service }} 的 HTTP 5xx 錯誤率達到 {{ $value | printf \"%.2f\" }}%，已超過 SLA 警示閾值。"
          runbook_url: "https://windcoldchen.github.io/windcold-kb/notes/workflow/#incident-response"
          dashboard_url: "https://grafana.internal.windcold.com/d/inhouse-api?var-service={{ $labels.service }}"
```

### B. Alertmanager 轉送設定

配置 Alertmanager 將接收到的告警轉送至 KeepHQ 的 Webhook 端點：

```yaml
# /etc/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'service', 'environment']
  group_wait: 10s
  group_interval: 30s
  repeat_interval: 2h
  receiver: 'keep-aiops-hub'

receivers:
  - name: 'keep-aiops-hub'
    webhook_configs:
      - url: 'http://keep-backend.monitoring.svc.cluster.local:8080/v1/alerts/event/prometheus'
        send_resolved: true
        http_config:
          bearer_token: 'KEEP_INGEST_API_KEY_SECRET'
```

### C. Keep AIOps 工作流架構與基本概念

KeepHQ 採用宣告式工作流（Workflow as Code），所有自動化處理邏輯皆以 YAML 檔案維護，具備以下四大核心要素：

1. **Triggers（觸發條件）**：定義工作流何時啟動。支援依據告警屬性（如 `severity: critical|high`、`service: payment-gateway`）、來源（`source: prometheus`）或定時排程（Cron）進行過濾。
2. **Steps（執行步驟）**：按順序執行上下文蒐集、數據關聯與推理。步驟可呼叫內建 Provider（如 HTTP、Python 腳本、SQL 查詢、VictoriaMetrics API、LLM 大模型）。前一步驟的輸出（`steps.<step-name>.output`）可直接被後續步驟作為 Jinja2 變數引用。
3. **Enrichments（原位告警豐富化）**：此為 AIOps 最核心能力。工作流能將 AI 推論產生的根因報告、信心度或排查連結，動態寫入該告警實體的 `enrichments` 屬性中，讓 Web UI 與後續動作隨取隨用。
4. **Actions（派案與通知動作）**：根據告警等級分流派發至通訊頻道（Teams、Slack）、階梯式呼叫平台（GoAlert）、緊急強提醒（Pushover）或自研自癒 Webhook。

---

### D. KeepHQ AI 根因研判 (AI RCA) 核心機制與運作管線

傳統告警體系的最大痛點是「知其然不知其所以然」：值班人員收到「API 5xx 錯誤率 > 5%」的告警後，仍需耗費大量時間登入 Grafana 查指標、登入伺服器查日誌、翻閱 Wiki 找 SOP。

本方案在 KeepHQ 中建構的 **AI 根因研判 (AI Root Cause Analysis, AI RCA)** 機制，在告警到達 Keep 的前 10 秒內，自動完成「異常指標追溯 + 知識庫 Runbook 檢索 + 大模型思維鏈推演 + 原位告警豐富化」的無人值守診斷閉環。

#### 1. AI 根因研判全鏈路運作架構

```mermaid
flowchart TD
    Alert["Alertmanager / 自研業務告警<br/>(Webhook Ingest)"] --> KeepHub["KeepHQ AIOps 引擎<br/>(動態聚類與排程分派)"]

    subgraph StepContext["步驟一：跨源上下文動態採集 (Multi-Source Context)"]
        KeepHub --> CtxVM["1. VictoriaMetrics PromQL 查詢<br/>(拉取過去 15 分鐘關鍵指標曲線)"]
        KeepHub --> CtxKB["2. MkDocs MCP / 知識庫檢索<br/>(精確檢索對應服務 Runbook / SOP)"]
        KeepHub --> CtxMeta["3. 提取 Kubernetes / 業務標籤<br/>(Service、Namespace、Pod、錯誤率)"]
    end

    CtxVM --> PromptBuilder["Prompt 動態渲染器<br/>(Jinja2 整合告警、時序數據與 SOP)"]
    CtxKB --> PromptBuilder
    CtxMeta --> PromptBuilder

    subgraph StepAI["步驟二：AI 根因研判與推理 (Reasoning Plane)"]
        PromptBuilder --> AIModel["Anthropic Claude 3.5 Sonnet /<br/>私有化 Ollama (DeepSeek / Qwen)"]
        AIModel --> CoT["SRE 思維鏈 (Chain of Thought)<br/>比對指標異常、錯誤模式與知識庫"]
        CoT --> OutputParser["結構化 JSON 輸出驗證<br/>(根因假設 + 信心度 + 處置代碼)"]
    end

    subgraph StepEnrich["步驟三：原位告警豐富化 (Alert Enrichment)"]
        OutputParser --> KeepDB["Keep 告警資料庫<br/>寫入 alert.enrichments.ai_rca"]
    end

    subgraph StepDispatch["步驟四：多管道精準協同與派案 (Dispatch & Action)"]
        KeepDB --> Teams["Microsoft Teams<br/>(互動式 Adaptive Card 卡片)"]
        KeepDB --> GoAlert["GoAlert 階梯 On-Call<br/>(附帶 AI 診斷之語音電話 / SMS)"]
        KeepDB --> Pushover["Pushover 緊急推播<br/>(iOS 穿透靜音強提醒)"]
        KeepDB --> AutoHeal["自研自癒 Controller<br/>(依據 AI 建議提供一鍵修復 Webhook)"]
    end
```

---

### E. Keep 原生 AI Provider 與本地私有化 LLM 配置實踐

KeepHQ 支援原生調用雲端大模型或內部私有化模型，維運團隊可依資安等級自由切換：

#### 1. 原生公有雲 Provider (Anthropic Claude / OpenAI)

在 Keep 後台 Settings ➔ Providers 或透過環境變數注入 API Key：

```bash
# 啟動 Keep 時注入金鑰
export ANTHROPIC_API_KEY="sk-ant-api03-xxxx"
export OPENAI_API_KEY="sk-proj-xxxx"
```

#### 2. 地端私有化與離線模型配置 (Ollama / vLLM / DeepSeek-R1 / Qwen 2.5)

針對金融業、醫療或機敏資料嚴禁出境之環境，可在內部架設高效能推論引擎（如 Ollama 或 vLLM），並透過相容 OpenAI API 介面接入 Keep：

```yaml
# /etc/keep/providers/ollama_provider.yml
providers:
  - id: internal-ollama-llm
    type: openai
    config:
      api_key: "internal-dummy-key"
      api_url: "http://ollama-server.internal.windcold.com:11434/v1"
      model: "deepseek-r1:14b" # 或 qwen2.5-coder:14b
```

---

### F. 生產級多階段 AI RCA Workflow 工作流實作 (Keep YAML)

以下為完整的生產級工作流設定檔，實現「自動拉取 VictoriaMetrics 遙測數據 ➔ 檢索 MkDocs SOP ➔ 呼叫 Claude 3.5 診斷 ➔ 原位 Enrich 告警 ➔ 派發 Teams / Pushover / GoAlert」：

```yaml
# /etc/keep/workflows/aiops_incident_enrichment.yml
workflow:
  id: inhouse-incident-aiops-workflow
  description: "端到端 AIOps 告警富化：整合 VictoriaMetrics 時序指標、MkDocs Runbook 與 Claude AI 根因研判"
  triggers:
    - type: alert
      filters:
        - key: severity
          value: "critical|high"

  steps:
    # 步驟 1：向 VictoriaMetrics 查詢近 15 分鐘的指標趨勢 (異常時序遙測)
    - name: fetch-metric-telemetry
      provider:
        type: http
        config:
          url: "http://victoria-metrics.monitoring.svc.cluster.local:8428/api/v1/query_range"
          method: GET
          params:
            query: 'sum(rate(http_requests_total{service="{{ alert.labels.service }}"}[2m])) by (status)'
            step: "30s"
            start: "-15m"
            end: "now"

    # 步驟 2：向 MkDocs 知識庫或 MCP 服務檢索標準處置手冊 (Runbook)
    - name: fetch-mkdocs-runbook
      provider:
        type: http
        config:
          url: "http://ai-agent-service.internal.windcold.com/v1/runbook/search"
          method: GET
          params:
            keyword: "{{ alert.labels.service }}"

    # 步驟 3：調用 Claude 3.5 Sonnet 大模型執行 SRE 根因推演 (AI RCA)
    - name: ai-rca-reasoning
      provider:
        type: anthropic
        config:
          model: "claude-3-5-sonnet-20241022"
          temperature: 0.2
          max_tokens: 800
          system_prompt: |
            你是一位擁有 10 年資歷的頂尖 Site Reliability Engineer (SRE) 專家。
            請根據提供的「告警原始資訊」、「VictoriaMetrics 即時指標遙測數據」以及「MkDocs 官方知識庫 Runbook」，
            以客觀嚴謹、具備因果邏輯鏈的方式產出根因分析。
            嚴格規定：
            1. 僅基於給定的數據進行合理推論，嚴禁憑空捏造不存在的錯誤。若指標不足，請如實標明「指標不足，需額外排查日誌」。
            2. 回應必須為嚴格的 JSON 格式，不得包含任何額外解說文字或 Markdown 標籤。
          prompt: |
            【告警基本資訊】：
            - 告警名稱：{{ alert.name }}
            - 影響服務：{{ alert.labels.service }}
            - 嚴重層級：{{ alert.labels.severity }}
            - 環境：{{ alert.labels.environment }}
            - 描述：{{ alert.annotations.description }}

            【VictoriaMetrics 近 15 分鐘指標遙測】：
            {{ steps.fetch-metric-telemetry.output.body | string | truncate(1500) }}

            【MkDocs 官方 Runbook 指南】：
            {{ steps.fetch-mkdocs-runbook.output.body | string | truncate(1500) }}

            請輸出符合以下 JSON Schema 的診斷結果：
            {
              "root_cause_hypothesis": "核心根因假設（80字內，說明可能直接導因）",
              "confidence_score": 85,
              "evidence": "從指標或描述中獲得的關鍵佐證",
              "recommended_actions": [
                "緊急處置動作一（條列具體指令或步驟）",
                "預防與根治措施二"
              ],
              "auto_heal_action": "建議自動自癒動作代碼 (例如 scale_db_pool, restart_pod, clear_cache, none)",
              "safe_to_auto_heal": true
            }

    # 步驟 4：Keep 原位告警豐富化 (Alert Enrichment)
    # 將 AI RCA 診斷與修復建議寫入 Keep 告警主體，讓 Web UI 與所有後續動作共享
    - name: enrich-keep-alert
      provider:
        type: keep
        config:
          action: "enrich-alert"
          alert_fingerprint: "{{ alert.fingerprint }}"
          enrichments:
            ai_rca_hypothesis: "{{ (steps.ai-rca-reasoning.output.content | from_json).root_cause_hypothesis }}"
            ai_confidence: "{{ (steps.ai-rca-reasoning.output.content | from_json).confidence_score }}%"
            ai_recommended_action: "{{ (steps.ai-rca-reasoning.output.content | from_json).recommended_actions | join('；') }}"
            ai_auto_heal_suggested: "{{ (steps.ai-rca-reasoning.output.content | from_json).auto_heal_action }}"

    # 步驟 5：派案至 GoAlert (當為 critical 致命告警時觸發語音/SMS 升級)
    - name: trigger-goalert
      if: "alert.labels.severity == 'critical'"
      provider:
        type: http
        config:
          url: "https://goalert.internal.windcold.com/api/v2/generic/incoming?token=GOALERT_INTEGRATION_KEY"
          method: POST
          headers:
            Content-Type: "application/json"
          body:
            summary: "[{{ alert.labels.environment }}] {{ alert.name }} - {{ alert.labels.service }}"
            details: "【Claude AI RCA】{{ (steps.ai-rca-reasoning.output.content | from_json).root_cause_hypothesis }}\n信心度: {{ (steps.ai-rca-reasoning.output.content | from_json).confidence_score }}%\n建議處置: {{ (steps.ai-rca-reasoning.output.content | from_json).recommended_actions | join('\n') }}\nRunbook: {{ alert.annotations.runbook_url }}"

    # 步驟 6：Pushover 發送 Priority 2 穿透靜音強提醒 (每 30 秒奪命連環催)
    - name: notify-pushover-emergency
      if: "alert.labels.severity == 'critical'"
      provider:
        type: http
        config:
          url: "https://api.pushover.net/1/messages.json"
          method: POST
          headers:
            Content-Type: "application/x-www-form-urlencoded"
          body:
            token: "ENV_PUSHOVER_APP_TOKEN"
            user: "ENV_PUSHOVER_ONCALL_GROUP"
            title: "🔥 [CRITICAL] {{ alert.name }} - {{ alert.labels.service }}"
            message: "AI 根因研判：{{ (steps.ai-rca-reasoning.output.content | from_json).root_cause_hypothesis }}\n信心度：{{ (steps.ai-rca-reasoning.output.content | from_json).confidence_score }}%\n請儘速處置！"
            priority: "2"
            retry: "30"
            expire: "3600"
            url: "{{ alert.annotations.runbook_url }}"
            url_title: "📖 開啟 MkDocs Runbook"

  # 派送動作：向 Teams 告警群組推送豐富化後的互動卡片
  actions:
    - name: notify-teams-channel
      provider:
        type: teams
        config:
          webhook_url: "https://windcold.webhook.office.com/webhookb2/TEAMS_WORKFLOW_KEY"
          template: "teams-adaptive-card-template.json"
          params:
            alert: "{{ alert }}"
            rca: "{{ steps.ai-rca-reasoning.output.content | from_json }}"
```

---

### G. SRE 工業級 Prompt Engineering 規範與結構化輸出設計

為了防止大模型產生幻覺（Hallucination）或給予模稜兩可的建議，Keep 中的 AI 提示詞應落實以下四大標準規範：

1. **角色賦予與情境約束 (Role & Grounding)**：
   - 明確設定 SRE 角色與事故影響範圍。
   - 限制大模型**僅能依據上下文提供的時間序列數值（如 HTTP 5xx 激增比例、MySQL 連線池飽和度、記憶體使用百分比）與 Runbook 文件**進行推理。
2. **思維鏈推理要求 (Chain of Thought, CoT)**：
   - 指導模型依序比對：`指標異常現象` $\rightarrow$ `拓撲依賴關聯` $\rightarrow$ `知識庫已知案例` $\rightarrow$ `得出根因假設`。
3. **嚴格 JSON Schema 約束**：
   - 規範輸出特定 JSON 欄位（`root_cause_hypothesis`、`confidence_score`、`recommended_actions`、`auto_heal_action`、`safe_to_auto_heal`）。
   - 確保下游 Jinja2 模板（如 Teams 卡片、GoAlert）可透過 `from_json` 進行欄位存取，避免字串解析失敗。
4. **置信度評分機制 (Confidence Scoring)**：
   - 當上下文同時包含指標時序與精確 Runbook SOP 時，置信度應 $\ge 80\%$；若缺少特定指標數據或 Runbook 僅為泛型說明，置信度應自動降至 $< 60\%$，並標示需要人工進一步比對日誌。

---

### H. 進階代理架構：Claude + MkDocs MCP 智慧診斷微服務後台

若企業希望在 Keep 之外維護更強大的獨立診斷代理（例如支援 FastMCP、多輪問答或調用自研除錯工具），可將步驟委託給獨立的 Python AI Agent 微服務：

```python
"""
Claude AI RCA 診斷微服務：
整合 Anthropic Claude 3.5 Sonnet、VictoriaMetrics API 與 MkDocs MCP Server，
結合即時指標與知識庫 Runbook 產出結構化高精確度診斷報告。
"""
from anthropic import Anthropic
import os
import json
import requests

anthropic_client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
VM_SELECT_URL = os.getenv("VM_URL", "http://victoria-metrics:8428/api/v1/query")

def fetch_victoriametrics_metric(query_expr: str) -> dict:
    """查詢 VictoriaMetrics 當前異常指標數值與趨勢"""
    try:
        resp = requests.get(VM_SELECT_URL, params={"query": query_expr}, timeout=3)
        if resp.status_code == 200:
            return resp.json().get("data", {}).get("result", [])
    except Exception as err:
        print(f"查詢 VictoriaMetrics 失敗: {err}")
    return {}

def get_mkdocs_runbook_content(service_name: str) -> str:
    """透過 HTTP 查詢 MkDocs search_index.json 精確抓取相應故障處置章節內容"""
    search_url = "https://windcoldchen.github.io/windcold-kb/search/search_index.json"
    try:
        resp = requests.get(search_url, timeout=5)
        if resp.status_code == 200:
            docs = resp.json().get("docs", [])
            for doc in docs:
                title = doc.get("title", "").lower()
                text = doc.get("text", "").lower()
                if service_name.lower() in title or service_name.lower() in text:
                    return f"### 文件標題: {doc.get('title')}\n{doc.get('text')[:1200]}"
    except Exception as e:
        print(f"查詢 MkDocs 知識庫失敗: {e}")
    return "無直接匹配的專屬 Runbook，請參考微服務通用排查手冊。"

def analyze_incident_rca(alert_name: str, service: str, description: str, promql_query: str) -> dict:
    """執行全鏈路根因研判推理"""
    # 1. 檢索知識庫 SOP
    sop_content = get_mkdocs_runbook_content(service)

    # 2. 補充指標遙測數據
    telemetry_data = fetch_victoriametrics_metric(promql_query) if promql_query else "無指定指標表達式"

    # 3. 呼叫 Claude 3.5 Sonnet
    system_prompt = (
        "你是一位擁有 10 年資歷的頂尖 Site Reliability Engineer (SRE) 專家。"
        "請根據傳入的告警資訊、當前 VictoriaMetrics 即時指標以及 MkDocs 知識庫 Runbook，"
        "以客觀、精準的繁體中文完成根因診斷，並以嚴格的 JSON 格式輸出。"
    )

    user_prompt = f"""
    【告警名稱】：{alert_name}
    【影響服務】：{service}
    【告警描述】：{description}
    【即時指標數據】：
    {json.dumps(telemetry_data, ensure_ascii=False)}
    【MkDocs Runbook 參考內容】：
    {sop_content}

    請輸出標準 JSON：
    {{
      "root_cause_hypothesis": "核心根因假設 (80字內)",
      "confidence_score": 85,
      "recommended_actions": ["即時處置動作一", "根本預防措施二"],
      "suggested_remediation_action": "scale_db_pool",
      "summary_markdown": "在 Teams 卡片呈現的Markdown重點摘要 (150字內)"
    }}
    """

    response = anthropic_client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=800,
        temperature=0.2,
        system=system_prompt,
        messages=[{"role": "user", "content": user_prompt}]
    )

    try:
        parsed_result = json.loads(response.content[0].text)
    except Exception:
        parsed_result = {
            "root_cause_hypothesis": "解析 AI 回應為 JSON 失敗",
            "confidence_score": 50,
            "recommended_actions": ["檢閱原始告警與日誌"],
            "suggested_remediation_action": "none",
            "summary_markdown": response.content[0].text
        }

    return parsed_result
```

---

### I. Keep Web UI 視覺化呈現與驗證指引

完成 AI RCA 工作流配置後，維運工程師可直接在 KeepHQ Web 控制台即時觀察運作成果：

1. **Alerts 總覽與原位富化展示**：
   - 登入 Keep UI（預設 `http://<keep-host>:3000`），點擊側邊欄 **Alerts**。
   - 告警列表中將直接顯示 `ai_rca_hypothesis` 標籤與 `ai_confidence` 徽章（如 `85%`）。
   - 點開任一告警展開側欄抽屜（Drawer），切換至 **Enrichments** 分頁，即可檢閱由 Claude 3.5 自動生成的根因推演依據、指標快照與處置指引。
2. **工作流手動觸發與本機模擬驗證**：
   - 可利用 Keep CLI 或 Webhook API 模擬注入測試告警驗證整條推理鏈：

     ```bash
     # 模擬向 Keep 發送測試告警
     curl -X POST http://localhost:8080/v1/alerts/event/prometheus \
       -H "Authorization: Bearer KEEP_INGEST_API_KEY_SECRET" \
       -H "Content-Type: application/json" \
       -d '{
         "receiver": "keep-aiops-hub",
         "status": "firing",
         "alerts": [{
           "status": "firing",
           "labels": {
             "alertname": "InHouseApiHighErrorRate",
             "severity": "critical",
             "service": "payment-gateway-api",
             "environment": "production"
           },
           "annotations": {
             "summary": "自研支付網關 API 錯誤率超過 5%",
             "description": "過去 2 分鐘內 HTTP 5xx 錯誤率達 12.8%，觸發臨界線。",
             "runbook_url": "https://windcoldchen.github.io/windcold-kb/notes/workflow/"
           },
           "startsAt": "2026-10-23T05:00:00Z"
         }]
       }'
     ```

   - 在 Keep **History (Execution Logs)** 頁籤中即可檢視各步驟執行時間、Claude 回應與 Enrichment 寫入日誌。

---

### J. 資料安全、脫敏與合規防線

在將企業內部的告警資料與指標傳遞至大模型進行 RCA 前，必須設立三道資安防護閘門：

1. **機敏資訊正則脫敏 (Data Sanitization)**：
   - 在 Prompt 動態組裝前，利用正則過濾器自動隱匿資料庫連線密碼、JWT Token、內部憑證、身分證號或手機號碼等 PII 資訊：

     ```python
     # 脫敏正則遮蔽範例
     sanitized_desc = re.sub(r'(password|token|secret)=([^\s&]+)', r'\1=***REDACTED***', raw_desc, flags=re.I)
     ```

2. **零訓練承諾 (Zero Data Retention / Enterprise Privacy)**：
   - 商業大模型 API（如 Anthropic Claude API、OpenAI API for Teams/Enterprise）皆預設遵循 API 資料不被用於模型訓練的規範。
3. **Air-Gapped 本地私有化隔離部署**：
   - 國防、金融或嚴苛法規要求時序與拓撲結構不得接觸網際網路者，優先採行 **Section 4-E** 介紹的內部 **Ollama + DeepSeek-R1 / Qwen 2.5-Coder** 方案，實現 100% 離線推論與零資料外洩。

---

## 5. Microsoft Teams 互動式 Adaptive Card 設計

當告警經過 Keep 與 Claude 豐富化後，會向 Teams SRE 告警頻道推送符合 **Adaptive Cards 1.5** 規範的卡片。

### A. 卡片視覺結構

```text
┌─────────────────────────────────────────────────────────────────┐
│ 🚨 [CRITICAL] InHouseApiHighErrorRate - 支付網關 API            │
│ 環境: Production | 觸發時間: 2026-10-21 23:25:00 CST            │
├─────────────────────────────────────────────────────────────────┤
│ 💡 Claude AI 根因研判摘要:                                     │
│ 經分析近 5 分鐘 VictoriaMetrics 數據，主要為 MySQL 連線池耗盡引發 │
│ 504 Gateway Timeout。比對 MkDocs Runbook，建議優先清理長事務。 │
├─────────────────────────────────────────────────────────────────┤
│ 📖 MkDocs SOP: [支付系統故障排查手冊](https://windcoldchen.github.io/windcold-kb/) │
│ 📊 Grafana:    [檢視支付系統監控盤](https://grafana.internal.windcold.com/)   │
├─────────────────────────────────────────────────────────────────┤
│ [ ⚡ 一鍵重啟 Pod ]   [ 🛡️ 切換降級開關 ]   [ 📞 GoAlert 認領 ] │
└─────────────────────────────────────────────────────────────────┘
```

### B. Adaptive Card JSON 完整定義檔

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.5",
  "msteams": {
    "width": "Full"
  },
  "body": [
    {
      "type": "Container",
      "style": "attention",
      "bleed": true,
      "items": [
        {
          "type": "TextBlock",
          "text": "🚨 系統告警：InHouseApiHighErrorRate",
          "weight": "Bolder",
          "size": "Medium",
          "color": "Attention"
        },
        {
          "type": "TextBlock",
          "text": "服務名稱: payment-gateway-api | 環境: Production",
          "isSubtle": true,
          "spacing": "None"
        }
      ]
    },
    {
      "type": "Container",
      "spacing": "Medium",
      "items": [
        {
          "type": "TextBlock",
          "text": "🤖 Claude AI 根因診斷與修復建議",
          "weight": "Bolder",
          "size": "Default",
          "color": "Accent"
        },
        {
          "type": "TextBlock",
          "text": "比對 VictoriaMetrics 指標趨勢與 MkDocs 知識庫，發現資料庫連線池使用率達 100%，導致下游訂單連鎖超時。建議立即執行臨時連線池動態擴容，並隔離慢查詢交易。",
          "wrap": true
        }
      ]
    },
    {
      "type": "FactSet",
      "facts": [
        { "title": "目前錯誤率", "value": "12.8% (閾值 5%)" },
        { "title": "持續時間", "value": "2 分 45 秒" },
        { "title": "On-Call 值班", "value": "Luke Chen (GoAlert 階梯一)" }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "📖 MkDocs 知識庫 SOP",
      "url": "https://windcoldchen.github.io/windcold-kb/notes/workflow/"
    },
    {
      "type": "Action.OpenUrl",
      "title": "📊 Grafana 即時監控盤",
      "url": "https://grafana.internal.windcold.com/"
    },
    {
      "type": "Action.Execute",
      "title": "⚡ 呼叫自研 API 一鍵擴容/自癒",
      "verb": "trigger_remediation",
      "data": {
        "action": "scale_db_pool",
        "service": "payment-gateway-api",
        "environment": "production"
      }
    }
  ]
}
```

---

## 6. 自研系統雙向整合架構

### A. 自研系統暴露與推送 Metrics

自研系統透過官方 Prometheus SDK 或定時工作推送指標至 VictoriaMetrics：

```python
# 自研系統中介軟體 (Python 範例)
from prometheus_client import Counter, Histogram

REQUEST_COUNT = Counter(
    "inhouse_api_requests_total",
    "自研系統業務請求計數",
    ["service", "method", "endpoint", "status"]
)

LATENCY_HISTOGRAM = Histogram(
    "inhouse_api_latency_seconds",
    "自研系統處理延遲分布",
    ["service", "endpoint"]
)
```

### B. 自研自癒 Controller (Webhook Receiver)

當值班者在 Teams 卡片點選「一鍵自癒」動作時，Teams 後台會發起安全 HTTP POST 請求至自研維運 API：

```python
# /app/remediation_controller.py
from fastapi import FastAPI, Header, HTTPException, Depends
from pydantic import BaseModel
import hmac
import hashlib

app = FastAPI()
WEBHOOK_SECRET = "IN_HOUSE_SECURE_HMAC_SECRET"

class RemediationPayload(BaseModel):
    action: str
    service: str
    environment: str

def verify_teams_signature(x_signature: str = Header(None)):
    if not x_signature:
        raise HTTPException(status_code=401, detail="缺少簽章標頭")
    # 進行 HMAC SHA256 驗證
    # ...

@app.post("/v1/remediation/execute", dependencies=[Depends(verify_teams_signature)])
async def execute_remediation(payload: RemediationPayload):
    if payload.action == "scale_db_pool":
        # 呼叫 K8s 或雲端資料庫 API 調配連線池
        print(f"執行自癒: 動態擴大 {payload.service} 連線池")
        return {"status": "success", "message": "連線池已成功擴大至 200"}
    elif payload.action == "restart_pod":
        # 執行 Pod 滾動重啟
        print(f"執行自癒: 重啟服務 {payload.service}")
        return {"status": "success", "message": "已觸發滾動重啟"}
    
    raise HTTPException(status_code=400, detail="不支援的自癒動作")
```

---

## 7. 最佳實踐與資安架構守則

### A. 資安與憑證管控

1. **Webhook 簽章驗證**：Teams 卡片回呼自研系統必須嚴格實施 HMAC SHA-256 簽署機制，杜絕偽造請求。
2. **敏感金鑰統一管理**：所有 API Key（Claude、Keep、GoAlert Token）應儲存於 HashiCorp Vault、AWS Secrets Manager 或 Kubernetes Secrets 中，禁止硬編碼。
3. **MkDocs 知識庫私密性**：若知識庫包含內部架構機密，應透過 [Cloudflare Pages + Access (Entra ID)](file:///Users/luke.chen/Documents/github/windcold-kb/docs/notes/cloudflare-pages-entra-id.md) 施加零信任防護，Claude 查詢時使用專用 Service Token。

### B. 告警治理與防風暴機制 (Anti-Alert Fatigue)

1. **智慧降噪三道防線**：
   - **第一道 (vmalert)**：使用 `for: 3m` 避免瞬間抖動 (Flapping)。
   - **第二道 (Alertmanager)**：使用 `group_by` 與 `inhibit_rules`，底層網路或機房中斷時自動抑制數百個上層 API 告警。
   - **第三道 (KeepHQ)**：藉由時間窗口與拓撲關聯分析，聚合多源告警為單一 Incident。
2. **知識庫 (Runbooks) 維護規範**：
   - 每條告警規則的 `annotations.runbook_url` 必須強制通過 CI 檢查，確保不是無效連結 (Broken Link)。
   - 當架構重構時，MkDocs 知識庫內的故障排查指引需同步在 Pull Request 中修訂。

---

## 8. 總結架構矩陣

```text
┌────────────────────────────────────────────────────────────────────────┐
│                   新世代 AIOps 可觀測性閉環管理架構                      │
├────────────────────┬───────────────────────────────────────────────────┤
│ 時序數據與指標儲存 │ VictoriaMetrics (高效能、超低資源、MetricsQL)       │
│ 視覺化監控面板     │ Grafana (即時儀表板與日誌聯動)                     │
│ 告警收斂與路由     │ Alertmanager ➔ KeepHQ (多源彙整與拓撲關聯)         │
│ AI 智慧大腦       │ Anthropic Claude 3.5 (根因假設分析與處置指引)       │
│ 團隊智慧技術中樞   │ MkDocs 知識庫 (透過 MCP 實現 SOP 即時自動檢索)     │
│ 階梯式 On-Call 輪值│ GoAlert (開源無許可證費用、支援電話/簡訊/語音升級)   │
│ 個人行動推播強提醒 │ Pushover (支援 iOS Critical Alerts 穿透勿擾模式)    │
│ ChatOps 協同平台   │ Microsoft Teams (互動式 Adaptive Cards)           │
│ 維運自癒執行端     │ 自研系統 (Prometheus Metrics + 安全 Webhook API)  │
└────────────────────┴───────────────────────────────────────────────────┘
```
