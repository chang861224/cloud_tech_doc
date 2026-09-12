# ⛓️ Terraform `depends_on` 完整使用指南與最佳實踐

> 💡 **簡短摘要**：本指南將深入探討 Terraform 中的資源相依性管理，詳細說明隱式相依與顯式相依的差異，並透過 GCP 的實戰範例示範如何正確使用 `depends_on` 來控制資源建立順序、避免部署失敗，以及掌握最佳實踐。

---

## 🎯 前置準備 (Prerequisites)

在開始學習與使用 `depends_on` 之前，請確保您的環境與知識已符合以下條件：
- **必備工具**：安裝 [Terraform CLI](https://www.terraform.io/) (v1.5+) 與 [Google Cloud SDK (`gcloud`)](https://cloud.google.com/sdk)。
- **先備知識**：已掌握 Terraform 基礎語法（如 Provider、Resource 的宣告方式），並具備 GCP 雲端平台的基礎觀念。
- **GCP 專案權限**：具有在 GCP 專案中啟用 API、指派 IAM 角色與建立 Cloud Run 服務的權限。

---

## 🗺️ 資源相依性原理 (Understanding Dependencies)

Terraform 是一個基於「宣告式 (Declarative)」的基礎架構即代碼 (IaC) 工具。當您執行 `terraform apply` 時，Terraform 會自動解析資源之間的關聯，並建構出一個**有向無環圖 (Directed Acyclic Graph, DAG)**，以此決定並行建立或更新資源的最佳順序。

資源之間的相依性可以分為兩種類型：

### 1. 隱式相依 (Implicit Dependency)
這是 Terraform 最推薦且最常用的相依方式。當一個資源的參數直接引用了另一個資源的屬性時，Terraform 就會自動推導出相依性，確保被引用的資源先被建立。

例如：
```terraform
# 宣告一個 GCP 虛擬網路 (VPC)
resource "google_compute_network" "vpc_network" {
  name                    = "my-custom-vpc"
  auto_create_subnetworks = false
}

# 宣告子網路，並引用 VPC 的 ID
resource "google_compute_subnetwork" "subnet" {
  name          = "my-custom-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = "asia-east1"
  # 隱式相依：這裡直接引用了 google_compute_network.vpc_network 的 id
  network       = google_compute_network.vpc_network.id
}
```
在這個範例中，Terraform 知道必須先建立 `vpc_network`，才能拿到它的 `id` 去建立 `subnet`。

### 2. 顯式相依 (Explicit Dependency) 與 `depends_on`
然而，在某些特殊場景下，資源之間存在**邏輯上的先後順序**，但彼此在程式碼中**沒有直接的參數引用關係**。此時 Terraform 的依賴圖無法自動偵測到這種相依，就需要使用 `depends_on` 元引數 (Meta-argument) 來手動告知 Terraform 順序。

```mermaid
graph TD
    subgraph 隱式相依 (Implicit)
        A[google_compute_network] -->|自動偵測| B[google_compute_subnetwork]
    end
    subgraph 顯式相依 (Explicit)
        C[google_project_service] -->|depends_on 手動指定| D[google_cloud_run_v2_service]
    end
```

---

## 🚀 什麼是 `depends_on` 與語法 (Syntax & Usage)

`depends_on` 是 Terraform 的保留字，可以用於任何 `resource` 或 `module` 區塊中。它接受一個由資源參考組成的**列表 (List)**。

### 基本語法格式

```terraform
resource "resource_type" "resource_name" {
  # 資源參數設定
  
  # 手動指定此資源必須在以下資源建立完成後才能開始建立
  depends_on = [
    resource_type.dependency_name,
    module.some_module
  ]
}
```

### 何時必須使用 `depends_on`？
最常見的顯式相依場景包括：
1. **API/服務啟用**：在 GCP 或 AWS 上，必須先啟用某個服務的 API（如 `run.googleapis.com`），才能在該專案中建立該服務的資源（如 Cloud Run）。
2. **IAM 權限生效**：需要先指派 IAM 角色給 Service Account（例如授予 Artifact Registry 讀取權限），然後部署需要下載該映像檔的運算資源。
3. **應用程式層級相依**：例如 Kubernetes 叢集必須先完全建置就緒，才能開始部屬 Helm Chart。

---

## 💻 實戰教學 (Step-by-Step Guide)

我們將以 GCP 為例，示範如何先啟用 **Cloud Run API**，隨後再安全地建立一個 **Cloud Run Service**。

### 步驟 1：撰寫 Terraform 設定檔

建立一個 `main.tf`，在建立 Cloud Run 服務時，使用 `depends_on` 顯式地宣告它必須等待 API 啟用完成。

```terraform
# 宣告 GCP Provider
provider "google" {
  project = "your-gcp-project-id" # 請替換為您的 GCP Project ID
  region  = "asia-east1"
}

# 1. 啟用 Cloud Run API
resource "google_project_service" "run_api" {
  service                    = "run.googleapis.com"
  disable_dependent_services = true
  disable_on_destroy         = false
}

# 2. 部署 Cloud Run 服務
resource "google_cloud_run_v2_service" "hello_service" {
  name     = "hello-run-service"
  location = "asia-east1"
  ingress  = "INGRESS_TRAFFIC_ALL"

  template {
    containers {
      image = "us-docker.pkg.dev/cloudrun/container/hello:latest"
    }
  }

  # 顯式相依：由於 Cloud Run 服務的參數中沒有直接引用 run_api 的任何輸出欄位，
  # 必須使用 depends_on 強制 Terraform 先執行並完成 API 的啟用，否則會因 API 未啟用而報錯。
  depends_on = [
    google_project_service.run_api
  ]
}
```

### 步驟 2：執行部署

在終端機中初始化並套用此設定：

```bash
# 初始化 Terraform 專案
terraform init

# 預覽執行計畫
terraform plan

# 部署資源到 GCP
terraform apply -auto-approve
```

> 📝 **觀察重點**：
> 在執行 `apply` 的日誌中，您會看到 `google_project_service.run_api` 的狀態變為 `Creation complete` 後，`google_cloud_run_v2_service.hello_service` 才會開始執行 `Creating...`。這正是 `depends_on` 的作用。

---

## 🛡️ 最佳實踐與潛在副作用 (Best Practices & Side Effects)

雖然 `depends_on` 非常強大，但它應該是您在管理相依性時的**最後手段**。

### 1. 優先使用隱式相依
如果可以透過參數引用來建立關聯，就不要使用 `depends_on`。
* **為什麼**：隱式相依能保持代碼的整潔，並讓 Terraform 能夠最優化地處理並行建立。顯式相依會人為地限制並行度，可能拉長部署時間。

### 2. 了解對 `terraform destroy` 的影響
當您銷毀資源時，Terraform 會**反向**解析相依性。
* 如果資源 B `depends_on` 資源 A，則在銷毀時，Terraform 會先完整地銷毀資源 B，最後才銷毀資源 A。如果相依鏈過於複雜，可能會導致銷毀過程卡住或報錯。

### 3. 謹慎在 Module 層級使用 `depends_on`
自 Terraform v0.13 起，您可以在 `module` 區塊中使用 `depends_on`：
```terraform
module "gcp_infrastructure" {
  source = "./modules/infra"
  
  depends_on = [
    google_project_service.enable_apis
  ]
}
```
* ⚠️ **警告**：這會導致該模組內部的**所有資源**都被延遲，直到列表中的相依資源全部建立完畢。這會顯著降低 Terraform 部署的效能，並容易在複雜的模組中引發循環相依。

---

## 🔍 常見問題 & 故障排除 (Troubleshooting)

### Q1: 部署時出現 `Error: Cycle:` 錯誤（循環相依）
- **原因**：當資源 A 相依於資源 B，而同時資源 B 又直接或間接地相依於資源 A 時，Terraform 無法決定誰該先建立，因而拋出 `Cycle` 錯誤。
- **解決方案**：
  1. 使用以下指令輸出相依性圖檔，找出循環迴圈：
     ```bash
     terraform graph | dot -Tpng > graph.png
     ```
  2. 檢查您的 `depends_on` 配置，確保沒有將關係指回上游資源。
  3. 嘗試將一些參數拆分到第三個獨立資源中，以打破循環。

### Q2: 為什麼我已經加了 `depends_on`，後續資源建立還是因為權限或 API 未就緒而失敗？
- **原因**：有些雲端資源（如 GCP 的 IAM 政策或 API 啟用）在雲端控制台回傳「建立成功」後，其底層系統通常還需要幾秒鐘到幾分鐘的**傳播延遲 (Propagation Delay)** 才能真正生效。
- **解決方案**：
  在 `depends_on` 之外，可以使用 `time_sleep` 資源來強制 Terraform 在兩個資源之間等待一段固定的時間：
  ```terraform
  resource "google_project_service" "run_api" {
    service = "run.googleapis.com"
  }

  # 強制等待 30 秒以確保 API 傳播完成
  resource "time_sleep" "wait_30_seconds" {
    depends_on = [google_project_service.run_api]
    create_duration = "30s"
  }

  resource "google_cloud_run_v2_service" "hello_service" {
    name     = "hello-run-service"
    # 相依於 time_sleep，間接確保 API 已啟用且傳播完成
    depends_on = [time_sleep.wait_30_seconds]
    # ... 其它參數
  }
  ```

---

## 📚 延伸閱讀 (References & Further Reading)
- [HashiCorp Terraform: The depends_on Meta-Argument](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on)
- [Terraform Core Concepts: Resource Dependencies](https://developer.hashicorp.com/terraform/tutorials/cli/dependencies)
- [GCP Resource Manager: API Rate Limits and Propagation](https://cloud.google.com/resource-manager/docs/limits)
