# Terraform 中的 Resource 與 Module：建立資源與管理 IAM 的差異

## 1. 前言

在 Terraform 中，`resource` 和 `module` 是兩個核心概念，用於定義和管理基礎設施。儘管它們都涉及資源的創建，但在抽象層次、重用性及複雜度管理方面有顯著差異。本文將深入探討 `resource` 和 `module` 的特性、使用時機、優缺點，並提供實用的程式碼範例，幫助您在不同的情境下做出最佳選擇。

## 2. Resource Block (資源區塊)

### 2.1 定義

`resource` 區塊是 Terraform 中最基本的組成單元，用於宣告單一或一組特定類型的基礎設施資源。每個 `resource` 區塊都綁定到一個特定的 Provider (例如 AWS、GCP、Azure)，並定義了該資源的屬性。

### 2.2 使用時機

- **建立單一、獨立的資源**：當您需要定義一個單一的 EC2 實例、S3 Bucket、GCP Cloud Storage Bucket 或 Azure Virtual Machine 時。
- **簡單的基礎設施組件**：對於不需高度抽象或重用的少量資源。
- **初學者入門**：學習 Terraform 的基本語法和資源配置時。

### 2.3 優缺點

**優點**：
- **直觀且易於理解**：直接映射到雲端供應商的資源類型。
- **精確控制**：可以直接配置每個資源的詳細屬性。
- **無需額外學習曲線**：與 Terraform 核心語法緊密結合。

**缺點**：
- **重複性高**：當需要建立多個相似資源時，會導致程式碼重複，難以維護。
- **缺乏抽象**：無法將一組相關資源封裝成可重用單元。
- **管理複雜度高**：隨著基礎設施規模擴大，單純使用 `resource` 會讓程式碼變得龐大且難以管理。

### 2.4 範例程式碼

以下是一個使用 `resource` 區塊在 GCP 上建立 Cloud Storage Bucket 的範例：

```terraform
resource "google_storage_bucket" "my_bucket" {
  name          = "my-unique-bucket-name-12345"
  location      = "ASIA-EAST1"
  project       = "your-gcp-project-id"
  force_destroy = true

  uniform_bucket_level_access = true
}

resource "google_project_iam_member" "bucket_viewer_iam" {
  project = "your-gcp-project-id"
  role    = "roles/storage.viewer"
  member  = "user:your-email@example.com"
}

output "bucket_name" {
  value = google_storage_bucket.my_bucket.name
}
```

## 3. Module Block (模組區塊)

### 3.1 定義

`module` 區塊允許將一組相關的 `resource` (或其他 `module`) 封裝成一個邏輯單元，並透過輸入變數 (input variables) 和輸出值 (output values) 進行參數化。模組是 Terraform 中實現程式碼重用、標準化和複雜度管理的關鍵機制。

### 3.2 使用時機

- **程式碼重用**：當您需要多次部署相同的基礎設施模式時，例如創建多個環境 (開發、測試、生產) 或多個服務實例。
- **標準化部署**：確保所有團隊成員使用一致的基礎設施配置，減少人為錯誤。
- **抽象化複雜度**：將複雜的基礎設施組件 (如 VPC 網路、Kubernetes Cluster) 封裝起來，提供簡潔的介面供其他團隊使用。
- **團隊協作**：不同團隊可以專注於開發和維護各自的模組，提高開發效率。

### 3.3 優缺點

**優點**：
- **高度重用性**：只需定義一次，即可在多個地方甚至多個專案中重複使用。
- **降低複雜度**：將底層的資源細節隱藏在模組內部，對使用者提供簡化的介面。
- **提升一致性與標準化**：確保所有部署都遵循相同的最佳實踐和配置。
- **加速部署**：透過重用已驗證的模組，可以更快地部署新的基礎設施。
- **易於維護**：修改模組內部邏輯時，所有使用該模組的地方都會自動更新。

**缺點**：
- **初始開發成本較高**：設計和開發高品質的模組需要較多時間和精力。
- **學習曲線**：理解模組的輸入、輸出、版本控制和源碼管理需要額外學習。
- **過度抽象的風險**：如果模組設計不當，可能會導致難以除錯或限制靈活性。

### 3.4 範例程式碼

假設我們有一個用於建立 Cloud Storage Bucket 並授予 IAM 角色的模組。模組結構可能如下：

```
├── modules/
│   └── gcs_bucket/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── main.tf
```

**`modules/gcs_bucket/main.tf` (模組定義)**:

```terraform
resource "google_storage_bucket" "this" {
  name          = var.bucket_name
  location      = var.location
  project       = var.project_id
  force_destroy = var.force_destroy

  uniform_bucket_level_access = true
}

resource "google_project_iam_member" "viewer_iam" {
  project = var.project_id
  role    = var.iam_role
  member  = var.iam_member
}
```

**`modules/gcs_bucket/variables.tf` (模組變數)**:

```terraform
variable "bucket_name" {
  description = "The name of the GCS bucket."
  type        = string
}

variable "location" {
  description = "The GCS bucket location."
  type        = string
  default     = "ASIA-EAST1"
}

variable "project_id" {
  description = "The GCP project ID."
  type        = string
}

variable "force_destroy" {
  description = "When deleting a bucket, this boolean option will delete all contained objects. If false, Terraform will fail to destroy buckets which are not empty."
  type        = bool
  default     = false
}

variable "iam_role" {
  description = "The IAM role to grant on the project."
  type        = string
  default     = "roles/storage.viewer"
}

variable "iam_member" {
  description = "The IAM member to grant the role to (e.g., user:email@example.com)."
  type        = string
}
```

**`modules/gcs_bucket/outputs.tf` (模組輸出)**:

```terraform
output "bucket_self_link" {
  description = "The self_link of the GCS bucket."
  value       = google_storage_bucket.this.self_link
}
```

**`main.tf` (根模組調用)**:

```terraform
module "dev_bucket" {
  source        = "./modules/gcs_bucket"
  bucket_name   = "my-dev-bucket-456"
  project_id    = "your-dev-project"
  iam_member    = "user:dev-user@example.com"
  force_destroy = true
}

module "prod_bucket" {
  source        = "./modules/gcs_bucket"
  bucket_name   = "my-prod-bucket-789"
  project_id    = "your-prod-project"
  iam_member    = "user:prod-user@example.com"
}

output "dev_bucket_link" {
  value = module.dev_bucket.bucket_self_link
}

output "prod_bucket_link" {
  value = module.prod_bucket.bucket_self_link
}
```

## 4. Resource 與 Module 的關鍵差異

| 特性         | `resource` 區塊                                | `module` 區塊                                                      |
| :----------- | :--------------------------------------------- | :----------------------------------------------------------------- |
| **抽象層次** | 低：直接定義雲端供應商的單一資源               | 高：將一組相關資源抽象為一個邏輯單元                               |
| **重用性**   | 低：程式碼複製貼上，易造成重複                 | 高：透過輸入變數和輸出值，實現程式碼的高度重用                     |
| **職責範圍** | 創建和管理單一資源                             | 封裝複雜的基礎設施模式，提供統一介面                               |
| **維護性**   | 隨著基礎設施規模擴大，維護成本急劇上升         | 集中管理邏輯，模組修改後可自動應用於所有調用點，降低維護成本       |
| **複雜度**   | 管理複雜的基礎設施時，易導致程式碼冗長和混亂   | 有效降低根模組的複雜度，提升可讀性和可管理性                       |
| **版本控制** | 無直接版本控制概念，依賴於根模組的版本控制     | 支援版本控制 (Local, Registry, Git)，有利於團隊協作和標準化       |

## 5. 使用時機與最佳實踐

### 5.1 何時使用 `resource`？

- **非常簡單、一次性的資源定義**：當您確定某個資源不會在其他地方被重用，且其配置邏輯非常簡單時。
- **模組內部**：在定義一個模組時，模組內部會使用大量的 `resource` 區塊來構建其功能。
- **快速原型開發**：在初期探索或驗證某個雲端資源的功能時。

### 5.2 何時使用 `module`？

- **重複性模式**：任何需要在多個環境、多個服務或多個專案中重複部署的基礎設施模式，例如：
    - 一個標準化的 VPC 網路設置。
    - 一個包含資料庫、應用程式伺服器和負載平衡器的服務堆疊。
    - 針對不同環境 (dev/staging/prod) 的資源部署。
- **基礎設施標準化**：希望所有團隊成員都使用統一、經過審核的基礎設施配置。
- **團隊邊界劃分**：將基礎設施責任劃分給不同的團隊，每個團隊負責維護和提供特定的模組。
- **複雜度管理**：當根模組變得過於龐大和難以理解時，應考慮將相關資源提取到模組中。

### 5.3 最佳實踐

- **保持模組精簡和單一職責**：每個模組應專注於一個特定的功能或解決一個特定的問題。
- **明確的輸入和輸出**：模組的變數 (inputs) 應清楚定義其用途，輸出 (outputs) 應提供有用的資訊給調用者。
- **模組版本控制**：對於共享模組，應使用版本控制來管理變更，確保穩定性和可預測性。
- **文件化**：為您的模組撰寫清晰的說明文件，包括如何使用、輸入、輸出和任何注意事項。
- **測試模組**：對模組進行單元測試和整合測試，確保其行為符合預期。
- **避免過度抽象**：不要為了使用模組而使用模組，當資源足夠簡單且不會重用時，直接使用 `resource` 可能更有效率。

## 6. 總結

`resource` 和 `module` 都是 Terraform 中不可或缺的組件。`resource` 提供對單一雲端資源的精確控制，而 `module` 則提供了一種強大的方式來抽象、重用和標準化基礎設施程式碼。理解兩者的差異和適用情境，並結合最佳實踐，將能幫助您建構出高效、可維護且易於擴展的基礎設施即程式碼 (IaC) 解決方案。在實際專案中，通常會以 `module` 的形式來調用和組合多個 `resource`，以實現複雜而有條理的基礎設施管理。