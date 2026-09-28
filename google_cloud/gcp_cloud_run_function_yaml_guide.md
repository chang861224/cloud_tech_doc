# 🚀 GCP Cloud Run 函式 (Cloud Run Functions) YAML 部署完整指南

本文件旨在詳細介紹如何在 Google Cloud Platform (GCP) 上透過宣告式的 YAML 設定檔來部署與管理 **Cloud Run 函式 (Cloud Run Functions，即第 2 代 Functions)**。

---

## 一、 🚀 簡介與使用時機

Cloud Run Functions (第 2 代) 底層實際上是直接運行在 **Cloud Run** 之上的服務，並搭配 **Eventarc** 觸發器來處理事件或 HTTP 請求。因此，我們完全可以使用 Cloud Run 的宣告式 YAML 格式，結合對應的 annotations 與結構來進行部署。

### 為什麼選擇 YAML 部署 Cloud Run Functions？
1. **版本控制**：與基礎設施即程式碼 (IaC) 或 GitOps 完美整合，設定檔可放入 Git 進行版控。
2. **自動化 CI/CD**：在 Pipelines 中透過 `gcloud` 指令直接套用 YAML 設定，省去繁瑣的指令參數。
3. **一致性與透明度**：清楚掌握函式的記憶體、逾時時間、環境變數及對外觸發來源。

---

## 二、 📁 完整 YAML 範本解析

以下是一個標準的 Cloud Run 函式 (第 2 代) 部署 YAML 範本 (`function-service.yaml`)：

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-cloud-run-function
  labels:
    cloud.googleapis.com/location: asia-east1
  annotations:
    # 允許公開存取 HTTP 觸發器 (若只需內部呼叫可改為 internal)
    run.googleapis.com/ingress: all
    run.googleapis.com/ingress-status: all
spec:
  template:
    metadata:
      annotations:
        # 指定第 2 代執行環境
        run.googleapis.com/execution-environment: gen2
        # 設定 Cloud Run Functions 的進入點函式名稱 (Function Target)
        run.googleapis.com/function-target: helloHttp
        # 自動擴展設定
        autoscaling.knative.dev/minScale: "0"
        autoscaling.knative.dev/maxScale: "10"
    spec:
      containerConcurrency: 1
      timeoutSeconds: 60
      serviceAccountName: your-service-account@your-project-id.iam.gserviceaccount.com
      containers:
        - image: us-central1-docker.pkg.dev/your-project-id/gcf-artifacts/my-cloud-run-function:latest
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: "1"
              memory: "512Mi"
          env:
            - name: LOG_LEVEL
              value: "info"
            - name: TARGET_ENV
              value: "production"
```

### 🔑 關鍵欄位與 Cloud Run Functions 專屬 Annotations 說明
- **`run.googleapis.com/function-target`**：這是 Cloud Run Functions 專屬的關鍵註解，用來指定您的程式碼中要被呼叫的函式名稱（例如 Node.js 的 `helloHttp` 或 Python 的 `hello_http`）。
- **`run.googleapis.com/execution-environment: gen2`**：Cloud Run Functions 第 2 代必須啟用 `gen2` 執行環境。
- **`containerConcurrency`**：函式通常建議設定為 `1`（除非您的函式支援安全的同時多執行緒處理），確保每個執行個體一次只處理一個請求。
- **`image`**：Cloud Run Functions 封裝後上傳至 Artifact Registry (`gcf-artifacts`) 的容器映像檔。

---

## 三、 🛠️ 部署與管理步驟

### 步驟 1：準備並上傳原始碼或建立映像檔
雖然可以直接由原始碼透過 `gcloud functions deploy` 部署，但若採用 YAML 宣告式部署，通常會搭配 Cloud Build 或自行建置容器映像檔推送到 Artifact Registry。

您可以透過以下指令先透過原始碼建立好函式與映像檔，再匯出 YAML：
```bash
gcloud functions deploy my-cloud-run-function \
    --gen2 \
    --runtime=nodejs20 \
    --region=asia-east1 \
    --source=. \
    --entry-point=helloHttp \
    --trigger-http \
    --allow-unauthenticated
```

### 步驟 2：套用 YAML 進行更新與部署
當映像檔與 YAML 準備好後，即可直接透過 `gcloud` 進行部署：
```bash
gcloud run services replace function-service.yaml --region=asia-east1
```
或者使用 `apply` 指令：
```bash
gcloud run services apply --file=function-service.yaml --region=asia-east1
```

---

## 四、 💡 最佳實務與注意事項

1. **第 2 代架構底層即 Cloud Run**：Cloud Run Functions 第 2 代與 Cloud Run Service 共享幾乎完全相同的設定模型，這讓您可以享有 Cloud Run 強大的流量管理、安全性與擴展能力。
2. **事件觸發器 (Eventarc)**：若您的函式不是 HTTP 觸發（例如 Cloud Storage、Pub/Sub 觸發），底層會自動透過 Eventarc 產生對應的訂閱與事件路由，建議搭配 Terraform 或 gcloud 進行事件源的綁定。
3. **資源與逾時調整**：根據函式的工作負載適當調整 `timeoutSeconds`（最高可達 60 分鐘）與記憶體配置。
