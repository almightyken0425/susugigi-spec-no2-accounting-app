# 匯入精靈: ImportWizardScreen

## 畫面目標

- 提供從 CSV 批次匯入交易或轉帳的引導式多步驟介面

---

## 觸發情境

- 以 Modal 呈現
- 由 DataManagementScreen 觸發，帶入匯入模式，匯入模式可為交易、轉帳
- 兩種模式共用同一套四步流程，僅各步驟內容隨模式微調

---

## 線框圖

### 選擇檔案步驟

```
┌──────────────────────────────────┐
│ 關閉            選擇檔案      前進 │
├──────────────────────────────────┤
│ 來源時區                         │
│        UTC+10:00                  │
│        UTC+09:00                  │
│  ====  UTC+08:00  ====（選中）    │
│        UTC+07:00                  │
│        UTC+06:00                  │
│                                  │
│        transactions_2026.csv      │  選後才出現
│         [    選擇檔案    ]        │
│ ──────────────────────────────   │
│         [    下載範本    ]        │
│         [    下載說明    ]        │
└──────────────────────────────────┘
```

### 欄位對應步驟

```
┌──────────────────────────────────┐
│ 返回            欄位對應      前進 │
├──────────────────────────────────┤
│ 日期 *                           │
│ [ date                       v ] │
│ 金額 *                           │
│ [ amount                     v ] │
│ 類別 *                           │
│ [ category                   v ] │
│ 帳戶 *                           │
│ [ account                    v ] │
│ 幣別 *                           │
│ [ currency                   v ] │
│ 備註                             │
│ [ note                       v ] │
└──────────────────────────────────┘
```

### 內容比對步驟

```
┌──────────────────────────────────┐
│ 返回            內容比對      前進 │
├──────────────────────────────────┤
│ 帳戶                             │
│ 玉山活儲 (TWD)                   │
│ [ 沿用                       v ] │
│ USD 旅費 (USD)                   │
│ [ 新建                       v ] │
│ 支出類別                         │
│ 飲食                             │
│ [ 沿用                       v ] │
│ 訂閱                             │
│ [ 新建                       v ] │
│ 收入類別                         │
│ 薪資                             │
│ [ 沿用                       v ] │
│ 獎金                             │
│ [ 新建                       v ] │
└──────────────────────────────────┘
```

### 預覽步驟

```
┌──────────────────────────────────┐
│ 返回            預覽匯入      送出 │
├──────────────────────────────────┤
│ 匯入摘要                         │
│ ┌──────────────────────────────┐ │
│ │ 共匯入                  124  │ │
│ │ 新增帳戶                  1  │ │
│ │ 新增類別                  1  │ │
│ │ 將略過紀錄數              8  │ │
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
```

---

## 佈局

### Header

- Header 模式: F
- 左動作
  - **IF** 選擇檔案步驟:
    - 關閉
  - **IF** 其餘步驟:
    - 返回
- 標題，隨步驟切換
  - 選擇檔案步驟: 選擇檔案
  - 欄位對應步驟: 欄位對應
  - 內容比對步驟: 內容比對
  - 預覽步驟: 預覽匯入
- 右動作
  - **IF** 預覽步驟:
    - 送出
  - **IF** 其餘步驟:
    - 前進
  - **IF** 當前步驟驗證未通過:
    - 不可點按

### 選擇檔案步驟

- List 模式: Custom
- 來源時區區塊
  - 來源時區 標題
  - 匯入資料的日期時間依所選來源時區解析 說明文字
  - 來源時區選擇器，常駐展開
    - 預設帶使用者目前偏好時區
    - 在此步驟內直接選擇，不另跳畫面
- 分隔線
- 檔案區塊
  - **IF** 已選擇檔案:
    - 檔案名稱
  - 選擇檔案 按鈕
- 下載區塊
  - 下載範本 按鈕
  - 下載說明 按鈕

### 欄位對應步驟

- 資料由 suggestColumnMapping 產出
- 欄位對應列表，每個系統欄位一列
  - 系統欄位名稱，見 CSV 欄位規格
  - 必填標記
  - CSV 欄位選擇器，顯示目前對應的 CSV 欄位，點按展開可選 CSV 欄位
    - **IF** 無符合格式的可用欄位:
      - 顯示無符合格式的欄位提示

### 內容比對步驟

- 資料由 analyzeImportContent 產出
- 比對的既有帳戶與類別限當前使用者的活躍紀錄，排除其他使用者與已軟刪項
- 帳戶段
  - 帳戶 標題
  - 帳戶比對列，每個 CSV 帳戶名稱加幣別一列
    - CSV 帳戶名稱與幣別
    - 同名但不同幣別的 CSV 帳戶，各自獨立一列
    - 動作選擇器
      - **IF** 與既有帳戶相符:
        - 沿用、新建、跳過
      - **IF** 無相符帳戶:
        - 新建、跳過
- **IF** 模式含收支類別資料:
  - 支出類別段
    - 支出類別 標題
    - 類別比對列，每個 CSV 支出類別名稱一列
      - CSV 類別名稱
      - 動作選擇器，規則同帳戶比對列
  - 收入類別段
    - 收入類別 標題
    - 類別比對列，每個 CSV 收入類別名稱一列
      - CSV 類別名稱
      - 動作選擇器，規則同帳戶比對列
  - 同名但分屬收入與支出的 CSV 類別，分別列於支出類別段與收入類別段

### 預覽步驟

- 匯入摘要
  - 共匯入紀錄數
  - 將新建帳戶數
  - **IF** 模式含收支類別資料:
    - 將新建類別數
  - 將略過紀錄數

---

## CSV 欄位規格

系統欄位定義，為下載範本、下載說明、欄位對應、驗證與匯出的單一依據。
欄位名稱、CSV 欄位順序、格式與範例於此定義。
各環節一律對齊，不另立格式。
欄位對應步驟的列顯示順序屬該步驟畫面安排，與此處 CSV 欄位順序未必相同。

下列順序即 CSV 欄位順序。

- **IF** 交易模式:
  - transaction_datetime，必填
    - Date and time, YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.
    - Date-only defaults the time to 00:00:00.
  - category，必填
    - Category name.
    - Income or expense follows the amount sign.
  - account，必填
    - Account name.
    - Matched together with its currency.
  - amount，必填
    - A positive value is income, a negative value is expense.
    - Use a period for decimals, up to 4 places.
    - No comma, thousands separator, or currency symbol.
  - currency，必填
    - ISO 4217 code, e.g. TWD, USD, JPY.
    - Case-insensitive.
  - note，可選
    - Free-text memo.
- **IF** 轉帳模式:
  - transfer_datetime，必填
    - Date and time, YYYY-MM-DD or YYYY-MM-DD HH:MM:SS.
    - Date-only defaults the time to 00:00:00.
  - from_account，必填
    - Source account name.
    - Matched together with its currency.
  - from_currency，必填
    - Source ISO 4217 code, e.g. TWD.
    - Case-insensitive.
  - from_amount，必填
    - Source amount.
    - Use a period for decimals, up to 4 places.
    - No comma, thousands separator, or currency symbol.
  - to_account，必填
    - Destination account name.
    - Matched together with its currency.
  - to_currency，必填
    - Destination ISO 4217 code.
    - Case-insensitive.
  - to_amount，可選
    - Destination amount.
    - Needed when the two currencies differ.
  - note，可選
    - Free-text memo.

### 範例列

下載範本除標頭外附範例列，示意合法格式。
範例值一律符合上述欄位格式，使範本可直接作為匯入輸入。

- **IF** 交易模式:
  - 2026-01-21 12:30:00,Food,Cash,-150.00,TWD,Lunch
  - 2026-01-22 09:00:00,Salary,Bank,50000.00,TWD,January salary
- **IF** 轉帳模式:
  - 2026-01-21 14:00:00,Cash,TWD,10000.00,Bank,TWD,10000.00,Deposit
  - 2026-01-22 10:30:00,Bank,TWD,30000.00,USD account,USD,1000.00,FX exchange

---

## 互動

- **點按關閉:**
  - 關閉 Modal

- **點按返回:**
  - 返回上一步驟

- **點按前進:**
  - 前進至下一步驟

- **選擇來源時區:**
  - 更新匯入解析所用的來源時區

- **點按選擇檔案按鈕:**
  - 開啟系統檔案選擇器
  - **IF** 選擇非 CSV 檔案:
    - 顯示僅支援 CSV 格式對話框
  - **IF** 讀取檔案錯誤:
    - 顯示讀取檔案失敗對話框
  - **IF** 選擇有效 CSV:
    - 呼叫 parseCsvFile
    - 顯示檔案名稱

- **點按下載範本按鈕:**
  - 呼叫 shareTemplate
  - **IF** 操作失敗:
    - 顯示下載失敗對話框

- **點按下載說明按鈕:**
  - 呼叫 shareInstructions
  - **IF** 操作失敗:
    - 顯示下載失敗對話框

- **選擇 CSV 欄位:**
  - 更新該系統欄位的對應

- **選擇比對動作:**
  - 更新該帳戶或類別的匯入動作

- **點按送出:**
  - 呼叫 executeImport
  - **IF** 操作中:
    - 顯示載入狀態
  - **IF** 操作成功:
    - 顯示已匯入筆數與略過筆數對話框
    - 使用者點擊確認後關閉 Modal
  - **IF** 操作失敗:
    - 顯示匯入失敗對話框
