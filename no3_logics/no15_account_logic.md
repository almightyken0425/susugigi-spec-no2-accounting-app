# 帳戶核心邏輯: AccountLogic

## createAccount 建立帳戶

- **輸入:**
  - 帳戶資料
- 匯入新建的名稱處理由 [DataTransferLogic](no21_data_transfer_logic.md) 的 executeImport 承載
- **IF** 帳戶名稱去除前後空白後長度超過名稱長度上限:
  - **回傳:** 驗證失敗，原因為名稱過長
  - 作為一般建立路徑的名稱長度驗證後備
- **寫入 Account:**
  - **條件:**
    - `iconId` 必須引用 `tags` 含 `account` 的 `IconDefinitions` 記錄
  - **執行:**
    - 新增一筆記錄至 `Accounts` 表
- **IF** 幣別非主要貨幣:
  - 呼叫 createInitialCurrencyRate 種入佔位匯率

## updateAccount 更新帳戶

- **輸入:**
  - 帳戶資料
- **IF** 帳戶名稱去除前後空白後長度超過名稱長度上限:
  - **回傳:** 驗證失敗，原因為名稱過長
  - 作為一般更新路徑的名稱長度驗證後備
- **更新 Account:**
  - **執行:**
    - 更新 `Accounts` 表中的記錄

## deleteAccount 刪除帳戶

- **輸入:**
  - 帳戶識別碼
- **軟刪除 Account:**
  - **執行:**
    - 更新 `Accounts` 表
- **串聯軟刪除 Transaction:**
  - **執行:**
    - 更新該帳戶所屬的 `Transactions` 表記錄
- **串聯軟刪除 Transfer:**
  - **執行:**
    - 更新該帳戶作為轉出方或轉入方的 `Transfers` 表記錄

## reorderAccounts 重排帳戶

- **輸入:**
  - 有序的帳戶識別碼清單
- **更新 Account:**
  - **執行:**
    - 批次更新 `Accounts` 表的排序欄位
