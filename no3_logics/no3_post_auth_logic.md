# 身分後初始化流程: PostAuthLogic

## handlePostAuth 處理身分後初始化

- 身分就緒後依本機與雲端資料是否存在分派初始化路徑
- 身分來源為匿名建立、熱啟動沿用、或 legacy 登入帳號升級沿用，路徑一致
- 全 user 一致、無條件執行，不分訂閱等級
- Premium 狀態刷新由 PremiumLogic 的 refreshStatus 承載，不在本流程內，bootstrap 不等待 IAP 服務回應
- 資料清除中斷復原命中時整段跳過，不得重建剛刪除的雲端文件，詳見 DeleteUserAccountLogic
- **執行:**
  - 讀取 Firestore `users/{uid}` 文件的存在性
  - 呼叫 initializeLocalUser，分別確認目前 UID 的 Users 與 Settings，僅補建缺少的資料
  - **IF** 本機資料查詢或補建失敗:
    - 停止後續初始化並回報失敗，交由 AnonymousBootstrapLogic 提供重試
  - **ELSE:**
    - 呼叫 initializeDefaultEntities
    - **IF** 雲端文件不存在:
      - 呼叫 initializeCloudUser
    - **ELSE:**
      - 委派 PreferenceUploadLogic 的 uploadAllPreferences，上傳本機實際值並接受覆寫雲端
  - 不再讀取並套用雲端 preference，不依文件存在與否決定是否下載

---

## initializeLocalUser 初始化本機使用者資料

- 沿用目前 Auth UID，分別確認 Users 與 Settings，保留已存在的資料並補齊缺少的項目
- **性質:**
  - 純本機，無雲端動作
- **確認本機資料:**
  - **執行:**
    - 分別查詢目前 UID 的 Users 與 Settings
    - 成功查詢且確認沒有資料，才判定該項缺少
    - 查詢失敗不視為資料不存在，停止本輪補建並回報失敗
    - 其他 UID 的資料不作為目前 UID 已完成初始化的依據
- **補建與保留:**
  - **行為:**
    - 有 Users、沒有 Settings：沿用 Users，依首次使用規則建立 Settings
    - 沒有 Users、有 Settings：以目前 UID 建立 Users，完整保留既有 Settings
    - 兩者都沒有：以目前 UID 建立 Users 與 Settings
    - 兩者都有：直接沿用，不重新初始化
    - 補建不建立新的 Auth 身分，不更換 UID
    - 不因補建清除或重建既有帳戶、類別、收支、轉帳與排程
    - 既有資料的主鍵、UID 歸屬與引用關係保持
    - 預設帳戶與類別是否需要建立，仍由 initializeDefaultEntities 依其存在條件判定
- **決定預設值:**
  - **條件:**
    - 已確認目前 UID 沒有 Settings
  - **執行:**
    - 主要貨幣:
      - 從裝置 Locale 推導
      - 若無則預設 TWD，並轉換為 Currency ID
    - 語系:
      - 將裝置 Locale 的完整語系代碼與支援語系清單比對
      - 完整代碼相符才採該語系
      - 比對未匹配則 fallback 預設 en
      - 不做語族內 fallback，避免繁簡混雜或非預期語系出現
    - 時區:
      - 讀取裝置時區
    - 主題:
      - 預設 theme1
    - 啟動模式:
      - 預設 home
    - 週起始日:
      - 預設 auto，代表依使用者語系慣例決定，不做 Locale 推導
- **建立本機資料:**
  - **執行:**
    - 僅新增已確認缺少的記錄
    - 新建 Users 的 id 對應目前 Auth UID，其他欄位依本機使用者模型
    - 新建 Settings 的 userId 對應相同 UID，其他欄位依設定模型與下列預設值
    - 既有 Settings 的主鍵、全部欄位及時間戳保持原值，不套用裝置或雲端偏好
  - **欄位:**
    - `baseCurrencyId`: 依主要貨幣決定
    - `language`: 依語系決定
    - `timeZone`: 依時區決定
    - `theme`: theme1
    - `launchMode`: home
    - `weekStart`: auto
- **失敗與重試:**
  - **行為:**
    - 補建失敗時保留原有資料，也保留本輪已成功建立的資料
    - 回報初始化失敗，由 AnonymousBootstrapLogic 結束載入並提供重試
    - 重試時重新查詢目前 UID 的 Users 與 Settings，只補仍然缺少的項目
    - 程序中斷後再次啟動也依持久資料判斷，不刪除已完成項目重做
    - 兩項都已存在時，重試不新增重複資料
- **清除恢復優先:**
  - **行為:**
    - 每次啟動與重試先依 AnonymousBootstrapLogic 判斷資料清除持久旗標
    - 正在恢復清除時，依 DeleteUserAccountLogic 處理，不執行本機補建
    - 清除結果未明或本機收尾未完成時，維持禁止補建
    - 清除流程允許恢復正常啟動後，才依當下身分與本機資料重新判定

---

## initializeDefaultEntities 建立預設記帳資料

- 帳戶與類別皆無紀錄時建立預設資料
- **性質:**
  - 純本機，無雲端動作
- **執行:**
  - 查詢目前使用者的全部帳戶與類別
  - 查詢範圍包含已刪除紀錄
  - **IF** 任一帳戶或類別已存在:
    - **回傳:**
      - 不建立任何資料
  - **ELSE:**
    - 建立現金與信用卡帳戶
    - 兩個帳戶皆採主要貨幣
    - 建立三個支出類別
    - 建立兩個收入類別
    - 名稱採本機設定語系
    - 不建立交易、轉帳與排程

---

## initializeCloudUser 建立雲端用戶文件

- 在 Firestore 建立 `users/{uid}` 文件，preferences 取本機 Settings 實際值
- 無條件建立，不分訂閱等級
- **建立雲端用戶文件:**
  - **執行:**
    - 在 Firestore 建立 `users/{uid}` 文件
    - preferences 欄位取本機 Settings 的實際值，而非寫死預設
    - 本機語系尚未取得時跳過雲端建立，不補入雲端預設語系，也不阻塞本機使用
  - **欄位:**
    - `uid`
    - `email`: 固定為 Null，不從認證資料取得 Email，legacy 登入身分也適用
    - `provider`: 匿名身分標 `anonymous`，legacy Google 與 Apple 登入身分保留實際提供者
    - `createdAt`
    - `preferences`
