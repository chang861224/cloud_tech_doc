---
name: tech-doc-writer
description: '協助撰寫、編修與優化高品質的技術文件、Markdown 教學、系統設計或 API 規格書。當需要：規劃文件大綱、撰寫技術手冊、檢查 Markdown 格式、優化程式碼區塊時使用。'
argument-hint: '想要撰寫的文件主題、大綱或檔案路徑'
user-invocable: true
---

# Technical Documentation Writer (技術文件寫作助手)

本 Skill 提供了一套結構化的標準寫作流程，協助您從零開始規劃、撰寫、編修並格式化專業的技術文件與 Markdown 教學。

## 適用場景
- 撰寫新技術的教學（如 Terraform、Docker、Kubernetes、Python 等）
- 建立系統架構說明、設計規格書或 API 文件
- 優化現有 Markdown 文件的結構、語意與排版格式
- 確保技術名詞正確性與程式碼區塊的完整性

---

## 標準寫作流程

### 步驟 1：規劃文件大綱 (Outline & Planning)
在開始撰寫內文之前，必須先與使用者確認以下關鍵資訊：
1. **目標讀者 (Target Audience)**：讀者具備什麼先備知識？（例如：初學者、DevOps 工程師、後端開發者）
2. **文件目的 (Objective)**：讀者看完後能學會什麼或解決什麼問題？
3. **大綱草案 (Draft Outline)**：擬定 1~3 級標題。
> **注意**：大綱確認後才進入步驟 2。

### 步驟 2：內文撰寫與結構化 (Drafting)
1. **套用範本**：優先參考內建的 [文件通用範本](./assets/template.md)。
2. **圖文並茂**：適時使用 Mermaid 流程圖、表格與條列式清單來輔助說明。
3. **深入淺出**：在程式碼區塊或配置設定（如 Terraform `main.tf`）前後提供清晰的口語化解釋。

### 步驟 3：格式、風格與程式碼檢查 (Format & Style Check)
在完成初稿後，進行以下品質檢查：
1. **Markdown 格式規則**：嚴格遵守 [Markdown 風格指南](./references/markdown-rules.md)。
2. **專用術語與中英混排**：
   - 英文與中文之間保留空格（例如：「使用 Terraform 部署」而不是「使用Terraform部署」）。
   - 專有名詞大小寫正確（如：GitHub, Terraform, Node.js, AWS）。
3. **程式碼區塊語法高亮 (Syntax Highlighting)**：
   - 務必為所有程式碼區塊指定語言（如 ````terraform````, ````bash````, ````markdown````, ````yaml````）。
   - 程式碼區塊內加入註解以解釋核心邏輯。
4. **大標題貼圖**：在每個 H2 標題前新增一個簡單的 emoji 或貼圖來標示文件主題。

---

## 相關資源
- [Markdown 通用文件範本](./assets/template.md)
- [Markdown 風格與排版指南](./references/markdown-rules.md)
