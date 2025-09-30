
# 專案計畫：智慧城市公車即時追蹤與到站提醒系統

## 1. 專案核心目標

本專案旨在建立一個基於雲原生技術的即時數據平台。系統將從政府公開 API 獲取公車即時動態，透過串流處理引擎進行分析，實現「公車到站提醒」功能，並將所有服務部署在 Kubernetes 上，搭配完整的 CI/CD 與監控流程。

## 2. 核心技術棧

- **雲端與 IaC:** GCP/AWS, Terraform
- **容器化與編排:** Docker, Kubernetes (K8s), Helm
- **數據串流與工作流:** Kafka, Spark Streaming, **Airflow**
- **資料庫:** **MongoDB**, Elasticsearch
- **應用程式:** Python, PySpark
- **監控與日誌:** Prometheus, Grafana, ELK Stack
- **自動化:** GitHub Actions

---

## 3. 開發方法論建議 (Recommended Development Methodology)

我們建議採用一個混合策略，結合不同方法論的優點，來應對專案中不同性質的任務。

- **領域驅動設計 (DDD) - 用於宏觀設計:**
  在動手寫程式碼前，先用 DDD 的思維來劃分系統的核心概念，如 `Bus`, `Stop`, `Route`, `GeoFence`, `Alert`。這有助於建立清晰、可擴充的系統架構。

- **測試驅動開發 (TDD) - 用於核心邏輯:**
  針對核心演算法（如地理圍籬計算），嚴格遵循「紅燈-綠燈-重構」的循環。先為你的計算函式編寫一個失敗的單元測試，再撰寫能讓測試通過的最小化程式碼，最後在測試的保護下進行重構。這能確保系統最關鍵部分的品質。

- **持續整合與 GitOps (CI/CD & GitOps) - 用於基礎設施與部署:**
  將所有東西（應用程式碼、Terraform 腳本、K8s YAMLs）都視為程式碼並存放在 Git 中。所有變更都透過 Pull Request 進行審查，並由 CI/CD 管線自動化測試與部署。這提供了完整的變更追蹤、自動化與系統穩定性。

---

## 4. 開發階段 (Phases)

### Phase 1: 基礎設施與數據源 (Infrastructure & Data Source)

- [ ] **任務 1.1:** 使用 Terraform 撰寫腳本，在雲端平台 (GCP/AWS) 建立一個託管的 Kubernetes 叢集 (GKE/EKS)。
    - **技術細節:**
        - **Terraform 是什麼？** Terraform 是一個「基礎設施即程式碼」(Infrastructure as Code, IaC) 工具。它讓你能用宣告式的設定檔（HCL 語言）來定義、管理你的雲端資源（例如虛擬機、網路、資料庫）。
        - **為什麼用 Terraform？**
            - **可追蹤性與版本控制:** 所有基礎設施的變更都像程式碼一樣，可以被 Git 管理、審查 (Code Review) 和追蹤。
            - **可重複性:** 同一份腳本可以在不同環境（開發、測試、生產）中重複部署，確保環境一致性。
            - **自動化:** 搭配 CI/CD 工具，可以實現基礎設施變更的自動化部署。
        - **GKE/EKS 是什麼？**
            - **GKE (Google Kubernetes Engine)** 和 **EKS (Amazon Elastic Kubernetes Service)** 是兩大雲端供應商提供的「託管式 Kubernetes」服務。
            - **為什麼用託管式 K8s？** Kubernetes 的「控制平面」(Control Plane) 是管理整個叢集的大腦，非常複雜。使用託管服務，雲端供應商會幫你管理、維護、擴展和升級這個控制平面，讓你只需要專注在部署你的應用程式即可，大幅降低維運負擔。
    - **實作思路:**
        1.  安裝 Terraform CLI。
        2.  設定對應雲端平台 (GCP/AWS) 的 CLI 工具與權限。
        3.  建立一個 `main.tf` 檔案。
        4.  在檔案中定義 `provider` (例如 `google` 或 `aws`)。
        5.  使用 `google_container_cluster` (GCP) 或 `aws_eks_cluster` (AWS) 資源來定義你的 K8s 叢集規格（例如節點數量、機器類型、所在區域）。
        6.  執行 `terraform init` -> `terraform plan` -> `terraform apply` 來建立叢集。

- [ ] **任務 1.2:** 使用 Helm 在 K8s 叢集中部署一個高可用的 Kafka 叢集。
    - **技術細節:**
        - **Helm 是什麼？** Helm 是 Kubernetes 的套件管理器。如果說 K8s 是作業系統，那 Helm 就是 `apt` (Ubuntu) 或 `brew` (macOS)。它將部署一個應用所需的所有 K8s 資源（如 `Deployment`, `Service`, `ConfigMap` 等）打包成一個稱為 "Chart" 的集合。
        - **為什麼用 Helm？**
            - **簡化複雜部署:** 像 Kafka 這種有狀態的複雜應用，需要部署多個互相依賴的元件。使用 Helm Chart，你只需要一條命令 (`helm install`) 就能完成整個部署。
            - **可配置性:** Chart 提供了 `values.yaml` 檔案，讓你可以輕鬆客製化部署的各種參數（例如副本數、資源限制），而不用手動修改一堆 YAML 檔案。
            - **版本管理與回滾:** Helm 會記錄你每次部署的版本，如果新版本出問題，可以輕鬆回滾到上一個穩定版本。
        - **Kafka 是什麼？** Apache Kafka 是一個分散式、高吞吐量的「發布-訂閱」訊息系統，是處理即時數據流的事實標準。在這個專案中，它扮演著數據中樞的角色，接收來自 `producer` 的原始數據，並提供給下游的 `Spark Streaming` 進行處理。
    - **實作思路:**
        1.  安裝 Helm CLI。
        2.  從 Artifact Hub (Helm Chart 的中央倉庫) 尋找一個受信任的 Kafka Chart，例如 Bitnami 或 Confluent 提供的版本。
        3.  執行 `helm repo add` 來加入該 Chart 的倉庫。
        4.  客製化 `values.yaml` 檔案，例如設定 Kafka Broker 和 Zookeeper 的副本數以達到高可用。
        5.  執行 `helm install <release-name> <chart-name> -f values.yaml` 來部署 Kafka。

- [ ] **任務 1.3:** **使用 Airflow 建立排程，定時執行 `producer.py` 獲取公車即時數據。**
    - **技術細節:**
        - **Airflow 是什麼？** Airflow 是業界主流的開源工作流管理系統 (Workflow Management System)，用於以編程方式編寫、排程和監控工作流。它被職缺列為**必要技能**。
        - **為什麼用 Airflow (取代 CronJob)？**
            - **複雜依賴管理:** Airflow 使用有向無環圖 (DAG) 來定義任務，可以輕鬆處理複雜的任務依賴關係（例如，任務 B 必須在任務 A 成功後才能開始）。
            - **監控與管理:** 提供豐富的 Web UI，可以視覺化任務流程、查看日誌、手動觸發、監控任務狀態和重試失敗的任務，這對於維運至關重要。
            - **可擴展性:** 擁有大量的 Operators (如 `BashOperator`, `PythonOperator`, `DockerOperator`)，可以輕鬆與各種外部系統整合。
    - **實作思路:**
        1.  在本機或 K8s 上部署 Airflow (可使用官方 Helm Chart)。
        2.  建立一個 `dags` 資料夾。
        3.  在 `dags` 資料夾中，撰寫一個名為 `bus_data_producer_dag.py` 的 DAG 檔案。
        4.  在這個 DAG 中，使用 `PythonOperator` 或 `BashOperator` 來定義一個 task，該 task 負責執行 `producer.py` 腳本。
        5.  設定 DAG 的 `schedule_interval` 為 `timedelta(seconds=10)` 或 cron 表達式 `"*/10 * * * * *"`，使其每 10 秒執行一次。

- [ ] **任務 1.4:** 將 `producer.py` 獲取到的數據發送到 Kafka 的 `bus-locations` Topic。
    - **技術細節:**
        - **Kafka Topic 是什麼？** Topic 是 Kafka 中用來分類訊息的「主題」或「頻道」。Producer 將特定類型的訊息發布到一個 Topic，而 Consumer 則訂閱它們感興趣的 Topic 來接收訊息。
        - **為什麼命名為 `bus-locations`？**
            - **語意清晰:** 這個名稱清楚地表明了這個 Topic 裡的訊息是關於「公車位置」的原始數據。
            - **可擴展性:** 未來如果我們還想從其他來源獲取例如「天氣資訊」，我們可以建立一個新的 `weather-updates` Topic，而不會跟現有的數據流混在一起。
        - **數據格式:** 發送到 Kafka 的數據通常會被序列化。最常見的格式是 JSON，因為它人類可讀且大多數程式語言都支援。在 Python 中，你可以將一個字典（dictionary）轉換成 JSON 字串，然後再發送到 Kafka。
    - **實作思路:**
        1.  在 `producer.py` 中，當你從 PTX API 獲取到公車數據（通常是一個 Python 的 list of dictionaries）後。
        2.  遍歷這個 list，將每一筆公車的數據（一個 dictionary）使用 `json.dumps()` 轉換成 JSON 字串。
        3.  使用 Kafka Producer 的 `.send()` 方法，將這個 JSON 字串發送到名為 `bus-locations` 的 Topic。記得要將字串編碼成 `utf-8` 位元組。

### Phase 2: 即時處理核心 (Real-time Processing Core)

- [ ] **任務 2.1 & 2.2:** 撰寫 PySpark Streaming 應用 `streaming_processor.py`，從 `bus-locations` Topic 持續讀取數據流。
    - **技術細節:**
        - **Spark Structured Streaming 是什麼？** 它是建立在 Spark SQL 引擎之上的高階串流處理 API。它將即時數據流視為一張「不斷被附加數據的表」，讓你可以像操作靜態數據一樣，用 DataFrame/SQL 的方式來進行串流計算。
        - **Micro-batching (微批次) vs. True Streaming (真串流):** 傳統的 Spark Streaming (DStream) 和現在的 Structured Streaming 在底層都是以微批次的方式運作，即每隔一個極短的時間（例如 1 秒），將這段時間內到達的數據作為一個小批次 (mini-batch) 來處理。這提供了高吞吐量和與批次處理一致的語法，但會帶來秒級的延遲。
        - **從 Kafka 讀取:** Spark 提供了內建的 Kafka 連接器。你可以使用 `spark.readStream.format("kafka")...` 來建立一個串流 DataFrame，它會直接連接到 Kafka 的 `bus-locations` Topic。
    - **實作思路:**
        1.  建立 PySpark 專案結構，引入 `pyspark` 函式庫。
        2.  建立一個 `SparkSession`。
        3.  使用 `spark.readStream` 定義數據源，指向你的 Kafka Broker 地址和 `bus-locations` Topic。
        4.  你需要提供 Kafka 傳回的數據的 Schema (結構)，或者讓 Spark 自行推斷。明確定義 Schema 是更穩健的做法。
        5.  對讀進來的數據流 (DataFrame) 進行初步的轉換，例如從 Kafka value (binary) 中解析出 JSON 字串，並將其轉換成結構化的欄位。

- [ ] **任務 2.3:** 實現核心地理圍籬 (Geo-fencing) 演算法，即時計算公車與站牌的距離。
    - **技術細節:**
        - **地理圍籬是什麼？** Geo-fencing 是一種虛擬的地理邊界。在這個專案中，每個公車站牌周圍都可以定義一個半徑（例如 100 公尺）的圓形圍籬。當公車進入這個圍籬時，我們就觸發一個「即將到站」的事件。
        - **距離計算：Haversine 公式:** 這是計算地球上兩點（給定經緯度）之間距離最常用的公式之一。它考慮了地球的曲率，比簡單的歐幾里得距離更準確。你需要找到或自己實現一個 Haversine 函式。
        - **站牌數據:** 你需要一份包含所有公車站牌 ID、名稱和經緯度的靜態數據。這份數據可以儲存在一個檔案（如 CSV, Parquet）或資料庫中。在 Spark 中，一個高效的做法是將這份數據讀取成一個 DataFrame，然後使用 `broadcast` 函式將其廣播到所有 Spark Executor 節點。這樣在進行 `join` 操作時可以避免大量的數據 shuffle，提升效能。
    - **實作思路:**
        1.  準備站牌位置的靜態數據檔案。
        2.  在 Spark 應用中，將站牌數據讀取成一個 DataFrame，並進行廣播 (`spark.sparkContext.broadcast(...)`)。
        3.  將即時的公車位置數據流 DataFrame 與廣播的站牌 DataFrame 進行 `join` 或 `crossJoin`。
        4.  建立一個 Spark UDF (User-Defined Function)，該函式接收公車的經緯度和站牌的經緯度作為輸入，並使用 Haversine 公式計算距離。
        5.  使用這個 UDF 來建立一個新的 `distance` 欄位。

- [ ] **任務 2.4:** 實現數據分流邏輯，將一般動態發送到 `bus-updates` Topic，將觸發的到站警示發送到 `arrival-alerts` Topic。
    - **技術細節:**
        - **串流分發 (Stream Forking):** 在 Spark Structured Streaming 3.0 之後，你可以使用 `foreachBatch` 來對每個微批次的輸出進行更複雜的操作，包括將數據寫入多個不同的目的地。
        - **為什麼要分流？**
            - **關注點分離:** `arrival-alerts` Topic 的消費者只需要關心「到站」這個高價值事件，而不需要處理大量的公車移動數據。
            - **效能:** `bus-updates` Topic 可能會有巨大的流量，用於儀表板展示；而 `arrival-alerts` 流量小但重要性高，可以為其設定不同的保留策略或權限。
    - **實作思路:**
        1.  在計算出 `distance` 之後，使用 `withColumn` 和 `when`/`otherwise` 語法，新增一個 `is_arrival` 欄位 (布林值)，例如 `distance < 100`。
        2.  定義一個 `process_batch` 函式，它接收一個批次的 DataFrame 和 batch ID 作為參數。
        3.  在 `process_batch` 函式內部：
            - `alerts_df = df.filter("is_arrival = true")`
            - `updates_df = df` (或者可以過濾掉一些不必要的欄位)
            - 使用 `alerts_df.write.format("kafka")...` 將警示數據寫入 `arrival-alerts` Topic。
            - 使用 `updates_df.write.format("kafka")...` 將所有動態數據寫入 `bus-updates` Topic。
        4.  在你的主串流查詢上使用 `.writeStream.foreachBatch(process_batch).start()`。

- [ ] **任務 2.5:** 將 PySpark 應用打包成 Docker Image，並使用 `spark-submit` 將其部署到 K8s 叢集上運行。
    - **技術細節:**
        - **為什麼要 Docker 化？** 將應用程式及其所有依賴（包括 Python 版本、PySpark 和其他函式庫）打包到一個 Docker Image 中，可以確保在任何地方（你的本機、CI/CD 伺服器、K8s 叢集）的運行環境都完全一致。
        - **`spark-submit` on Kubernetes:** `spark-submit` 是提交 Spark 任務的標準工具。當你指定 `--master k8s://<k8s-api-server>` 時，Spark 會在 Kubernetes 叢集中動態地建立 Driver Pod 和 Executor Pods 來執行你的任務，任務結束後這些 Pods 會被自動銷毀。
    - **實作思路:**
        1.  **撰寫 `Dockerfile`:**
            -  選擇一個包含 Python 和 Spark 的基礎映像 (e.g., `bitnami/spark`)。
            -  `COPY` 你的 `streaming_processor.py` 和其他必要的檔案到映像中。
            -  使用 `pip install` 安裝你的 Python 依賴。
        2.  **建置並推送 Image:**
            -  `docker build -t <your-repo>/smart-bus-processor:v1 .`
            -  `docker push <your-repo>/smart-bus-processor:v1`
        3.  **撰寫 `spark-submit` 命令:**
            ```bash
            ./bin/spark-submit \
              --master k8s://https://<KUBERNETES_MASTER_IP>:6443 \
              --deploy-mode cluster \
              --name bus-processor \
              --conf spark.kubernetes.container.image=<your-repo>/smart-bus-processor:v1 \
              --conf spark.kubernetes.namespace=spark \
              --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
              local:///opt/spark/work-dir/streaming_processor.py
            ```
            *   `--deploy-mode cluster`: Driver 和 Executor 都在 K8s 叢集中運行。
            *   `local:///...`: 指向 Docker Image 內部的主應用程式檔案路徑。

### Phase 3: 下游應用與告警 (Downstream Applications & Alerts)

- [ ] **任務 3.1 & 3.2:** 撰寫輕量級 Python 應用 `alerting_consumer.py`，訂閱 `arrival-alerts` Topic，並發出通知。
    - **技術細節:**
        - **輕量級 Consumer:** 對於這個任務，我們不需要像 Spark 那樣重量級的處理框架。一個簡單的 Python 腳本，使用 `kafka-python` 或 `confluent-kafka` 函式庫，在一個迴圈中持續監聽 `arrival-alerts` Topic 即可。
        - **Slack Incoming Webhook:** 這是 Slack 提供的一種簡單的整合方式。你可以在 Slack 設定中為某個頻道建立一個 Webhook URL。任何向這個 URL 發送一個特定格式的 HTTP POST 請求（包含 JSON payload），其內容就會以訊息的形式出現在該頻道。
    - **實作思路:**
        1.  在 `alerting_consumer.py` 中，建立一個 KafkaConsumer，訂閱 `arrival-alerts` Topic。
        2.  `for message in consumer:` 迴圈會阻塞，直到有新訊息進來。
        3.  解析收到的訊息 (JSON)。
        4.  格式化訊息內容，例如 `f"公車 {bus_id} 即將抵達 {stop_name}！"`。
        5.  使用 `requests.post()` 函式，將格式化好的訊息作為 JSON payload，發送到你的 Slack Webhook URL。將 Webhook URL 儲存在環境變數中。

- [ ] **任務 3.3:** 將 `alerting_consumer.py` 打包成 Docker Image，並作為 K8s `Deployment` 部署。
    - **技術細節:**
        - **K8s Deployment 是什麼？** `Deployment` 是 K8s 中用來管理無狀態應用程式的標準資源。你只需要告訴 `Deployment`：「我希望有 N 個這個 Pod 的副本在運行」，它就會負責建立、監控、並在 Pod 掛掉時自動重啟它們，確保始終有 N 個副本可用。
        - **為什麼用 Deployment？** 我們的 `alerting_consumer.py` 是無狀態的（它不儲存任何數據，只是接收並轉發），且需要長時間在背景運行。`Deployment` 非常適合這種場景。
    - **實作思路:**
        1.  為 `alerting_consumer.py` 撰寫一個簡單的 `Dockerfile`。
        2.  建置並推送到 Image Registry。
        3.  撰寫一個 `deployment.yaml` 檔案：
            ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: alerting-consumer
            spec:
              replicas: 2 # 部署兩個副本以實現高可用
              selector:
                matchLabels:
                  app: alerting-consumer
              template:
                metadata:
                  labels:
                    app: alerting-consumer
                spec:
                  containers:
                  - name: consumer
                    image: <your-repo>/alerting-consumer:v1
                    env:
                    - name: SLACK_WEBHOOK_URL
                      valueFrom:
                        secretKeyRef:
                          name: slack-secret
                          key: webhook-url
            ```

- [ ] **任務 3.4:** 使用 Helm 部署 ELK Stack，並設定 Logstash/Filebeat 從 `bus-updates` Topic 收集數據，存入 Elasticsearch。
    - **技術細節:**
        - **ELK Stack 角色:**
            - **Elasticsearch:** 一個基於 Lucene 的分散式搜尋與分析引擎。用來儲存、索引大量的公車動態數據，並提供快速的查詢能力。
            - **Logstash/Filebeat:** 數據收集與轉換工具。在這裡，我們可以使用 Logstash 的 Kafka input plugin 來直接從 `bus-updates` Topic 拉取數據。
            - **Kibana:** 一個數據視覺化與探索平台。你可以用它來建立儀表板、查詢 Elasticsearch 中的數據（例如「查詢某輛公車過去一小時的軌跡」）。
    - **實作思路:**
        1.  使用 Helm Chart (例如 `elastic/eck-helm`) 來部署整個 ELK Stack。
        2.  設定 Logstash 的 pipeline 設定檔 (`logstash.conf`)：
            ```conf
            input {
              kafka {
                bootstrap_servers => "kafka-broker:9092"
                topics => ["bus-updates"]
                codec => "json"
              }
            }
            output {
              elasticsearch {
                hosts => ["http://elasticsearch-master:9200"]
                index => "bus-logs-%{+YYYY.MM.dd}" # 每天建立一個新的 index
              }
            }
            ```
        3.  將這個設定檔作為 K8s `ConfigMap` 掛載到 Logstash Pod 中。

- [ ] **任務 3.5 (新增):** **將到站警示存入 MongoDB 以供應用查詢。**
    - **技術細節:**
        - **MongoDB 是什麼？** 一個主流的 NoSQL、文件導向資料庫，被職缺明確要求。它以靈活的 JSON-like (BSON) 文件格式儲存數據，非常適合儲存結構可能變動的應用數據。
        - **為什麼要增加 MongoDB？**
            - **職能對應:** 直接對應職缺中對 NoSQL (MongoDB) 的技能要求。
            - **場景分離:** Elasticsearch/Kibana 強於日誌聚合與全文搜索，而 MongoDB 更適合做為後端應用的主資料庫，提供低延遲的單筆或小範圍查詢（例如，查詢特定使用者的設定、特定公車的最後位置等）。
    - **實作思路:**
        1.  使用 Helm 在 K8s 叢集中部署一個 MongoDB 實例。
        2.  修改 `alerting_consumer.py` (或建立一個新的 consumer `mongo_writer.py`)。
        3.  在 consumer 中，除了發送 Slack 通知外，增加使用 `pymongo` 函式庫連接到 MongoDB。
        4.  將收到的到站警示訊息（一個 JSON 物件）直接 `insert_one()` 到名為 `arrival_alerts` 的 collection 中。這展現了您處理數據並將其落地到 NoSQL 資料庫的能力。

### Phase 4: 維運、監控與自動化 (Operations, Monitoring & Automation)

- [ ] **任務 4.1:** 使用 Helm 部署 Prometheus Operator，並設定其監控 K8s、Kafka 和 Spark 應用的效能指標。
    - **技術細節:**
        - **Prometheus Operator:** 它引入了一系列 K8s CRDs (Custom Resource Definitions)，如 `ServiceMonitor` 和 `PodMonitor`。你不再需要手動管理 Prometheus 的抓取設定檔，而是用 K8s 的方式，為你的服務建立一個 `ServiceMonitor` 物件，Operator 就會自動讓 Prometheus 開始抓取該服務的指標。
        - **指標暴露 (Metrics Exposition):** 大多數雲原生應用都會在一個 `/metrics` HTTP 端點上，以 Prometheus 格式暴露自身的內部狀態指標。你需要為 Kafka 和 Spark 部署相應的 "Exporter"（例如 `JMX Exporter`）來將它們的 JMX 指標轉換成 Prometheus 格式。
    - **實作思路:**
        1.  使用 Helm Chart (`prometheus-community/kube-prometheus-stack`) 部署 Prometheus Operator 和 Grafana。
        2.  為 Kafka 部署 JMX Exporter，並建立一個 `ServiceMonitor` 物件，指向 Kafka Exporter 的 `/metrics` 端點。
        3.  設定 Spark 任務在啟動時也加載 JMX Exporter，並為 Spark Driver/Executors 建立相應的 `PodMonitor`。

- [ ] **任務 4.2:** 在 Grafana 中建立儀表板，視覺化系統關鍵指標。
    - **技術細節:**
        - **Grafana + Prometheus:** Grafana 原生支援 Prometheus 作為數據源。你可以在 Grafana 中使用 PromQL (Prometheus Query Language) 來查詢你關心的指標，並用各種圖表（折線圖、儀表盤、熱力圖等）進行視覺化。
    - **實作思路:**
        1.  在 Grafana 中設定 Prometheus 為數據源。
        2.  匯入社群提供的現成儀表板 (Dashboard) for Kubernetes, Kafka, Spark。
        3.  建立一個自訂的業務儀表板，顯示：
            - **端到端延遲:** 數據從 Producer 發出到被 Spark 處理完畢的時間差。
            - **Kafka Topic 訊息量:** `bus-locations` 的寫入速率 vs `arrival-alerts` 的寫入速率。
            - **活躍的 Consumer Group 數量與延遲 (Lag)。**

- [ ] **任務 4.3:** 在 GitHub Repo 中設定 GitHub Actions，建立完整的 CI/CD 流程。
    - **技術細節:**
        - **CI/CD (持續整合/持續部署):**
            - **CI (Continuous Integration):** 當開發者推送程式碼到 Git 時，自動觸發測試和建置 (Build)。
            - **CD (Continuous Deployment):** 當 CI 成功後，自動將新的應用程式版本部署到目標環境（例如 K8s）。
        - **GitHub Actions Workflow:** 你在專案的 `.github/workflows/` 目錄下建立 YAML 檔案來定義工作流程。
    - **實作思路:**
        1.  建立一個 `ci-cd.yaml` 工作流程檔案。
        2.  **定義觸發條件:** `on: push: branches: [ main ]`
        3.  **定義 Jobs:**
            - **`test` job:**
                - `runs-on: ubuntu-latest`
                - `steps:`
                    - `actions/checkout@v3`
                    - `actions/setup-python@v4`
                    - `run: pip install -r requirements.txt`
                    - `run: pytest`
            - **`build-and-push` job (needs: `test`):**
                - `steps:`
                    - `actions/checkout@v3`
                    - `docker/login-action@v2` (使用儲存在 GitHub Secrets 中的 Docker Hub 密碼)
                    - `docker/build-push-action@v4` (建置並推送 Docker Image)
            - **`deploy` job (needs: `build-and-push`):**
                - `steps:`
                    - `actions/checkout@v3`
                    - `actions/setup-kubectl@v1` (或 `helm`)
                    - 設定 K8s 連線憑證 (儲存在 GitHub Secrets 中)
                    - `run: kubectl apply -f k8s/deployment.yaml` 或 `helm upgrade ...`

---

## 5. 驗證計畫 (Verification Plan)

- [ ] **使用者視角:** 開發過程中，持續透過手機上的官方公車 APP，比對即時位置與到站提醒的準確性。
    - **實作思路:** 隨機挑選幾班正在行駛的公車，同時在你的 Slack 通知和官方 APP 上觀察，驗證到站提醒是否在合理的時間點（例如到站前 1-2 分鐘）觸發。

- [ ] **開發者視角:** 在 Kibana 中查詢告警日誌，並手動透過 Google Maps 驗證觸發告警當下的公車與站牌距離。
    - **實作思路:** 在 Kibana 中使用查詢語法 `index: bus-logs-* AND is_arrival: true` 來找到所有警示事件。從日誌中取出公車和站牌的經緯度，貼到 Google Maps 中，手動量測距離，確認是否在你的地理圍籬半徑內。

- [ ] **維運視角:** 在 Grafana 儀表板上監控端到端延遲指標，確保系統的即時性符合預期（例如 < 5 秒）。
    - **實作思路:** 在 Grafana 中建立一個圖表，其 PromQL 查詢為 `(spark_streaming_processing_time_ms / 1000)`。設定一個 Alertmanager 規則，當這個指標的 95 百分位數 (p95) 連續 5 分鐘超過 5 秒時，發送警報到 PagerDuty 或 Slack。

---

## 6. 超級加分項 (Stretch Goal)

- [ ] 建立一個簡單的 Flask Web 應用，透過 WebSocket 連接後端，訂閱 `bus-updates` Topic，並使用 Leaflet.js 或 Mapbox.js 在網頁地圖上即時呈現所有公車的動態軌跡。
    - **技術細節:**
        - **後端 (Flask):**
            - **Flask-SocketIO:** 這個 Flask 擴充套件極大地簡化了在 Flask 應用中處理 WebSocket 的複雜性。
            - **背景執行緒:** 你需要在 Flask 應用啟動時，在一個背景執行緒中啟動一個 Kafka Consumer 來訂閱 `bus-updates` Topic。如果直接在請求處理中啟動，會阻塞整個應用。
            - **訊息推送:** 當背景的 Kafka Consumer 收到新訊息時，它會使用 `socketio.emit()` 方法將數據推送給所有連接到 WebSocket 的前端客戶端。
        - **前端 (JavaScript):**
            - **Socket.IO Client:** 使用官方的 `socket.io-client` JavaScript 函式庫來連接後端的 WebSocket 服務。
            - **Leaflet.js/Mapbox.js:** 這是兩個流行的開源地圖視覺化函式庫。你可以使用它們來建立一個地圖，並在地圖上放置 Marker (標記) 來代表公車。
    - **實作思路:**
        1.  **後端:**
            - `pip install flask flask-socketio kafka-python`
            - 建立 Flask app 和 SocketIO 實例。
            - 建立一個繼承 `threading.Thread` 的類別，在 `run` 方法中實作 Kafka Consumer 的邏輯。當收到訊息時，呼叫 `socketio.emit('bus_update', message_data)`。
            - 在 Flask 啟動時，創建並啟動這個背景執行緒。
        2.  **前端:**
            - 在 HTML 中引入 Leaflet.js 和 Socket.IO client 的 CDN。
            - 建立一個 Leaflet 地圖實例。
            - `const socket = io();` 來連接後端。
            - `socket.on('bus_update', (data) => { ... });` 來監聽後端推送的訊息。
            - 在監聽函式中，根據收到的公車 ID 和經緯度，在地圖上找到對應的 Marker。如果 Marker 不存在，就建立一個新的；如果已存在，就更新它的位置 (`marker.setLatLng([lat, lon]);`)。
