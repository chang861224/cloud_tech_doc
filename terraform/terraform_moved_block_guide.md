# Terraform `moved` 區塊概念與實戰指南

在本篇技術文件中，我們將深入探討 Terraform 1.1 引入的強大功能 —— **`moved` 區塊**。透過 `moved` 區塊，您可以安全、宣告式地在重構 Terraform 程式碼時重新命名或搬遷資源，而無需手動執行複雜的 `terraform state mv` 指令。

---

## 🎯 1. 什麼是 Terraform `moved` 區塊？

在早期的 Terraform 版本中，如果您想更改資源的名稱（例如將 `aws_instance.web` 改名為 `aws_instance.server`），或者將資源搬移至模組（Module）中，Terraform 會視為「刪除舊資源並建立新資源」。

為了避免這種破壞性的操作，工程師必須手動執行指令：
```bash
terraform state mv aws_instance.web aws_instance.server
```

而 **`moved` 區塊** 提供了一種**宣告式（Declarative）**的方法來處理狀態遷移。您只需直接寫在 `.tf` 程式碼中，Terraform 在執行 `terraform plan` 與 `terraform apply` 時就會自動識別並更新狀態，非常適合團隊協作與 CI/CD 自動化流程。

---

## 💡 2. 為什麼需要 `moved` 區塊？（解決痛點）

- **自動化與團隊協作**：所有人只要拉取最新的程式碼並執行 `terraform apply`，狀態遷移就會自動套用，無需團隊成員各自手動執行 `terraform state mv`。
- **防止資源重建**：避免因重新命名或結構調整而導致正式環境中的雲端資源被意外刪除並重新建立（Downtime）。
- **版本控制（Git History）**：遷移紀錄直接保留在程式碼變更紀錄中，清楚記錄架構演進的軌跡。

---

## 🛠️ 3. 常見使用情境與實務範例

### 情境 A：單純重新命名資源 (Renaming a Resource)

假設您原本有一個 GCP 雲端儲存桶（Cloud Storage Bucket）資源：

```terraform
resource "google_storage_bucket" "old_name" {
  name          = "my-app-bucket-prod"
  location      = "US"
  force_destroy = true
}
```

現在您想將其重新命名為 `main_bucket`。您只需要修改資源名稱，並加上 `moved` 區塊：

```terraform
resource "google_storage_bucket" "main_bucket" {
  name          = "my-app-bucket-prod"
  location      = "US"
  force_destroy = true
}

# 宣告狀態遷移
moved {
  from = google_storage_bucket.old_name
  to   = google_storage_bucket.main_bucket
}
```

---

### 情境 B：將資源移入或移出模組 (Moving Resources to/from Modules)

當專案逐漸龐大，您決定將獨立的 Google Cloud 虛擬機器（Compute Engine）封裝進共用模組 `compute_instance` 中：

原本的程式碼：
```terraform
resource "google_compute_instance" "web_server" {
  name         = "web-instance"
  machine_type = "e2-medium"
  zone         = "us-central1-a"

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

重構後的程式碼：
```terraform
module "compute_instance" {
  source = "./modules/compute_instance"
}
```

透過 `moved` 區塊，您可以輕鬆將原本的資源對應到模組內部的資源位址：

```terraform
moved {
  from = google_compute_instance.web_server
  to   = module.compute_instance.google_compute_instance.this
}
```

---

## ⚠️ 4. 注意事項與最佳實踐

1. **過渡期後可清除**：當團隊所有成員與 CI/CD 環境都已經執行過包含 `moved` 區塊的 `terraform apply` 後，該 `moved` 區塊就可以從程式碼中移除，狀態已經永久更新。
2. **配合版本控管**：強烈建議在進行大規模重構時，分批次、小範圍地使用 `moved` 區塊，確保每次計畫（Plan）的變更如預期。
3. **檢查 Plan 輸出**：在執行 `terraform apply` 之前，務必仔細檢查 `terraform plan` 的輸出，確認顯示的是 `# Object moved` 而不是 `(destroy)` 與 `(create)`。
