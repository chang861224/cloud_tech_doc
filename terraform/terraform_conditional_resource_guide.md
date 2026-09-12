# 🛠️ Terraform 控制資源是否建立的方式與依賴處理指南 (GCP 實戰篇)

在 Terraform 開發與維護過程中，我們經常需要根據不同環境（如 `dev`、`staging`、`prod`）、成本考量或是開關變數（Feature Flag）來決定**是否要建立特定的基礎設施資源**。

本篇文件將詳細介紹 Terraform 中用來控制資源建立的幾種主流方法，並**全面採用 Google Cloud Platform (GCP) 資源作為範例**，特別針對**當該資源作為其他資源的依賴項（Dependency）時**所產生的常見陷阱與最佳解法進行深入探討。

---

## 📌 一、 前言與應用場景

在 IaC (Infrastructure as Code) 的實踐中，動態控制資源建立的需求非常普遍：
1. **多環境差異**：生產環境 (`prod`) 需要建立高可用 Cloud VPN 或備份 Instance，而測試環境 (`dev`) 為了節省成本希望略過。
2. **開關控制 (Feature Flag)**：透過變數（Variable）動態啟用或停用 Cloud NAT、額外防火牆規則或日誌匯出機制。
3. **階段性部署**：在資源尚未準備好之前，先透過條件式暫緩建立。

---

## 📌 二、 方法一：使用 `count` 條件判斷（最常用）

利用 `count` 結合三元運算子（Ternary Operator），可以輕鬆實現 `0`（不建立）或 `1`（建立）的開關控制。

### 1. 基礎語法範例 (`main.tf`)

```hcl
variable "enable_cloud_nat" {
  type        = bool
  description = "是否啟用 Cloud NAT 閘道器"
  default     = false
}

resource "google_compute_router" "router" {
  name    = "my-router"
  region  = "asia-east1"
  network = google_compute_network.vpc.name
}

resource "google_compute_router_nat" "nat" {
  # 當變數為 true 時 count = 1（建立），為 false 時 count = 0（不建立）
  count = var.enable_cloud_nat ? 1 : 0

  name                                = "my-router-nat"
  router                              = google_compute_router.router.name
  region                              = google_compute_router.router.region
  nat_ip_allocate_option              = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
}
```

### 2. 引用時的注意事項
當資源使用 `count = 1` 或 `0` 時，該資源在 Terraform 中會變成一個**列表（List）**。因此在其他地方參照它時，必須加上索引 `[0]`：
* 正確寫法：`google_compute_router_nat.nat[0].id`
* ⚠️ **陷阱**：如果 `enable_cloud_nat = false`，直接參照 `google_compute_router_nat.nat[0].id` 會導致 Terraform 在執行計畫（Plan）時報錯（`Index out of range`）。

---

## 📌 三、 方法二：使用 `for_each` 與動態集合

除了 `count` 之外，`for_each` 也常被用來控制資源的建立與否。透過傳入空的 Map 或 Set，Terraform 會自動不建立任何資源。

### 1. 範例寫法

```hcl
variable "create_audit_bucket" {
  type        = bool
  description = "是否建立稽核專用 Cloud Storage Bucket"
  default     = false
}

resource "google_storage_bucket" "audit" {
  # 若為 true 則產生含有 1 個元素的集合，若為 false 則為空集合 {}
  for_each = var.create_audit_bucket ? toset(["main"]) : toset([])

  name          = "my-company-audit-logs-bucket"
  location      = "ASIA"
  force_destroy = true
}
```

### 2. 引用方式
當使用 `for_each` 時，參照方式為：`google_storage_bucket.audit["main"].id`。

---

## 📌 四、 核心重點：當被控制的資源作為其他資源的依賴項時

當資源 A（例如 Cloud NAT 或 GCS Bucket）透過 `count = 0` 或 `for_each = {}` 決定**不建立**時，若資源 B（例如防火牆規則 Firewall 或 IAM Policy Binding）需要依賴或參照資源 A 的屬性，就會面臨**「參照崩潰」**的問題。

以下是處理依賴關係的 3 種最佳解法：

### 🛠️ 解法 1：使用條件運算式（Conditional Expressions）進行安全參照
在資源 B 的屬性中，判斷該資源是否存在，若不存在則給予 `null` 或預設值。

```hcl
resource "google_compute_firewall" "custom_rule" {
  name    = "allow-custom-traffic"
  network = google_compute_network.vpc.name

  allow {
    protocol = "tcp"
    ports    = ["8080"]
  }

  # 安全參照：若 google_compute_router_nat.nat 存在則取 [0].id，否則為 null
  description = length(google_compute_router_nat.nat) > 0 ? "Managed by NAT ID: ${google_compute_router_nat.nat[0].id}" : "No NAT attached"
}
```

### 🛠️ 解法 2：利用 `try()` 函數組合（簡化複雜參照）
Terraform 內建的 `try()` 函數可以嘗試讀取屬性，若因為資源未建立而報錯，則自動回傳替代值（如 `null`）。

```hcl
resource "google_project_iam_member" "bucket_admin" {
  project = "my-gcp-project-id"
  role    = "roles/storage.admin"
  member  = "serviceAccount:my-sa@my-gcp-project.iam.gserviceaccount.com"

  # 嘗試取得選配 GCS 桶子的名稱，若失敗或不存在則略過或給予預設值
  condition {
    title       = "conditional_access"
    description = "Only active when bucket exists"
    expression  = "resource.name == '${try(google_storage_bucket.audit["main"].name, "none")}'"
  }
}
```

### 🛠️ 解法 3：搭配 `count` 控制下游資源的建立連鎖反應
如果下游資源（資源 B）完全依賴上游資源（資源 A），最乾淨的做法是讓下游資源的 `count` 條件**直接繫結**於同一個開關變數或上游資源的長度：

```hcl
# 上游資源：選配的 Pub/Sub Topic
resource "google_pubsub_topic" "alerts" {
  count = var.enable_monitoring ? 1 : 0
  name  = "production-alert-topic"
}

# 下游資源：只有在上游存在時才建立對應的 IAM 訂閱權限
resource "google_pubsub_topic_iam_member" "publisher" {
  count   = var.enable_monitoring ? 1 : 0
  topic   = google_pubsub_topic.alerts[0].name
  role    = "roles/pubsub.publisher"
  member  = "serviceAccount:monitoring-sa@my-gcp-project.iam.gserviceaccount.com"
}
```

---

## 📌 五、 方法三：使用 `lifecycle` 與條件檢查（Terraform 1.2+）

如果您希望在 Terraform `plan` 階段就對資源的相依性或條件進行防呆檢查，可以結合 `precondition`：

```hcl
resource "google_compute_instance" "app_server" {
  name         = "prod-app-server"
  machine_type = "e2-medium"
  zone         = "asia-east1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  network_interface {
    network = "default"
  }

  lifecycle {
    precondition {
      condition     = var.environment != "prod" || var.enable_cloud_nat == true
      error_message = "生產環境 (prod) 部署 App Server 時，必須同步啟用 Cloud NAT 網路閘道器。"
    }
  }
}
```

---

## 📌 六、 進階延伸：當控制的對象是「Module（模組）」時？

在 Terraform 中，除了單一的 `resource` 可以透過 `count` 或 `for_each` 控制建立與否之外，**`module` 區塊本身也完全支援 `count` 與 `for_each`**！

不過，當模組作為條件式建立時，在**輸出屬性（Outputs）的存取**與**下游依賴**上有一些重要的行為差異與注意事項：

### 1. 模組中使用 `count` 控制是否建立
透過傳入開關變數控制 `count`，若為 `0` 則整個 Module 內部包含的所有資源都不會被建立：

```hcl
variable "enable_gcp_vpc_module" {
  type        = bool
  description = "是否啟用企業專用 VPC 模組"
  default     = false
}

module "secured_vpc" {
  # 關鍵：模組原生支援 count 參數 (Terraform 0.13+)
  count = var.enable_gcp_vpc_module ? 1 : 0

  source = "./modules/gcp_vpc"

  project_id = "my-gcp-project-id"
  region     = "asia-east1"
  cidr_block = "10.0.0.0/16"
}
```

### 2. 下游如何安全參照條件式模組的 Outputs？
當模組使用了 `count = var.enable_gcp_vpc_module ? 1 : 0` 後，該模組的 Outputs 會自動變成**列表（List）**型態。若要在其他資源或模組中參照它的輸出（例如取得 VPC ID），必須加上索引 `[0]`，並配合條件判斷防止報錯：

```hcl
resource "google_compute_subnetwork" "custom_subnet" {
  name          = "app-subnetwork"
  region        = "asia-east1"
  ip_cidr_range = "10.0.1.0/24"
  
  # 安全參照：若 secured_vpc 模組存在則取 [0].vpc_id，否則為 null (或導向 default VPC)
  network       = length(module.secured_vpc) > 0 ? module.secured_vpc[0].vpc_id : "default"
}
```

或者使用 `try()` 函數簡化寫法：
```hcl
resource "google_compute_subnetwork" "custom_subnet" {
  name          = "app-subnetwork"
  region        = "asia-east1"
  ip_cidr_range = "10.0.1.0/24"
  
  # 嘗試取得模組輸出，失敗或模組未建立時回退到預設網路
  network       = try(module.secured_vpc[0].vpc_id, "default")
}
```

### 3. ⚠️ 模組條件控制的常見限制與陷阱
1. **Providers 傳遞限制**：條件式載入的模組若內部有複雜的 Provider 設定，需特別注意 Provider 的別名（alias）對應，避免在 `count = 0` 時因未初始化而報錯。
2. **Outputs 結構改變**：一旦模組加上了 `count`，外部任何地方呼叫該模組 Output 時都必須加上 `[0]` 或進行迭代（若使用 `for_each` 則改用 `["key"]`），無法直接使用 `module.xxx.output_name`。

---

## 📌 七、 各種方法的優缺點比較與總結建議

| 方法 | 適用情境 | 優進 | 缺點 / 注意事項 |
| :--- | :--- | :--- | :--- |
| **`count = var.flag ? 1 : 0`** | 單一 GCP 資源、簡單開關 (如 Cloud NAT) | 語法直覺、易於理解 | 引用時需加 `[0]`，若未建立直接參照會報錯 |
| **`for_each` 結合空集合** | 同質的多個 GCP 資源 (如多個 GCS Bucket) | 擴充性強、不會因索引變動重建 | 語法稍微繁瑣 (`toset([])`) |
| **條件運算式 / `try()`** | 下游資源需要安全參照可能不存在的上游 GCP 屬性 | 防止 `Index out of range`，提升模組韌性 | 邏輯較長時可讀性降低 |

### 💡 專案最佳實踐建議
1. **開關連動**：在 GCP 架構中，如 VPC 相關連線元件（Router NAT、VPN Tunnel、Firewall），若下游資源強烈依賴上游，建議讓下游資源的 `count` 也與上游保持一致的條件判斷。
2. **善用 `try()` 與三元運算**：在共用模組（Module）中，面對可能被使用者關閉的選配 GCP 資源，務必使用 `try(..., null)` 或 `length(...) > 0` 來確保模組具備高度容錯能力。
