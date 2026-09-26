# ☁️ Cloud & Infrastructure Tech Document Repository

歡迎來到 **Cloud & Infrastructure Tech Document Repository**！本專案匯集了關於 **Google Cloud (GCP)** 與 **Terraform** 基礎設施即程式碼 (IaC) 的核心技術指南、實戰教學與架構配置範例，旨在提供開發與維運團隊高水準、標準化的技術參考文件。

---

## 📂 專案目錄結構

```text
.
├── google_cloud/                 # Google Cloud (GCP) 相關技術指南
│   ├── gcp_cloud_run_job_yaml_guide.md
│   ├── gcp_cloud_run_pubsub_guide.md
│   ├── gcp_cloud_run_service_account_guide.md
│   ├── gcp_cloud_run_service_yaml_guide.md
│   └── gcp_cloud_run_terraform_guide.md
├── terraform/                    # Terraform 基礎設施與進階實戰指南
│   ├── terraform_conditional_resource_guide.md
│   ├── terraform_core_concepts.md
│   ├── terraform_depends_on_guide.md
│   ├── terraform_folder_structure_guide.md
│   ├── terraform_import_and_remove_guide.md
│   ├── terraform_intro.md
│   ├── terraform_moved_block_guide.md
│   └── terraform_resource_vs_module.md
└── README.md
```

---

## 📚 技術文件索引

### ☁️ Google Cloud (GCP) 雲端架構指南
| 文件名稱 | 說明簡介 |
| :--- | :--- |
| [`gcp_cloud_run_service_yaml_guide.md`](google_cloud/gcp_cloud_run_service_yaml_guide.md) | GCP Cloud Run Service YAML 配置與部署實戰指南 |
| [`gcp_cloud_run_job_yaml_guide.md`](google_cloud/gcp_cloud_run_job_yaml_guide.md) | GCP Cloud Run Job YAML 配置與批次任務執行指南 |
| [`gcp_cloud_run_pubsub_guide.md`](google_cloud/gcp_cloud_run_pubsub_guide.md) | 整合 Cloud Run 與 Pub/Sub 的非同步事件驅動架構指南 |
| [`gcp_cloud_run_service_account_guide.md`](google_cloud/gcp_cloud_run_service_account_guide.md) | Cloud Run 服務帳號權限管理與安全最佳實踐 |
| [`gcp_cloud_run_terraform_guide.md`](google_cloud/gcp_cloud_run_terraform_guide.md) | 使用 Terraform 自動化部署與管理 Cloud Run 服務 |

### 🏗️ Terraform 基礎設施即程式碼 (IaC) 指南
| 文件名稱 | 說明簡介 |
| :--- | :--- |
| [`terraform_intro.md`](terraform/terraform_intro.md) | Terraform 入門核心觀念與架構介紹 |
| [`terraform_core_concepts.md`](terraform/terraform_core_concepts.md) | Terraform 核心概念深入解析 (State, Provider, Provisioner) |
| [`terraform_folder_structure_guide.md`](terraform/terraform_folder_structure_guide.md) | Terraform 專案資料夾結構與最佳模組化設計 |
| [`terraform_resource_vs_module.md`](terraform/terraform_resource_vs_module.md) | Resource 與 Module 的差異比較與選型指南 |
| [`terraform_depends_on_guide.md`](terraform/terraform_depends_on_guide.md) | 處理相依性：深入理解 `depends_on` 的使用時機與陷阱 |
| [`terraform_conditional_resource_guide.md`](terraform/terraform_conditional_resource_guide.md) | 條件式資源建立：靈活運用 `count` 與 `for_each` |
| [`terraform_import_and_remove_guide.md`](terraform/terraform_import_and_remove_guide.md) | 狀態管理實戰：Resource Import 與 State Removal 教學 |
| [`terraform_moved_block_guide.md`](terraform/terraform_moved_block_guide.md) | 重構利器：使用 `moved` 區塊無痛遷移與重新命名資源 |

---

## 💡 貢獻與協作規範

本專案文件遵循嚴謹的技術文件編修規範：
- **中英排版規範**：英文與數字左右各保留半形空白。
- **程式碼高亮**：所有程式碼與 YAML/HCL 設定皆附帶完整註解與正確語法標示。
- **視覺化輔助**：重要架構與流程皆輔以 Mermaid 圖表說明。

歡迎透過 Pull Request 或 Issue 提出文件修正與補充建議！