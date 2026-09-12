# Terraform 核心概念

## 1. 簡介

Terraform 是一個開源的基礎設施即程式碼 (Infrastructure as Code, IaC) 工具，由 HashiCorp 開發。它允許您使用高階配置語言來定義和佈建雲端及本地資源。透過將基礎設施定義為程式碼，Terraform 實現了可版本控制、可重複和可預測的基礎設施管理。

### 基礎設施即程式碼 (IaC) 的優勢
*   **自動化**：自動佈建和管理基礎設施，減少手動錯誤。
*   **版本控制**：基礎設施配置儲存在版本控制系統中，方便追蹤更改、協同合作和回溯。
*   **可重複性**：確保在不同環境（開發、測試、生產）中部署相同的基礎設施。
*   **透明度**：基礎設施的狀態和預期配置一目了然。

## 2. 核心概念

### 2.1 供應商 (Providers)
供應商是 Terraform 用於與各種雲端服務、SaaS 產品或其他 API 進行互動的插件。每個供應商都定義了特定服務的資源類型和資料來源。例如，`google` 供應商用於管理 Google Cloud Platform (GCP) 資源，而 `aws` 供應商用於 Amazon Web Services。

### 2.2 資源 (Resources)
資源是 Terraform 配置中最重要的元素，它們代表了您的基礎設施組件，例如虛擬機、網路、資料庫或儲存桶。每個資源區塊都包含一個資源類型和一個本地名稱，以及定義該資源狀態所需的參數。

**範例:**
```terraform
resource "google_compute_instance" "default" {
  name         = "my-instance"
  machine_type = "e2-medium"
  zone         = "asia-east1-b"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }
}
```

### 2.3 資料來源 (Data Sources)
資料來源允許 Terraform 從現有的基礎設施或服務中獲取資訊，而無需管理該資源的生命週期。這對於引用已在 Terraform 外部建立或由其他 Terraform 配置管理的資源非常有用。

**範例:**
```terraform
data "google_project" "project" {
  project_id = "your-gcp-project-id"
}
```

### 2.4 變數 (Variables)
變數用於參數化 Terraform 配置，使其更具彈性和可重用性。您可以在執行 `terraform plan` 或 `terraform apply` 時透過命令列、檔案或環境變數提供變數值。

**範例 (variables.tf):**
```terraform
variable "project_id" {
  description = "The GCP project ID"
  type        = string
}

variable "region" {
  description = "The GCP region"
  type        = string
  default     = "asia-east1"
}
```

### 2.5 輸出 (Outputs)
輸出值用於從 Terraform 配置中匯出特定資料，例如新建立的資源的 IP 地址、ID 或其他屬性。這些輸出可以被其他 Terraform 配置或外部工具使用。

**範例 (outputs.tf):**
```terraform
output "instance_ip_address" {
  value       = google_compute_instance.default.network_interface[0].network_ip
  description = "The IP address of the compute instance"
}
```

### 2.6 模組 (Modules)
模組是將 Terraform 配置組織成邏輯單元的方法，可以封裝和重用。它們允許您將複雜的配置分解為更小、更易於管理的區塊，並在多個項目中共享這些區塊。

### 2.7 狀態管理 (State Management)
Terraform 狀態文件 (`terraform.tfstate`) 是一個 JSON 文件，它記錄了 Terraform 管理的所有真實基礎設施資源的狀態。它將您的配置與實際部署的資源進行映射，並儲存了資源的屬性。

**重要性:**
*   **映射**：Terraform 使用狀態文件來了解哪些真實世界資源對應到您的配置。
*   **性能**：Terraform 在執行 `plan` 時會讀取狀態文件，而不是每次都查詢雲端 API，以提高性能。
*   **同步**：確保您的配置與實際基礎設施保持同步。

**遠端狀態:**
將狀態文件儲存在遠端儲存（例如 GCP Cloud Storage、AWS S3、Terraform Cloud）中是最佳實踐，尤其是在團隊協作環境中。這可以防止狀態鎖定問題，並確保所有團隊成員都能訪問最新的狀態。

## 3. Terraform 工作流程

Terraform 的基本工作流程包括以下步驟：

1.  **初始化 (`terraform init`)**:
    *   此命令會初始化工作目錄，下載所需的供應商插件和模組。
    *   它會建立一個 `.terraform` 目錄，其中包含供應商插件和模組的快取。

2.  **規劃 (`terraform plan`)**:
    *   此命令會生成一個執行計畫，顯示 Terraform 將執行哪些操作（建立、修改或刪除）以達到配置中定義的所需狀態。
    *   它不會實際更改任何基礎設施。

3.  **應用 (`terraform apply`)**:
    *   此命令會執行 `plan` 生成的執行計畫，實際在您的雲端或本地環境中佈建或更改基礎設施。
    *   在執行之前，它會顯示一個確認提示，要求您輸入 `yes`。

4.  **銷毀 (`terraform destroy`)**:
    *   此命令會銷毀所有由當前 Terraform 配置管理的資源。
    *   它會顯示一個確認提示，要求您輸入 `yes`。

## 4. 基本 GCP 範例：建立 Compute Engine 實例

以下是一個簡單的 Terraform 配置，用於在 GCP 中建立一個 Compute Engine 虛擬機。

```terraform
# main.tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_instance" "web_server" {
  name         = "web-server-instance"
  machine_type = "e2-medium"
  zone         = "asia-east1-b"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }

  metadata_startup_script = "sudo apt-get update && sudo apt-get install -y apache2 && echo '<!doctype html><html><body><h1>Hello World from Terraform!</h1></body></html>' | sudo tee /var/www/html/index.html"
}
```

```terraform
# variables.tf
variable "project_id" {
  description = "The GCP project ID"
  type        = string
}

variable "region" {
  description = "The GCP region"
  type        = string
  default     = "asia-east1"
}
```

```terraform
# outputs.tf
output "instance_name" {
  value       = google_compute_instance.web_server.name
  description = "The name of the Compute Engine instance"
}

output "instance_ip" {
  value       = google_compute_instance.web_server.network_interface[0].network_ip
  description = "The internal IP address of the Compute Engine instance"
}
```

**如何執行此範例：**

1.  **設定 GCP 認證**：確保您的環境已配置 GCP 認證。通常可以透過 `gcloud auth application-default login` 或設定服務帳戶金鑰來完成。
2.  **建立檔案**：將上述三個代碼區塊分別儲存為 `main.tf`、`variables.tf` 和 `outputs.tf` 在同一個目錄中。
3.  **初始化 Terraform**：
    ```bash
    terraform init
    ```
4.  **規劃部署**：
    ```bash
    terraform plan -var="project_id=YOUR_GCP_PROJECT_ID"
    ```
    將 `YOUR_GCP_PROJECT_ID` 替換為您的實際 GCP 專案 ID。
5.  **應用部署**：
    ```bash
    terraform apply -var="project_id=YOUR_GCP_PROJECT_ID"
    ```
    輸入 `yes` 確認。
6.  **銷毀資源**：
    ```bash
    terraform destroy -var="project_id=YOUR_GCP_PROJECT_ID"
    ```
    輸入 `yes` 確認。

## 5. 最佳實踐

*   **版本控制**：將您的 Terraform 配置儲存在 Git 等版本控制系統中。
*   **遠端狀態**：始終使用遠端後端來儲存您的 Terraform 狀態文件，以實現團隊協作和狀態鎖定。
*   **模組化**：利用模組來組織和重用您的配置，避免重複程式碼。
*   **環境區隔**：為不同的環境（開發、測試、生產）使用獨立的 Terraform 工作空間或目錄。
*   **敏感資料處理**：不要將敏感資訊（例如密碼、API 金鑰）直接寫入 Terraform 配置中。使用 Terraform Vault、GCP Secret Manager 或其他安全工具來管理。
*   **程式碼審查**：在將更改應用到基礎設施之前，對 Terraform 配置進行程式碼審查。
