# 筆記範本與撰寫原則

在知識庫中維護高品質技術筆記的推薦架構與示範。

---

## 結構建議

一篇結構清晰的技術或學習筆記通常包含：

1. **背景與目標**：解決什麼問題？動機為何？
2. **核心概念 / 架構設計**：關鍵術語、架構圖或工作原理。
3. **實作步驟 / 程式碼片段**：具體重現方法或配置方式。
4. **踩坑紀錄與常見問題 (FAQ)**：碰到的錯誤與排除方法。
5. **參考資源**：官方文件或深度閱讀連結。

---

## 範例：Docker 容器健康檢查實踐

### 1. 背景

在微服務架構中，單純檢查進程是否存在並不足以判斷服務是否具備正常處理請求的能力。透過 `HEALTHCHECK` 機制，容器編排平台（如 Docker Swarm / Kubernetes / Compose）可以在服務異常時自動重啟或切換流量。

### 2. 配置方式

在 `Dockerfile` 中加入健康檢查邏輯：

```dockerfile linenums="1"
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

EXPOSE 3000

# 每 30 秒檢查一次，超時 3 秒判定失敗，連續失敗 3 次標記為 unhealthy
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/healthz || exit 1

CMD ["node", "server.js"]
```

### 3. 常見狀態說明

| 狀態 (State) | 意義 |
| :--- | :--- |
| `starting` | 容器剛啟動，處於初始寬限期 (`start-period`) |
| `healthy` | 檢查指令返回 exit code 0，運作正常 |
| `unhealthy` | 檢查指令連續失敗達到指定次數 (`retries`) |

!!! tip "健康檢查端點 (Healthz) 設計"
    端點應保持輕量快速，僅檢查資料庫連線、關鍵相依服務是否正常，避免進行昂貴運算。
