# 🚀 GCP Cloud Run 服務 (Service) YAML 部署完整指南

本文件旨在詳細介紹如何在 Google Cloud Platform (GCP) 上透過宣告式的 YAML 設定檔來部署與管理 **Cloud Run 服務 (Cloud Run Service)**。

---

## 一、 🚀 簡介與使用時機

在 Cloud Run 中，您可以透過 `gcloud run services replace` 或 `gcloud run services apply` 指令直接套用 YAML 檔案來建立或更新服務。

### 為什麼選擇 YAML 部署服務？
1. **版本控制**：設定檔可以放入 Git 儲存庫進行版控與 Code Review。
2. **重現性**：確保多個環境（Staging、Production）之間的配置完全一致。
3. **進階設定**：支援圖形介面或簡單指令不易完整涵蓋的複雜細節（如多容器 Sidecar、環境變數與密碼管理、流量分配）。

---

## 二、 📁 完整 YAML 範本解析

以下是一個標準的 Cloud Run 服務 YAML 範本 (`service.yaml`)：

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-cloud-run-service
  labels:
    cloud.googleapis.com/location: asia-east1
  annotations:
    run.googleapis.com/ingress: all
    run.googleapis.com/ingress-status: all
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: "10"
        autoscaling.knative.dev/minScale: "0"
        run.googleapis.com/execution-environment: gen2
    spec:
      containerConcurrency: 80
      timeoutSeconds: 300
      serviceAccountName: your-service-account@your-project-id.iam.gserviceaccount.com
      containers:
        - image: gcr.io/your-project-id/my-app:v1.0.0
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: "1000m"
              memory: "512Mi"
          env:
            - name: NODE_ENV
              value: "production"
```

### 關鍵欄位說明
- **`apiVersion` / `kind`**：Cloud Run 服務基於 Knative Serving 規範，因此使用 `serving.knative.dev/v1`。
- **`metadata.name`**：您的 Cloud Run 服務名稱。
- **`autoscaling.knative.dev/maxScale` 與 `minScale`**：控制執行個體的自動擴展範圍（設定 `minScale: "0"` 可在無流量時縮減至零以節省成本）。
- **`resources.limits`**：設定每個容器執行個體的 CPU 與記憶體配額。
- **`containerConcurrency`**：每個容器執行個體允許的最大並發請求數。

---

## 三、 🛠️ 部署與更新步驟

### 步驟 1：準備 YAML 檔案
將上述內容儲存為 `service.yaml`，並替換為您的實際專案 ID 與映像檔網址。

### 步驟 2：登入 GCP 與設定專案
```bash
gcloud auth login
gcloud config set project your-project-id
```

### 步驟 3：部署或更新服務
```bash
gcloud run services replace service.yaml --region=asia-east1
```

---

## 四、 💡 最佳實務與注意事項

1. **最小權限原則 (Least Privilege IAM)**：為每個 Service 建立專屬的 Service Account。
2. **敏感資訊管理**：全面結合 GCP Secret Manager 透過 `secretKeyRef` 注入機密設定。
3. **健康檢查與探針**：適當設定啟動與存活探針，確保服務穩定。
