# Search Policy 搜尋列共用元件政策

## 政策定位

- 本 module 內所有含搜尋輸入的 screen 共用政策
- 統一搜尋列位置與互動行為
- screen spec 只標模式代號，不重述輸入框細節
- 視覺定案由對側 Design git 的 `10_foundations/` 與 `20_components/` 承載
- 實作元件、平台 API、套件選用由對側 Impl git 的 `CLAUDE.md` UI Coding Guideline 承載

---

## 模式分類

- **模式 Bottom Pill — 底部固定 dock:**
    - **適用:**
        - 搜尋頁
        - 主要貨幣設定
        - 貨幣清單
        - 匯率紀錄清單
        - 語系設定
        - 時區設定
    - **元件條目順序:**
        - 前綴搜尋 icon 槽位
        - 中央輸入框
            - placeholder 文案 = 搜尋
            - 各搜尋畫面共用同一通用文案，不逐畫面特化
        - 後綴 clear 動作
            - **IF** 輸入框 focus:
                - 顯示
    - **容器變體:**
        - 底部 dock 情形固定在畫面底部
        - modal 情形內嵌於 modal 頂部，隨 modal 開關出現消失
        - 兩種容器共用同一搜尋列結構與互動行為
    - **互動:**
        - 底部 dock 情形固定底部；modal 變體隨 modal 容器定位
        - 不隨 scroll 隱藏
        - 底部 dock 情形隨鍵盤上推、鍵盤收起落回底部；modal 變體固定於 modal 頂部、不隨鍵盤移動
        - 點擊輸入框觸發 focus
        - 輸入內容時即時通知引用端，篩選或搜尋的執行時機依引用畫面規格
        - clear 動作一鍵清空

---

## 互動規則

- **autoFocus 規則:**
    - 搜尋頁進入時 autoFocus
    - 其餘搜尋畫面進入時不 autoFocus

- **Clear 動作:**
    - 僅 focus 時顯示
    - 點擊清空輸入並重新觸發引用端篩選

- **Scroll 行為:**
    - 搜尋頁列表滾動時收起鍵盤
    - 滾動期間保留點擊事件可命中列項

---

## screen spec 引用方式

- **要求:**
    - 各 screen spec 在搜尋列段落標註 `Search 模式: Bottom Pill`
    - 不重述 pill 結構
    - 不重述視覺定案
    - 不重述 placeholder 文案
    - 不重述 autoFocus 行為

- **合規寫法:**

```
### 搜尋輸入區

- Search 模式: Bottom Pill
```
