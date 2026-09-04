# 專案：百年歷史地圖 (Web GIS)

## 語言與交互規範 (Token-Saving Rules)
- **語言偏好：** 一律使用繁體中文（台灣習慣用語）溝通與註解。
- **輸出極小化 (Diff Only)：**
  - 嚴禁重印未修改的完整程式碼檔案，優先使用 `Edit` 工具進行局部修改。
  - 對話開頭切勿使用客套話或重複問題；任務完成後僅回報：異動檔案、修改行數、測試結果。
  - 嚴禁主動載入 `data/layers.bundle.json` 等超大打包檔，避免爆衝上下文 (Context bloat)。

## 常用指令 (Commands)
- 本機啟動：`.\start-website.bat` 或 `npx serve`
- 全域測試：`node tests/run-all.mjs`
- 單元測試：`node tests/run-all.mjs tests/specs/<test-file>.mjs`
- 圖資打包：`node tools/build-layers-bundle.js`
- 圖層類型自動打標：`node tools/tag-layer-types.js`（以 title/keywords/階層繼承自動判定 type，新增圖層後、打包 bundle 前執行）

## 子代理分工與路由 (Subagents Routing)
遇到具體模組需求時，主代理請即刻將任務派發給對應的 Subagent，勿在主階段載入過多非權責程式碼：

| 任務領域 | 調度代理 | 權責檔案邊界 |
| :--- | :--- | :--- |
| 地圖底層、圖磚容錯、座標換算 | `map-core-agent` | `src/mapCore.js`, `src/core/`, `src/tileChecker.js`, `src/geocode.js` |
| 介面樣式、RWD、側邊欄、時間軸滑桿 | `ui-frontend-agent` | `index.html`, `style.css`, `src/ui/`, `src/timelineUI.js`, `src/sidebarUI.js` |
| 模式切換、雙圖比對、繪圖工具、Store | `feature-state-agent` | `src/features/`, `src/store.js`, `src/drawTool.js` |
| 圖層 JSON、地名映射、圖資打包 | `data-processing-agent` | `data/layers/`, `data/historical-names.json`, `tools/` |
| 整合回歸測試、品質把關 | `qa-testing-agent` | `tests/` |

## 開發守則與防護 (Guardrails)
1. **原生 ESM 架構：** 保持純原生 JavaScript ES Module，非必要絕不安裝任何重型 npm 第三方依賴。
2. **資料管線同步：** 凡異動 `data/layers/*.json`（尤其新增圖層），完成後必須先執行 `node tools/tag-layer-types.js` 自動打標 type，再執行 `node tools/build-layers-bundle.js` 重新打包。
3. **驗證先行：** 所有邏輯或狀態修改，結束前必須執行對應的測試檔確認通過，嚴禁留下未驗證的 break changes。
