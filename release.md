# Release Notes

本檔案記錄本專案各版本的更新項目與異動日誌。

---

## [v1.1.0] - 2026-10-09 - 新增實作執行 Skill 與專案版號發布追蹤機制

### ✨ 新增功能 (New Features)
- **個人層級實作執行 Skill (`implementation-executor`)**: 
  - 建立專屬 Skill，支援依據 `specs/` 目錄下的開發計畫（Dev Plan）進行自動化或逐步實作、程式碼編寫與更新。
  - 自動整合專案導覽更新與 Release Notes 維護流程。
- **發布日誌追蹤 (`release.md`)**:
  - 建立專案根目錄的 `release.md` 發布紀錄檔案，規範以 Append 方式於最上方新增最新版次。

### 📝 文件維護 (Documentation Updates)
- 更新 `README.md` 目錄結構，將 `.github/` 與 `release.md` 納入專案架構導覽。
