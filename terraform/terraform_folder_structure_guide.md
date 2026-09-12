# Terraform 專案結構與模組化指南

## 1. 簡介
Terraform 專案結構是指在管理基礎設施即程式碼 (IaC) 時，如何組織你的 Terraform 檔案和目錄。一個清晰、有組織的專案結構對於提高可讀性、可維護性、可重用性以及團隊協作效率至關重要。模組化 (Modularization) 則是將可重複使用的基礎設施組件封裝成獨立的模組，進一步提升效率與一致性。

### 模組化的好處：
*   **可重用性 (Reusability)**：將常見的資源模式（如 VPC、EC2 實例、S3 儲存桶）封裝為模組，可以在不同專案或環境中重複使用。
*   **一致性 (Consistency)**：確保所有部署的資源都遵循相同的標準和最佳實踐。
*   **可維護性 (Maintainability)**：當需要修改特定資源的配置時，只需更新對應的模組即可，降低維護成本。
*   **簡化複雜性 (Simplifying Complexity)**：將大型基礎設施分解為更小、更易於管理的模組，降低理解和管理的複雜性。
*   **團隊協作 (Team Collaboration)**：不同的團隊成員可以專注於開發和維護不同的模組。

## 2. 核心概念
在深入探討專案結構之前，理解一些 Terraform 的核心概念至關重要。

### 工作區 (Workspace) 與環境 (Environment)
*   **工作區 (Workspace)**：Terraform 提供工作區功能來管理多個獨立的狀態文件。這允許你在相同的 Terraform 配置下，部署多個獨立的基礎設施實例，例如開發、測試和生產環境。預設情況下，Terraform 會在 `default` 工作區中運行。
*   **環境 (Environment)**：通常指的是不同部署階段，如 `dev` (開發), `stg` (預生產), `prd` (生產)。雖然工作區可以用於管理多環境，但更推薦的方式是使用資料夾來物理隔離不同環境的配置。

### 遠端狀態 (Remote State)
Terraform 狀態文件 (`terraform.tfstate`) 記錄了 Terraform 管理的所有資源的真實狀態。在團隊協作中，將狀態文件儲存在遠端後端（如 S3, Azure Blob Storage, GCS）是最佳實踐，以確保狀態的一致性和安全性。

### 後端 (Backend)
後端是 Terraform 用來儲存狀態文件和執行操作的儲存機制。配置遠端後端是實現遠端狀態管理的關鍵。

### 輸入變數 (Input Variables) 與輸出 (Outputs)
*   **輸入變數 (Input Variables)**：允許你自定義 Terraform 配置中的值，使其更具彈性。例如，你可以定義一個 `region` 變數來指定部署區域。
*   **輸出 (Outputs)**：模組的輸出值，可以被其他模組或根模組引用。例如，一個 VPC 模組可以輸出其 ID，供後續的子網路模組使用。

## 3. 建議的專案結構
一個良好的 Terraform 專案結構通常採用模組化的方法，將基礎設施劃分為可重用的組件。

### 根模組 (Root Module)
*   **定義**：執行 `terraform apply` 的頂級目錄。它將子模組組合成完整的基礎設施。
*   **職責**：主要用於定義後端配置、提供環境特定的變數值，並調用子模組。
*   **範例**：通常包含 `main.tf`, `versions.tf`, `providers.tf`, `terraform.tfvars`。

### 子模組 (Child Modules)
*   **定義**：封裝了一組相關資源的獨立 Terraform 配置。這些模組可以在多個根模組中重複使用。
*   **職責**：定義可重用的基礎設施組件，如 EC2 實例、VPC、資料庫等。
*   **範例**：可以有自己的 `main.tf`, `variables.tf`, `outputs.tf`。

### 環境特定配置 (Environment-specific Configurations) - 基於資料夾隔離 (推薦用於複雜場景)
為每個環境（如 dev, stg, prd）創建單獨的資料夾，每個資料夾內部包含該環境特有的配置。
*   **結構**：
    ```
    .
    ├── environments/
    │   ├── dev/
    │   │   ├── main.tf
    │   │   ├── variables.tf
    │   │   └── terraform.tfvars
    │   ├── stg/
    │   │   ├── main.tf
    │   │   ├── variables.tf
    │   │   └── terraform.tfvars
    │   └── prd/
    │       ├── main.tf
    │       ├── variables.tf
    │       └── terraform.tfvars
    └── modules/
        ├── vpc/
        │   ├── main.tf
        │   ├── variables.tf
        │   └── outputs.tf
        └── ec2/
            ├── main.tf
            ├── variables.tf
            └── outputs.tf
    ```
*   **優點**：清晰地隔離不同環境，避免意外更改，並允許不同環境使用不同的後端和變數值。

### 更簡潔的專案結構 (當基礎設施架構一致時)
對於基礎設施架構在各環境中完全相同，僅配置參數（如專案 ID、區域、機器類型、計數等）不同的情況，採用共享程式碼庫搭配獨立的 `vars/` 資料夾來管理環境變數，是一種更簡潔高效的做法。

*   **優點**：極高的程式碼重用性、簡化的變更管理、清晰的環境差異配置、減少檔案數量。
*   **缺點**：操作風險較高（易將錯誤變數應用到錯誤環境）、彈性受限（難以處理環境間顯著的架構差異）、變數管理可能變得複雜。

*   **結構**：
    ```
    .
    ├── main.tf                 # 核心 Terraform 程式碼 (定義資源並引用變數)
    ├── variables.tf            # 定義所有會用到的變數
    ├── providers.tf            # GCP Provider 配置
    ├── versions.tf             # Terraform 版本限制
    ├── vars/
    │   ├── dev.tfvars          # 開發環境變數值
    │   ├── stg.tfvars          # 預生產環境變數值
    │   └── prd.tfvars          # 生產環境變數值
    └── modules/                # (可選) 可重用的子模組，如網路、計算等
        └── ...
    ```

*   **`main.tf` 範例 (假設用於 GCP Compute Instance)**：
    ```terraform
    # main.tf

    variable "project_id" {
      description = "The GCP project ID."
      type        = string
    }

    variable "region" {
      description = "The GCP region."
      type        = string
    }

    variable "instance_name_prefix" {
      description = "Prefix for the compute instance name."
      type        = string
    }

    variable "machine_type" {
      description = "Machine type for the compute instance."
      type        = string
    }

    variable "instance_count" {
      description = "Number of compute instances to create."
      type        = number
      default     = 1
    }

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

    data "google_compute_image" "debian_image" {
      family  = "debian-11"
      project = "debian-cloud"
    }

    resource "google_compute_instance" "app_instance" {
      count        = var.instance_count
      name         = "${var.instance_name_prefix}-${count.index}"
      machine_type = var.machine_type
      zone         = "${var.region}-a" # 假設都部署在第一個可用區

      boot_disk {
        initialize_params {
          image = data.google_compute_image.debian_image.self_link
        }
      }

      network_interface {
        network = "default"
      }

      labels = {
        environment = split(".", var.project_id)[0] # 從 project_id 判斷環境標籤
      }
    }
    ```

*   **`vars/dev.tfvars` 範例**：
    ```terraform
    project_id         = "your-dev-gcp-project-id"
    region             = "us-central1"
    instance_name_prefix = "dev-app"
    machine_type       = "e2-small"
    instance_count     = 2
    ```

*   **`vars/prd.tfvars` 範例**：
    ```terraform
    project_id         = "your-prd-gcp-project-id"
    region             = "us-central1"
    instance_name_prefix = "prd-app"
    machine_type       = "e2-medium"
    instance_count     = 5
    ```

*   **執行方式**：

    要部署開發環境，你會這樣執行：
    ```bash
    cd /path/to/your/terraform_project
    terraform init
    terraform plan -var-file=./vars/dev.tfvars
    terraform apply -var-file=./vars/dev.tfvars
    ```

    要部署生產環境，你會這樣執行：
    ```bash
    cd /path/to/your/terraform_project
    terraform init
    terraform plan -var-file=./vars/prd.tfvars
    terraform apply -var-file=./vars/prd.tfvars
    ```

### 公用模組庫 (Shared Modules Repository)
對於組織內多個專案通用的模組，可以考慮將其放置在一個單獨的 Git 儲存庫中，作為一個共享模組庫。
*   **優點**：促進模組的廣泛重用，統一標準，便於集中管理和版本控制。

## 4. 檔案組織與命名慣例
Terraform 鼓勵使用清晰的檔案命名慣例，以提高可讀性。

*   `main.tf`：主要的配置檔案，通常包含資源定義和模組調用。
*   `variables.tf`：定義所有輸入變數。
*   `outputs.tf`：定義所有輸出值。
*   `versions.tf`：定義 Terraform 版本要求和 provider 版本限制。
*   `providers.tf`：配置使用的 cloud provider 及其認證資訊。
*   `terraform.tfvars`：為變數提供預設值或環境特定值。不應提交到版本控制（對於敏感資料）。
*   `.terraformignore`：類似 `.gitignore`，用於指定 Terraform 應忽略的檔案或目錄。

## 5. 模組間的互動
模組化是 Terraform 的核心功能之一，理解模組如何互相作用至關重要。

### 如何引用子模組
在根模組或另一個子模組中，可以使用 `module` 區塊來引用子模組：

```terraform
module "my_vpc" {
  source = "./modules/vpc" # 本地模組路徑
  # source = "github.com/my-org/terraform-modules//vpc?ref=v1.0.0" # 遠端 Git 模組
  cidr_block = "10.0.0.0/16"
  region     = var.aws_region
}
```

*   `source`：指定模組的來源，可以是本地路徑、Git 儲存庫、Terraform Registry 等。
*   傳遞變數：在 `module` 區塊內，將值賦給子模組的輸入變數。

### 跨模組傳遞變數與輸出
模組之間的通訊主要通過輸入變數和輸出值進行。

*   **輸入**：父模組通過在 `module` 區塊中定義變數來向子模組傳遞值。
*   **輸出**：子模組通過 `output` 區塊暴露值，這些值可以被父模組引用。

```terraform
# modules/vpc/outputs.tf
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "The ID of the VPC."
}

# environments/dev/main.tf (引用 VPC 模組的輸出)
resource "aws_subnet" "example" {
  vpc_id            = module.my_vpc.vpc_id # 引用子模組輸出
  cidr_block        = "10.0.1.0/24"
  availability_zone = "ap-southeast-1a"
}
```

### 資料來源 (Data Sources) 的使用
資料來源允許 Terraform 讀取現有基礎設施的資訊，而不是管理它們。這對於跨模組或跨專案共享資訊非常有用。

```terraform
data "google_compute_image" "debian_image" {
  family  = "debian-11"
  project = "debian-cloud"
}

resource "google_compute_instance" "default" {
  name         = "gcp-instance-example"
  machine_type = "e2-medium"
  zone         = "us-central1-a"
  boot_disk {
    initialize_params {
      image = data.google_compute_image.debian_image.self_link
    }
  }
  network_interface {
    network = "default"
  }
}
```

## 6. 多環境管理
管理多個環境是大型專案的常見需求。

### 使用工作區 (Workspaces)
*   **優點**：快速切換不同環境，適用於輕量級的多環境場景。
*   **缺點**：所有工作區共享相同的配置程式碼，難以管理環境之間的差異。不推薦用於大型或複雜的多環境部署。

```bash
terraform workspace new dev
terraform workspace select dev
terraform apply -var-file="dev.tfvars"
```

### 使用資料夾分離不同環境 (推薦)
*   **優點**：
    *   **明確隔離**：每個環境都有自己的配置資料夾，物理上隔離了代碼。
    *   **獨立後端**：每個環境可以配置不同的遠端後端，確保狀態文件完全獨立。
    *   **靈活配置**：允許不同環境之間存在顯著的配置差異，易於管理環境特定的變數和資源。
*   **操作**：導航到對應的環境資料夾，然後運行 `terraform init`, `terraform plan`, `terraform apply`。

## 7. 最佳實踐
遵循這些最佳實踐，可以幫助你構建健壯且易於維護的 Terraform 專案。

*   **保持模組的單一職責 (Single Responsibility Principle)**：每個模組應該只負責管理一組相關的資源。例如，一個模組負責 VPC，另一個模組負責 EC2 實例。
*   **版本控制模組**：將模組視為程式碼庫，進行版本控制。當模組更新時，使用版本標籤 (tag) 來確保穩定性。
*   **自動化測試**：為你的 Terraform 配置編寫自動化測試（例如使用 Terratest），以確保基礎設施按預期部署。
*   **文件化**：為每個模組和專案結構提供清晰的文檔，解釋其用途、輸入變數、輸出值和使用方法。
*   **CI/CD 整合**：將 Terraform 工作流整合到你的持續整合/持續部署 (CI/CD) 管道中，實現自動化的部署和管理。
*   **避免敏感資訊提交**：使用環境變數、Terraform Cloud 變數集或外部密鑰管理服務（如 AWS Secrets Manager, HashiCorp Vault）來管理敏感資訊，不要將其硬編碼或提交到 `terraform.tfvars`。
