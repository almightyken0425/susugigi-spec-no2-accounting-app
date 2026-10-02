# List Policy 列表共用元件政策

## 政策定位

- 本 module 內所有 screen 共用的列表政策
- 統一所有 list 場景的模式分類與互動行為
- screen spec 只標模式代號，不重述 row 細節
- 視覺定案由對側 Design git 的 `10_foundations/` 與 `20_components/` 承載
- 實作元件、平台 API、套件選用由對側 Impl git 的 `CLAUDE.md` UI Coding Guideline 承載
- 高度客製化的特殊資料列表標 Custom，不套用本政策的模式分類；
- 狀態行為仍受互動規則約束

---

## 模式分類

- **模式 A — Navigation 列表:**
    - **適用:**
        - 設定主頁
        - 偏好設定
        - 在地化設定
        - 資料管理
        - 貨幣清單
        - 匯率紀錄清單
    - **元件條目順序:**
        - leading icon
        - 主標
        - 副標可選
        - trailing 值文字可選
        - 後綴 chevron
    - **互動:**
        - 點擊跳轉至下層 screen 或開啟 modal
        - 啟用按下回饋

- **模式 B — Selection 列表:**
    - **B-1 單列適用:**
        - 語言設定
        - 啟動模式
        - 時區
        - 主要貨幣
        - Account Editor 內幣別 picker
        - Category Editor 內收支 picker
        - Currency Detail Config 小數位數
    - **B-2 網格適用:**
        - 主題設定
        - 首頁篩選帳戶 visibility-toggle 變體
    - **B-1 元件條目順序:**
        - leading icon 可選
        - 主標
        - 副標可選
        - trailing 勾選圖示
    - **B-2 元件條目順序:**
        - 預覽區
        - 名稱
        - 勾選圖示 overlay
    - **互動:**
        - 單選或多選
        - 選中以勾選圖示標示
        - 啟用按下回饋
        - 多選用法不變更 trailing 樣式

    - **B-2 visibility-toggle 變體:**
        - 適用 HomeFilter 帳戶選取 grid
        - 每格內含 icon 與名稱
        - 以選中狀態與未選中狀態的視覺差異標示
        - 不使用 勾選圖示 overlay
        - 點 tile 切換選取狀態
        - 最後一個 selected 不可取消

- **模式 C — 簡化資料列表:**
    - **適用:**
        - 直接顯示一條條資料且可點擊跳編輯頁
        - row 結構接近 A 模式
    - **說明:**
        - 視為 A 模式變體
        - 含金額換算、多幣別、分組折疊、搜尋 highlight 等複雜資料列表標為 Custom 不套用

- **模式 D — Reorderable 列表:**
    - **適用:**
        - 帳戶清單
        - 類別清單
    - **元件條目順序:**
        - leading icon
        - 主標
        - 副標可選
    - **互動:**
        - 長按拖拉排序
        - 點擊整列導向編輯
        - 停用按下回饋，避免與拖拉手勢衝突

- **Custom Layout 例外:**
    - **適用:**
        - 高度客製化資料列表
        - 例如首頁交易 pager
        - 例如 Period 交易列表含多幣別匯率換算與分組折疊
        - 例如搜尋結果含關鍵字 highlight
        - 例如匯入精靈
    - **規範:**
        - screen spec 標 `List 模式: Custom`
        - 視覺仍對齊對側 Design git 的 list token 群組
        - 含搜尋或空狀態時，空狀態與載入呈現依互動規則的空狀態與空狀態切換節，不另定義

---

## 互動規則

- **按下回饋:**
    - A / B 模式啟用
    - D 模式停用，避免與拖拉手勢衝突

- **Disabled 樣式:**
    - 整列是否套 disabled 視覺弱化由引用端決定
    - 按下回饋停用

- **Selected 樣式:**
    - B-1 單列 trailing 顯勾選圖示
    - B-1 selected 採主色
    - B-2 網格右上以 overlay 標示選中

- **空狀態:**
    - 任何含搜尋的列表必顯空狀態
    - 搜尋無結果採 title 為 `common.no_results`
    - 搜尋無結果採 description 為 `「{searchQuery}」`
    - 初始無資料採該 screen 對應的描述性訊息
    - 載入中不顯示空狀態

- **空狀態切換:**
    - 含群組卡片的搜尋列表，兩態切換以動畫互換避免硬切
    - empty 態不渲染卡片底色避免視覺殘留
    - 切換期間舊態不接受互動

---

## screen spec 引用方式

- **要求:**
    - 各 screen spec 在 list 區段開頭標註 `List 模式: A` 此類代號
    - 不重述 row 結構
    - 不重述視覺定案

- **合規寫法:**

```
### List

- List 模式: A
- 列項
  - leading icon 槽位
  - 主標 = i18n 鍵 settings.manage_categories
  - 後綴 chevron 跳轉至 CategoryList
```

- **組合寫法:**
    - 含多種模式的 screen 用 `+` 連接，例如 `List 模式: A+B-1+B-1`
    - 純 Custom Layout 標 `List 模式: Custom` 並補充說明 row 結構
