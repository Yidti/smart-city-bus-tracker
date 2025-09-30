# 專案計畫：智慧城市公車即時追蹤系統 (本地開發方案)

## 1. 專案核心目標

本專案旨在**在本機環境中**，建立一個模擬雲原生技術的即時數據平台。系統將從政府公開 API 獲取公車即時動態，透過串流處理引擎進行分析，實現「公車到站提醒」功能，並將所有服務部署在**本地的 Kubernetes (Minikube) 環境**上，搭配完整的 CI/CD 與監控流程。**此方案 100% 免費。**

## 2. 核心技術棧

- **本地環境:** **Docker Desktop**, **Minikube**, **Docker Compose**
- **容器化與編排:** Docker, Kubernetes (K8s), Helm
- **數據串流與工作流:** Kafka, Spark Streaming, Airflow
- **資料庫:** MongoDB, Elasticsearch
- **應用程式:** Python, PySpark
- **監控與日誌:** Prometheus, Grafana, ELK Stack
- **自動化與 IaC (學習):** GitHub Actions, Terraform

---

## 4. 開發階段 (Phases)

### Phase 1: 本地環境建置 (Local Environment Setup)

- [ ] **任務 1.1: 安裝本地開發核心工具**
    - **技術細節:**
        - **Docker Desktop:** 提供本機運行的 Docker 引擎，是運行所有容器化應用的基礎。
        - **Minikube:** 在 Docker 之上，建立一個單節點的 Kubernetes 叢集，讓我們可以模擬雲端 K8s 環境。
        - **kubectl:** Kubernetes 的命令列管理工具，用於與 Minikube 叢集互動。
        - **Helm:** Kubernetes 的套件管理器，簡化應用的部署。
    - **實作思路:**
        1.  根據您的作業系統，安裝 Docker Desktop。
        2.  安裝 kubectl CLI。
        3.  安裝 Minikube CLI。
        4.  安裝 Helm CLI。

- [ ] **任務 1.2: 使用 Docker Compose 建立核心數據服務**
    - **技術細節:**
        - **Docker Compose:** 用於定義和運行多容器 Docker 應用程式的工具。透過一個 `docker-compose.yml` 檔案，我們可以一鍵啟動所有後台基礎服務。
        - **為什麼用 Docker Compose？** 對於 Kafka、Airflow、MongoDB 這類有狀態的複雜服務，使用 Docker Compose 在本地管理比直接在 Minikube 中部署更簡單、更快速，也能將「基礎服務」和我們自己開發的「應用服務」進行有效隔離。
    - **實作思路:**
        1.  在專案根目錄建立一個 `docker-compose.yml` 檔案。
        2.  在檔案中，分別定義 `services` 給 Zookeeper, Kafka, Airflow (包含 webserver, scheduler, database), MongoDB, Prometheus, Grafana, ELK Stack。
        3.  為每個服務配置好映像檔版本、端口映射 (ports) 和儲存卷 (volumes)。
        4.  執行 `docker-compose up -d` 在背景啟動所有服務。

- [ ] **任務 1.3: 啟動並設定 Minikube**
    - **技術細節:**
        - Minikube 會在您的 Docker 中建立一個名為 `minikube` 的容器，這個容器就是您的 K8s 叢集。
    - **實作思路:**
        1.  執行 `minikube start --driver=docker` 來啟動您的本地 K8s 叢集。
        2.  執行 `minikube addons enable ingress` 來啟用 Ingress 控制器，方便後續管理服務的對外連線。
        3.  執行 `kubectl get nodes` 來確認您的單節點叢集處於 `Ready` 狀態。

- [ ] **任務 1.4 (可選，純學習): 撰寫 Terraform 腳本**
    - **技術細節:**
        - **目標:** 學習 Infrastructure as Code (IaC) 的概念與 Terraform 語法，為未來上雲做準備。
        - **注意:** 這個任務只包含程式碼的撰寫與規劃 (`terraform plan`)，**不會**執行 `terraform apply`，因此不會產生任何費用。
    - **實作思路:**
        1.  建立一個 `terraform` 資料夾。
        2.  在其中撰寫 `.tf` 檔案，定義如何在雲端平台 (GCP/AWS) 建立一個託管的 K8s 叢集。
        3.  執行 `terraform init` 和 `terraform plan` 來驗證語法並預覽執行計畫。

### Phase 2: 數據獲取與處理

- [ ] **任務 2.1: 使用 Airflow 建立排程，定時執行 `producer.py`**
    - **實作思路:**
        1.  建立 `dags` 和 `scripts` 資料夾。
        2.  在 `docker-compose.yml` 中，將這兩個資料夾分別掛載到 Airflow 容器的對應路徑下。
        3.  在 `dags` 中撰寫 `bus_producer_dag.py`，使用 `BashOperator` 來執行位於 `scripts` 資料夾中的 `producer.py`。
        4.  `producer.py` 腳本負責呼叫 API，並將數據發送到 Docker Compose 中運行的 Kafka 服務 (`kafka:9092`)。

- [ ] **任務 2.2: 撰寫 PySpark Streaming 應用 `streaming_processor.py`**
    - **實作思路:** (邏輯不變，但需注意 Spark 連接的 Kafka 地址是 Docker Compose 中的服務名，例如 `kafka:9092`)

- [ ] **任務 2.3:** 將 PySpark 應用打包成 Docker Image，並使用 `spark-submit` 將其部署到 **Minikube** 叢集上運行。
    - **實作思路:**
        1.  撰寫 `Dockerfile` 將 PySpark 應用打包。
        2.  執行 `eval $(minikube -p minikube docker-env)`，讓本地 Docker CLI 指向 Minikube 內部的 Docker daemon。
        3.  執行 `docker build`，建置好的映像檔會直接存在於 Minikube 內部，供 K8s 使用。
        4.  修改 `spark-submit` 命令，將 `--master` 指向 Minikube 的 K8s API Server 地址 (可透過 `kubectl config view` 獲取)。

### Phase 3: 下游應用與儲存

- [ ] **任務 3.1:** 將 `alerting_consumer.py` 打包成 Docker Image，並作為 K8s `Deployment` 部署到 **Minikube**。
    - **實作思路:**
        1.  撰寫 `Dockerfile` 和 `deployment.yaml`。
        2.  同樣使用 `eval $(minikube docker-env)` 和 `docker build` 來建置映像檔。
        3.  使用 `kubectl apply -f deployment.yaml` 將其部署到 Minikube。
        4.  `alerting_consumer.py` 腳本會連接到 Docker Compose 中的 Kafka (`kafka:9092`) 和 MongoDB (`mongodb:27017`)。

- [ ] **任務 3.2:** 將到站警示存入 MongoDB 以供應用查詢。
    - (此任務已合併到 3.1 的 `alerting_consumer.py` 中實現)

### Phase 4: 維運、監控與自動化

(此階段所有設定與監控的對象，都將是本地 Docker Compose 和 Minikube 中的服務)

- [ ] **任務 4.1:** 設定 Prometheus 監控 Minikube 和 Docker Compose 中的服務。
- [ ] **任務 4.2:** 在本地 Grafana 中建立儀表板。
- [ ] **任務 4.3:** 在 GitHub Actions 中建立 CI 流程 (持續整合)，自動化測試與建置 Docker 映像檔。
    - **技術細節:** CD (持續部署) 到本地 Minikube 較為複雜，初期可專注於 CI 部分。
