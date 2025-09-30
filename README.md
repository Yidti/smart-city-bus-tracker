# 智慧城市公車即時追蹤與到站提醒系統

## 專案核心目標

本專案旨在建立一個基於雲原生技術的即時數據平台。系統將從政府公開 API 獲取公車即時動態，透過串流處理引擎進行分析，實現「公車到站提醒」功能，並將所有服務部署在 Kubernetes 上，搭配完整的 CI/CD 與監控流程。

## 核心技術棧

- **雲端與 IaC:** GCP/AWS, Terraform
- **容器化與編排:** Docker, Kubernetes (K8s), Helm
- **數據串流與工作流:** Kafka, Spark Streaming, **Airflow**
- **資料庫:** **MongoDB**, Elasticsearch
- **應用程式:** Python, PySpark
- **監控與日誌:** Prometheus, Grafana, ELK Stack
- **自動化:** GitHub Actions

## 開發階段 (Phases)

### Phase 1: 基礎設施與數據源 (Infrastructure & Data Source)

- [ ] **任務 1.1:** 使用 Terraform 撰寫腳本，在雲端平台 (GCP/AWS) 建立一個託管的 Kubernetes 叢集 (GKE/EKS)。
- [ ] **任務 1.2:** 使用 Helm 在 K8s 叢集中部署一個高可用的 Kafka 叢集。
- [ ] **任務 1.3:** **使用 Airflow 建立排程**，定時執行 Python 應用 `producer.py` 獲取公車即時數據。
- [ ] **任務 1.4:** 將 `producer.py` 獲取到的數據發送到 Kafka 的 `bus-locations` Topic。

### Phase 2: 即時處理核心 (Real-time Processing Core)

- [ ] **任務 2.1:** 撰寫 PySpark Streaming 應用 `streaming_processor.py`。
- [ ] **任務 2.2:** 在應用中，實現從 `bus-locations` Topic 持續讀取數據流的邏輯。
- [ ] **任務 2.3:** 實現核心地理圍籬 (Geo-fencing) 演算法，即時計算公車與站牌的距離。
- [ ] **任務 2.4:** 實現數據分流邏輯，將一般動態發送到 `bus-updates` Topic，將觸發的到站警示發送到 `arrival-alerts` Topic。
- [ ] **任務 2.5:** 將 PySpark 應用打包成 Docker Image，並使用 `spark-submit` 將其部署到 K8s 叢集上運行。

### Phase 3: 下游應用與告警 (Downstream Applications & Alerts)

- [ ] **任務 3.1:** 撰寫輕量級 Python 應用 `alerting_consumer.py`，訂閱 `arrival-alerts` Topic。
- [ ] **任務 3.2:** 當 `alerting_consumer.py` 收到訊息時，發出一個格式化的通知 (到 Console 或 Slack)。
- [ ] **任務 3.3:** 將 `alerting_consumer.py` 打包成 Docker Image，並作為 K8s `Deployment` 部署。
- [ ] **任務 3.4:** 使用 Helm 部署 ELK Stack，並設定 Logstash/Filebeat 從 `bus-updates` Topic 收集數據，存入 Elasticsearch 以供歸檔查詢。
- [ ] **任務 3.5:** **將到站警示存入 MongoDB** 以供未來應用查詢。

### Phase 4: 維運、監控與自動化 (Operations, Monitoring & Automation)

- [ ] **任務 4.1:** 使用 Helm 部署 Prometheus Operator，並設定其監控 K8s、Kafka 和 Spark 應用的效能指標。
- [ ] **任務 4.2:** 在 Grafana 中建立儀表板，視覺化系統關鍵指標（如：端到端延遲、Kafka 訊息量、Spark 處理速率）。
- [ ] **任務 4.3:** 在 GitHub Repo 中設定 GitHub Actions，建立完整的 CI/CD 流程，自動化所有自訂應用程式的測試、建置與部署。

## 驗證計畫 (Verification Plan)

- [ ] **使用者視角:** 開發過程中，持續透過手機上的官方公車 APP，比對即時位置與到站提醒的準確性。
- [ ] **開發者視角:** 在 Kibana 中查詢告警日誌，並手動透過 Google Maps 驗證觸發告警當下的公車與站牌距離，確保地理圍籬演算法的正確性。
- [ ] **維運視角:** 在 Grafana 儀表板上監控端到端延遲指標，確保系統的即時性符合預期（例如 < 5 秒）。

## 超級加分項 (Stretch Goal)

- [ ] 建立一個簡單的 Flask Web 應用，透過 WebSocket 連接後端，訂閱 `bus-updates` Topic，並使用 Leaflet.js 或 Mapbox.js 在網頁地圖上即時呈現所有公車的動態軌跡。

## 開發方法論建議 (Recommended Development Methodology)

我們建議採用一個混合策略，結合不同方法論的優點，來應對專案中不同性質的任務。

- **領域驅動設計 (DDD) - 用於宏觀設計:**
  在動手寫程式碼前，先用 DDD 的思維來劃分系統的核心概念，如 `Bus`, `Stop`, `Route`, `GeoFence`, `Alert`。這有助於建立清晰、可擴充的系統架構。

- **測試驅動開發 (TDD) - 用於核心邏輯:**
  針對核心演算法（如地理圍籬計算），嚴格遵循「紅燈-綠燈-重構」的循環。先為你的計算函式編寫一個失敗的單元測試，再撰寫能讓測試通過的最小化程式碼，最後在測試的保護下進行重構。這能確保系統最關鍵部分的品質。

- **持續整合與 GitOps (CI/CD & GitOps) - 用於基礎設施與部署:**
  將所有東西（應用程式碼、Terraform 腳本、K8s YAMLs）都視為程式碼並存放在 Git 中。所有變更都透過 Pull Request 進行審查，並由 CI/CD 管線自動化測試與部署。這提供了完整的變更追蹤、自動化與系統穩定性。
