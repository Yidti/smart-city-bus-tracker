# 智慧城市公車即時追蹤系統 (本地開發方案)

## 專案核心目標

本專案旨在**在本機環境中**，建立一個模擬雲原生技術的即時數據平台。系統將從政府公開 API 獲取公車即時動態，透過串流處理引擎進行分析，實現「公車到站提醒」功能，並將所有服務部署在**本地的 Kubernetes (Minikube) 環境**上。**此方案 100% 免費。**

## 核心技術棧

- **本地環境:** **Docker Desktop**, **Minikube**, **Docker Compose**
- **容器化與編排:** Docker, Kubernetes (K8s), Helm
- **數據串流與工作流:** Kafka, Spark Streaming, Airflow
- **資料庫:** MongoDB, Elasticsearch
- **應用程式:** Python, PySpark
- **監控與日誌:** Prometheus, Grafana, ELK Stack
- **自動化與 IaC (學習):** GitHub Actions, Terraform

## 開發階段 (Phases)

### Phase 1: 本地環境建置 (Local Environment Setup)

- [ ] **任務 1.1:** 安裝本地開發核心工具 (Docker, Minikube, kubectl, Helm)。
- [ ] **任務 1.2:** 使用 Docker Compose 建立核心數據服務 (Kafka, Airflow, MongoDB 等)。
- [ ] **任務 1.3:** 啟動並設定 Minikube 叢集。
- [ ] **任務 1.4 (可選):** 撰寫 Terraform 腳本 (純學習，不執行)。

### Phase 2: 數據獲取與處理

- [ ] **任務 2.1:** 使用 Airflow 建立排程，定時執行 `producer.py` 獲取數據。
- [ ] **任務 2.2:** 撰寫 PySpark Streaming 應用 `streaming_processor.py` 進行即時運算。
- [ ] **任務 2.3:** 將 PySpark 應用打包並部署到 Minikube 叢集。

### Phase 3: 下游應用與儲存

- [ ] **任務 3.1:** 撰寫 Python 消費者應用 `alerting_consumer.py`。
- [ ] **任務 3.2:** 將警示訊息存入 MongoDB。
- [ ] **任務 3.3:** 將消費者應用打包並部署到 Minikube。

### Phase 4: 維運、監控與自動化

- [ ] **任務 4.1:** 設定 Prometheus 監控本地服務。
- [ ] **任務 4.2:** 在 Grafana 中建立儀表板。
- [ ] **任務 4.3:** 在 GitHub Actions 中建立 CI (持續整合) 流程。
