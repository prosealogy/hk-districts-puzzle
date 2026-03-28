# PRD：香港十八區精確邊界地圖拼圖

**文件版本**：v1.0  
**日期**：2026-03-28  
**作者**：Sean  
**狀態**：Draft

---

## 1. 背景與目標

### 問題陳述
現有的香港行政區教學工具缺乏互動性，且多為靜態圖片或簡化多邊形，無法提供準確的地理邊界認知體驗。

### 目標
打造一個基於**官方 GeoJSON 精確邊界**的香港十八區互動拼圖遊戲，讓使用者透過遊戲化方式學習香港各行政區的地理位置與基本資訊。

### 成功指標
- 地圖邊界與官方資料一致（使用 DATA.GOV.HK 或同等可信來源）
- 使用者可在單一 HTML 檔案內完整運行，無需後端
- 支援拼圖模式（互動學習）與學習模式（查閱資訊）

---

## 2. 使用者故事

| 角色 | 需求 | 目的 |
|------|------|------|
| 一般使用者 | 透過拼圖方式認識香港 18 區位置 | 學習地理知識 |
| 旅遊者 | 查詢各區特色、位置關係 | 行程規劃參考 |
| 學生 | 快速記憶行政區分佈 | 考試複習 |

---

## 3. 功能規格

### 3.1 核心功能

#### 拼圖模式（Puzzle Mode）
- 地圖底圖顯示 18 個空白區域輪廓（精確 GeoJSON 邊界）
- 下方或側邊提供 18 個行政區名稱標籤（隨機排列）
- 使用者點選名稱 → 點選地圖區域 → 判斷正確 / 錯誤
  - **正確**：填入該區代表色 + 顯示區名標籤 + 右側面板顯示資訊
  - **錯誤**：視覺閃爍提示（紅色抖動），累計錯誤次數
- 完成 18 區 → 顯示完成畫面（用時、嘗試次數）

#### 學習模式（Learn Mode）
- 一次顯示所有行政區填色與名稱
- 點擊任一區域 → 右側面板顯示詳細資訊
- 可作為拼圖完成後的複習工具

### 3.2 行政區資訊面板
每個行政區顯示：
- 中文名稱、英文名稱
- 所屬大區（香港島 / 九龍 / 新界）
- 人口數（最新普查數據）
- 面積（km²）
- 代表性地標或特色描述（1–2 句）

### 3.3 進度追蹤
- 頂部進度條：已完成 N / 18
- 嘗試次數計數器
- 重置功能

### 3.4 視覺色彩系統
| 大區 | 色系 |
|------|------|
| 香港島（4 區） | 暖紅色系 |
| 九龍（5 區） | 藍色系 |
| 新界（9 區） | 綠色系 |

---

## 4. 技術規格

### 4.1 技術選型

| 項目 | 選擇 | 理由 |
|------|------|------|
| 地圖渲染 | **D3.js v7** | 直接將 GeoJSON 轉為 SVG path，可精確控制每個區塊的互動 |
| GeoJSON 來源 | DATA.GOV.HK 官方或 `hkdistrict-data` GitHub | 精確行政邊界，公開授權 |
| 地圖投影 | `d3.geoMercator()` | 符合香港常見地圖呈現方式 |
| 打包方式 | 單一 HTML 檔案（inline JS + CSS） | 無需伺服器，直接開啟即用 |
| 外部依賴 | D3.js（CDN）、Google Fonts | 最小化外部依賴 |

### 4.2 GeoJSON 資料來源優先順序
1. **第一選擇**：`https://raw.githubusercontent.com/hkdistrict-data/district-json/main/18districts.json`
2. **備用**：DATA.GOV.HK district boundary dataset（需轉換格式）
3. **Fallback**：hardcode 簡化座標（僅作降級處理）

### 4.3 資料結構
```json
{
  "id": "wan_chai",
  "name": "灣仔區",
  "eng": "Wan Chai",
  "region": "hk",
  "color": "#E89090",
  "population": "180,800",
  "area": "9.83 km²",
  "landmark": "香港會議展覽中心、金紫荊廣場",
  "desc": "..."
}
```

### 4.4 核心技術實作

```
GeoJSON fetch
  → d3.geoMercator().fitSize([width, height], geojsonData)
  → d3.geoPath(projection)
  → svg.selectAll('path').data(features).enter()
  → 每個 feature 綁定 district metadata
  → 互動事件（click handler）
```

### 4.5 響應式設計
- 桌面版：左側地圖 + 右側資訊面板
- 手機版：上方地圖 + 下方資訊面板（垂直排列）
- SVG viewBox 自適應容器寬度

---

## 5. 非功能需求

| 項目 | 規格 |
|------|------|
| 效能 | 首次載入 < 3 秒（含 GeoJSON fetch） |
| 瀏覽器支援 | Chrome / Firefox / Safari 最新版 |
| 離線能力 | GeoJSON 可嵌入 HTML，實現完全離線 |
| 無障礙 | 顏色對比度符合 WCAG AA，區塊有 aria-label |

---

## 6. 範圍外（Out of Scope）

- 後端 / 資料庫
- 使用者帳號、分數排行榜
- 多語言（英文版）
- 區界的即時更新機制
- 子區域（如選區）顯示

---

## 7. 里程碑

| 階段 | 內容 | 預計完成 |
|------|------|----------|
| M1 | GeoJSON 取得與 D3 渲染驗證 | Sprint 1 |
| M2 | 拼圖核心邏輯（選取、比對、判斷） | Sprint 1 |
| M3 | 資訊面板、學習模式 | Sprint 2 |
| M4 | 視覺優化、響應式、細節打磨 | Sprint 2 |

---

## 8. 參考資料

- [DATA.GOV.HK - 香港地區邊界](https://data.gov.hk/tc-data/dataset/hk-had-HKSAR_Area_Boundary)
- [hkdistrict-data GitHub](https://github.com/hkdistrict-data)
- [D3.js Geo Documentation](https://d3js.org/d3-geo)
- [現有 prototype](./hk-districts-puzzle.html)（簡化多邊形版本）
