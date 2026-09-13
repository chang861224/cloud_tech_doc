---
name: update-readme
description: 專門用於檢查、組織並自動更新專案根目錄 `README.md` 文件的專業技能。當需要新增技術手冊、同步目錄結構、更新文件清單或優化專案導覽時使用。
---

# Update README Skill

本技能協助開發者或技術文件寫作者有條理地維護與更新專案的 `README.md`，確保專案首頁始終反映最新的目錄結構、文件清單與架構說明。

## 適用情境
- 新增 GCP 雲端服務或 Terraform 教學與指南後，需要同步更新 `README.md` 的索引或目錄。
- 專案架構重新調整、資料夾異動時。
- 需要確保 `README.md` 具備清晰的導覽、分類、快速連結與說明時。

## 更新流程

1. **掃描專案現況**
   - 檢查專案根目錄與子資料夾（如 `google_cloud/`, `terraform/` 等）。
   - 列出目前所有的 Markdown (.md) 文件與其對應路徑。

2. **分析變更與對齊分類**
   - 確認是否有新增、刪除或重新命名的文件。
   - 將文件依據主題（例如：Google Cloud、Terraform、基礎設施等）進行合理分類。

3. **規劃 `README.md` 結構**
   - **專案簡介 (Overview)**：簡述專案核心目的。
   - **目錄結構 (Project Structure)**：以清晰的樹狀圖呈現。
   - **技術指南與文件清單 (Documentation Index)**：分門別類列出各篇教學的連結與短評。
   - **協作與貢獻指南 (Contribution / Usage)**：簡述如何閱讀或使用本專案。

4. **執行更新與驗證**
   - 使用編輯工具更新 `README.md`。
   - 檢查 Markdown 格式、相對路徑正確性與排版美觀度。
