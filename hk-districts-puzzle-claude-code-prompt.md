# Claude Code Prompt：香港十八區精確地圖拼圖

## 任務說明

請幫我建立一個**香港十八區互動拼圖遊戲**，以單一 HTML 檔案輸出，使用 D3.js 渲染官方 GeoJSON 精確行政區邊界。

---

## 技術要求

### 資料來源
1. 優先嘗試 fetch 以下 GeoJSON（含 fallback 邏輯）：
   - `https://raw.githubusercontent.com/hkdistrict-data/district-json/main/18districts.json`
   - 若失敗，嘗試 DATA.GOV.HK 官方 API
   - 若皆失敗，提示使用者並顯示降級說明

2. GeoJSON feature 的 `properties` 欄位中，識別行政區名稱的 key（可能是 `ENAME`、`CNAME`、`District` 等），需先 `console.log` 確認再綁定

### 地圖渲染（D3.js v7）
```javascript
// 核心流程
const projection = d3.geoMercator().fitSize([width, height], geojsonData);
const pathGenerator = d3.geoPath().projection(projection);

svg.selectAll('path')
  .data(geojsonData.features)
  .enter()
  .append('path')
  .attr('d', pathGenerator)
  .attr('id', d => 'district_' + normalizeId(d.properties.ENAME))
```

---

## 行政區 Metadata

以下資料需 hardcode 在 JS 中，依 district ID 對應：

```javascript
const DISTRICT_DATA = {
  'central_western': {
    name: '中西區', eng: 'Central and Western', region: 'hk',
    color: '#E8A0A0', pop: '240,700', area: '12.54 km²',
    desc: '香港政治及商業核心，設有中環金融區及香港特別行政區政府總部。'
  },
  'wan_chai': {
    name: '灣仔區', eng: 'Wan Chai', region: 'hk',
    color: '#E89090', pop: '180,800', area: '9.83 km²',
    desc: '香港會議展覽中心、香港藝術中心等地標雲集，著名夜生活區。'
  },
  'eastern': {
    name: '東區', eng: 'Eastern', region: 'hk',
    color: '#D87878', pop: '556,400', area: '18.85 km²',
    desc: '香港島人口最多的區域，北角、鰂魚涌、筲箕灣等地。'
  },
  'southern': {
    name: '南區', eng: 'Southern', region: 'hk',
    color: '#C86060', pop: '277,700', area: '38.85 km²',
    desc: '香港面積最大的島區，赤柱、香港仔、鴨脷洲，漁村文化著稱。'
  },
  'yau_tsim_mong': {
    name: '油尖旺區', eng: 'Yau Tsim Mong', region: 'kl',
    color: '#9090D8', pop: '355,800', area: '6.99 km²',
    desc: '油麻地、尖沙咀、旺角三大核心，香港最繁忙的都市區域。'
  },
  'sham_shui_po': {
    name: '深水埗區', eng: 'Sham Shui Po', region: 'kl',
    color: '#7878C8', pop: '424,700', area: '9.36 km²',
    desc: '九龍西部，以電子產品街、布料批發聞名，基層社區縮影。'
  },
  'kowloon_city': {
    name: '九龍城區', eng: 'Kowloon City', region: 'kl',
    color: '#6060B8', pop: '418,800', area: '10.0 km²',
    desc: '土瓜灣、紅磡、馬頭圍，啟德郵輪碼頭及舊啟德機場遺址。'
  },
  'wong_tai_sin': {
    name: '黃大仙區', eng: 'Wong Tai Sin', region: 'kl',
    color: '#5050A8', pop: '420,200', area: '9.30 km²',
    desc: '以黃大仙祠著稱，香港重要宗教聖地，大量公共屋邨。'
  },
  'kwun_tong': {
    name: '觀塘區', eng: 'Kwun Tong', region: 'kl',
    color: '#4040A0', pop: '664,800', area: '11.27 km²',
    desc: '香港人口最多的區域，由工業區轉型為商業及住宅區。'
  },
  'kwai_tsing': {
    name: '葵青區', eng: 'Kwai Tsing', region: 'nt',
    color: '#60C878', pop: '520,700', area: '23.34 km²',
    desc: '葵青貨櫃碼頭是全球最繁忙貨櫃港之一，青衣島連接。'
  },
  'tsuen_wan': {
    name: '荃灣區', eng: 'Tsuen Wan', region: 'nt',
    color: '#50B868', pop: '332,700', area: '307.18 km²',
    desc: '新界首批新市鎮，大帽山為香港最高峰，郊野活動勝地。'
  },
  'tuen_mun': {
    name: '屯門區', eng: 'Tuen Mun', region: 'nt',
    color: '#40A858', pop: '494,700', area: '82.89 km²',
    desc: '新界西北重要新市鎮，黃金海岸、青山禪院，輕鐵系統。'
  },
  'yuen_long': {
    name: '元朗區', eng: 'Yuen Long', region: 'nt',
    color: '#309848', pop: '655,000', area: '138.46 km²',
    desc: '天水圍、米埔自然保護區，農業傳統及圍村文化著稱。'
  },
  'north': {
    name: '北區', eng: 'North', region: 'nt',
    color: '#208838', pop: '327,000', area: '136.61 km²',
    desc: '毗鄰深圳邊境，上水、粉嶺、沙頭角，落馬洲管制站。'
  },
  'tai_po': {
    name: '大埔區', eng: 'Tai Po', region: 'nt',
    color: '#50A870', pop: '316,400', area: '136.15 km²',
    desc: '大埔新市鎮、吐露港，以海鮮、單車徑及文物館聞名。'
  },
  'sha_tin': {
    name: '沙田區', eng: 'Sha Tin', region: 'nt',
    color: '#40986A', pop: '692,500', area: '68.71 km²',
    desc: '沙田馬場舉世聞名，城門河步道宜人，香港中文大學所在地。'
  },
  'sai_kung': {
    name: '西貢區', eng: 'Sai Kung', region: 'nt',
    color: '#308860', pop: '479,700', area: '129.65 km²',
    desc: '香港後花園，清水灣、橋咀洲等絕美海岸線，郊野公園。'
  },
  'islands': {
    name: '離島區', eng: 'Islands', region: 'nt',
    color: '#208850', pop: '179,800', area: '175.12 km²',
    desc: '大嶼山、南丫島、長洲，香港國際機場及迪士尼樂園所在地。'
  }
};
```

---

## 功能規格

### 拼圖模式
- 地圖初始所有區域顯示淡藍色空白輪廓（`fill: #B8D4E8`）
- 下方 / 側邊欄列出 18 個行政區名稱 chip（隨機排列）
- 流程：點選 chip → chip 高亮 → 點選地圖區域 → 判斷
  - 正確：填色 + 顯示標籤 + 右側面板顯示資訊 + chip 變灰
  - 錯誤：地圖區域紅色抖動動畫 (`@keyframes shake`)，嘗試次數 +1
- 18 區全部完成 → 顯示完成畫面（恭喜 + 嘗試次數）

### 學習模式
- 全部填色顯示 + 顯示所有區名標籤
- 點擊任一區 → 右側面板顯示詳細資訊
- chip tray 隱藏

### 區名標籤
- 使用 `d3.geoCentroid()` 計算各區中心點
- 在 SVG 上疊加 `<text>` 元素
- 字體大小依地圖縮放動態調整

### 右側資訊面板
顯示：中文名、英文名、大區 badge（香港島/九龍/新界）、人口、面積、描述

### 進度追蹤
- 頂部顯示：已完成 N / 18 + 進度條
- 嘗試次數計數器
- 重置按鈕、顯示答案按鈕

---

## 視覺設計方向

沿用現有 prototype 的風格：
- 背景色：`#F5EDD8`（米色紙質感）
- 主色：`#1A1208`（深墨色）
- 強調色：`#D4A017`（金色）
- 字體：`Noto Serif TC`（Google Fonts）+ `DM Mono`
- 整體風格：復古印刷 / 地圖冊美學

色彩分區：
- 香港島：暖紅 `#C86060 ~ #E8A0A0`
- 九龍：藍紫 `#4040A0 ~ #9090D8`
- 新界：綠色 `#208850 ~ #60C878`

---

## 輸出規格

- **單一 `.html` 檔案**，可直接在瀏覽器開啟
- D3.js 從 CDN 引入：`https://d3js.org/d3.v7.min.js`
- Google Fonts 從 CDN 引入
- GeoJSON 資料透過 `fetch()` 取得（非 embed），若 fetch 失敗需顯示友善錯誤提示
- 不需要任何後端、Node.js、或 build 工具

---

## 注意事項

1. **GeoJSON 欄位確認**：fetch 後先 `console.log(features[0].properties)` 確認欄位名稱，再對應 DISTRICT_DATA
2. **ID 對應邏輯**：建立 `ENAME → id` 的 mapping table，因為 GeoJSON 英文名稱可能與 key 不完全一致
3. **投影設定**：使用 `fitSize` 讓地圖自動填滿容器，不要 hardcode 座標
4. **響應式**：地圖容器寬度 100%，高度依比例計算
5. **標籤避免重疊**：字體大小約 8–10px，過小的區域（如油尖旺）可縮小或隱藏標籤

---

## 參考資料

- GeoJSON 來源：`https://github.com/hkdistrict-data`
- D3 Geo API：`https://d3js.org/d3-geo`
- 現有 prototype（簡化版）：參考互動邏輯與視覺風格
