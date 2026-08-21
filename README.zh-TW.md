# BV Workplace — 費用報銷工作流程展示 (Expense Reimbursement Workflow Demo)

一個單一檔案、可離線執行的互動式數位辦公室入口原型，核心圍繞完整的**費用報銷工作流程**：
申請人 (Requester) → 部門經理 (Manager) → 總經理 (GM，條件式) → 財務審核員 (Finance Reviewer)
→ 出納 (Cashier) → 已付款結案 (Paid / Closed)。

本套件已包含審查此作業所需的全部內容，無需任何開發環境：

- `index.html` — 完整入口網站。直接雙擊開啟（或用 Chrome／Edge 開啟）即可執行，無需伺服器、
  建置流程或網路連線。
- `README.md` — 英文版說明文件。
- `README.zh-TW.md` — 本檔案（中文版說明文件）。

---

## 1. 安裝與執行方式

1. 直接以最新版 **Chrome 或 Edge** 開啟 `index.html`（雙擊檔案，或拖曳進瀏覽器視窗）。介面、
   商業邏輯、種子資料全部包在這單一檔案內。
2. 不需要安裝任何套件、不需要 `npm install`、不需要網路連線。字型（Google Fonts、Font Awesome）
   僅為漸進增強，離線時會自動退回系統內建字型／文字符號，版面與所有功能不受影響。
3. 每次操作後，狀態會即時儲存到 `localStorage`，因此在同一瀏覽器重新整理頁面也不會遺失（符合
   作業要求的「可於 User Session 期間保存 Demo 狀態」，以及規格書 Trade-off 3 的實作承諾）。
   隨時可於「費用報銷工作區」點選 **重置展示案例**，決定性地還原案例 A–D 並覆蓋已儲存的資料。
4. 若要作為「Live Demo URL」，可直接將 `index.html` 原封不動上傳至任何靜態主機（GitHub Pages、
   Netlify、Azure Static Web Apps，或 SharePoint 文件庫）——不需要任何後端。

## 2. 入口網站登入與展示／測試帳號

開啟 `index.html`時，一律**先進入登入頁**（此登入關卡不會被跳過，也不會跨重新整理保留登入狀態，
為刻意設計）。登入頁不再使用自由輸入的帳號欄位，而是提供一份**目前啟用中身分的下拉清單**，
選擇身分後輸入該身分的展示用密碼：

| 欄位 | 值 |
|---|---|
| 身分 | 從下拉清單選擇，例如「Alex Chen — 申請人」 |
| 密碼 | 該身分名字的小寫（詳見下表） |

帳號與密碼皆為必填欄位（原生 HTML5 驗證）；輸入錯誤會顯示錯誤提示，登入頁在桌面版與行動版皆置中
顯示。登入成功後，右上角 Banner 的頭像會顯示目前登入的身分，點擊後開啟整合選單——參考
Microsoft Entra 系統管理中心的樣式——內含「**我的帳戶**」（目前登入身分本身的個人資料展示畫面）、
「**切換至其他使用者**」（登出並回到登入頁，以便另一個身分登入）與「**登出**」。

本應用程式不再有獨立於五個工作流程身分之上的「管理員」帳號——登入本身即等於選擇要扮演哪一個
角色。因此切換身分一律經由**登入畫面**進行：點擊頭像 →「切換至其他使用者」→ 從清單選擇另一個
身分 → 輸入其密碼 → 登入。「我的工作」／「展示指南」上的「登入為此身分」捷徑會在登入畫面上
預先選好該身分，但仍需輸入密碼才能完成切換。

| 帳號 | 姓名 | 角色 | 展示用密碼 | 展示最低權限 |
|---|---|---|---|---|
| 1 | Alex Chen | 申請人 | `alex` | 建立申請、查看本人單據、補件並重新送出被退回的項目 |
| 2 | Jordan Lee | 部門經理 | `jordan` | 僅可查看並處理 Manager Queue |
| 3 | Morgan Patel | 總經理 | `morgan` | 僅可查看並處理 GM Queue |
| 4 | Riley Nakamura | 財務審核員 | `riley` | 執行政策／會計審查，可核准、拒絕或退回 |
| 5 | Casey Romero | 出納 | `casey` | 僅可於財務核准後登錄付款，不得修改已核准金額 |

每一個簽核階段皆為**獨立具名身分**——不存在單一通用的「Approver」帳號，符合角色分離的要求。
於「系統管理 → 簽核權限」停用某個身分，會立即將其從登入清單中移除（若該身分正在登入中，也會
立即登出）；萬一所有身分都被同時停用，登入頁會提供「重置展示案例」的緊急還原按鈕，避免整個
展示環境被鎖死。

## 2a. 介面語言與品牌風格

- 頂部 Banner 提供 **EN／中文切換按鈕**，切換後全站導覽、按鈕、表格欄名、提示訊息，以及
  「系統架構」／「展示指南」的內容皆會一併切換。切換為中文時，介面字型強制變更為
  **微軟正黑體**（若作業系統無此字型，退回 PMingLiU／黑體-繁／sans-serif）；切換為英文時
  強制採用 **Arial**。資料欄位（單號、金額、時間戳記）在兩種語言下皆維持等寬字體，以利辨識——
  此為對「介面字型」規則刻意且已揭露的例外處理。
- **側邊導覽列品牌風格**：白色背景，導覽文字與圖示採用品牌藍綠色 `#3BBCB3`，並以細微邊框與
  陰影與內容區隔開，符合視覺規格要求。「系統管理」已移至導覽列最下方，緊接在側邊欄底部資訊
  區塊之上。
- **登入頁面置中**：無論桌面版或行動版，登入卡片皆置中顯示於整個瀏覽器視窗。

## 3. 技術選型

- **原生 HTML／CSS／JavaScript**，單一檔案，不使用框架、不需建置流程——刻意如此選擇，讓
  「離線備份套件」（面試官零安裝即可開啟的單一 HTML 檔）這項要求從架構上直接滿足，而非事後補救。
- 狀態以純 JavaScript 物件保存，透過樣板字串 (template string) 渲染畫面，並以單一委派事件監聽器
  處理所有互動（不使用 Virtual DOM、不依賴外部 UI 套件）——面試現場若被要求即席修改，也容易閱讀。
- **LocalStorage 持久化**（依規格書 Trade-off 3 實作）：所有單據、稽核歷程、設定、通知與系統
  事件在每次狀態變更後即寫入 `window.localStorage`，下次以同一瀏覽器開啟本檔案時會自動還原，
  同時滿足「免安裝開發環境即可開啟」的交付要求，以及重新整理頁面不遺失資料的體驗。若環境
  無法使用 LocalStorage（例如受限的預覽 iframe），系統會自動偵測並降級為純記憶體模式運作，
  並於側邊欄底部顯示提示。
- **Font Awesome 6**（CDN）提供頂部 Banner 的會員登入／登出圖示
  (<i class="fa-solid fa-user"></i>)——與 Google Fonts 一樣屬於漸進增強；若離線無法連接 CDN，
  按鈕功能仍正常運作，僅圖示樣式退回預設。

## 4. 已實作範圍

- 完整費用報銷狀態機：**Draft → Submitted → Pending Manager → Pending GM（僅 Level 2／3）→
  Pending Finance → Pending Payment → Paid／Closed**，並將 **Reject** 與 **Return** 拆分為兩個
  獨立的狀態（不再共用同一個狀態）：Reject 一律退回申請人補正；Return（僅財務可執行）則可選擇
  退回申請人，或直接退回路由中先前任一審核角色（詳見下方）。兩者皆能於 Resubmit 後正確回到
  對應階段續轉。
- **角色感知的費用報銷工作區**：單據列表會依目前登入身分過濾——申請人僅看到自己的單據；
  審核角色（經理／總經理／財務／出納）僅看到「其簽核路由確實包含該角色」的單據，因此例如
  Level 1 單據（自動略過總經理）不會出現在總經理的工作區中，與該角色本來就不會收到此任務
  的事實一致。
- **簽核路由 Stepper**：由 Requester → Manager → GM → Finance → Cashier 共五個節點組成，原本
  獨立的「Paid」節點已併入 Cashier 節點（付款前顯示為進行中的脈動效果，實際付款完成後才顯示
  勾選完成），因為處理付款本來就是出納的職責，而非另一個獨立階段。
- **設定驅動的路由，並含匯率換算**：USD／JPY 金額會先以展示用固定匯率（非即時匯率）換算為
  台幣，再依 Level 1／2／3 門檻判斷路由層級，確保路由永遠依台幣等值金額判斷，而非原幣別金額。
  Level 1 ≤ NT$5,000（自動略過 GM）、Level 2 NT$5,001–20,000（需 GM）、Level 3 > NT$20,000
  （需 GM，並自動加註 High-Value）。門檻值可於「系統管理」集中編輯，僅套用於未來新申請。
- **存為草稿，並可完整重新編輯**：新申請可先儲存為草稿、不進入簽核流程。草稿的單據明細頁面
  會提供「編輯」按鈕，開啟同一張表單並完整帶入原本內容，申請人可修改後選擇再次存為草稿，或
  直接送出審核（路由與匯率換算會在實際送出審核當下才計算，而非首次儲存草稿時）。
- **新增報銷申請表單驗證**：所有必填欄位皆標示紅色 *；金額欄位即時檢核，輸入零、負數或文字
  時框線反紅並顯示紅字錯誤訊息。收據檢核採不對稱設計：選擇「**目前無收據**」時完全不會觸發
  任何檔案相關檢查，表單可正常送出、不會出現缺收據錯誤；選擇「**附上收據**」時，則必須實際
  上傳檔案（PDF／JPEG／PNG／DOCX）才能送出——若正在編輯的草稿或被退回單據已有既存檔案，
  重新上傳為選填（未重新選檔則沿用原檔案）。僅申請人身分可開啟此表單，其餘角色一律唯讀。
- **Fix & Resubmit 改為開啟完整編輯畫面**：被拒絕／退回單據的「Fix & Resubmit」按鈕，現在會
  彈出與草稿編輯相同的「編輯報銷申請」表單——完整帶入原本內容，並於畫面上顯示原始拒絕／退回
  原因以利對照——而非僅有一個簡易意見輸入框。所有欄位皆可修正，不僅限於附件；此模式下主要
  按鈕文字為「**Fix & Resubmit**」而非 Submit，且重新送出說明為必填。
- 五個角色專屬、具名的展示身分。切換身分一律經由**登入畫面**進行（詳見第 2 節），而非站內
  即時切換器；角色感知的「我的工作」佇列會反映目前登入的身分。在「我的工作」頁面中，僅目前
  使用中的身分顯示為啟用狀態，其餘四個反灰，並提示改用頭像選單中的「切換至其他使用者」，
  讓身分切換入口保持單一、一致。
- **「我的工作 → View」改為彈出視窗**顯示單據內容，不會離開目前的佇列頁面，方便審核人維持
  在原本的操作情境中。
- **彈性化 Return 流程**：財務執行 Return 時可選擇要退回給誰——申請人（進行補正），或路由中
  先前任一審核角色（例如財務可直接退回給經理重新審核，不必強制走一次完整的申請人補正流程）。
  Approve、Reject、Return、Resubmit 皆改為意見必填。
- 每筆單據皆有 Append-only 的 Approval History（操作者、角色、動作、時間戳記、意見、
  操作後狀態），支援單筆與全量匯出（JSON／CSV，瀏覽器內下載）。
- 通知與真實狀態轉換連動（新核准任務、退回／拒絕、付款完成）。
- **系統管理**：可編輯的路由門檻；完整可管理的簽核權限表（Role、User Name、Entra Group、
  啟用開關），每一列皆可啟用／停用，並可透過「＋ 新增角色」即時新增角色指派（選擇 Role 後
  Entra Group 會自動帶入）；流程健康指標；已處理例外模擬（Correlation ID、Owner Alert、
  冪等重試）；以及系統事件紀錄。停用某個身分後，該身分會立即從站內所有「切換身分」清單中
  移除（若該身分正是目前使用中的身分，系統會自動切換至其他啟用中的身分）。
- 系統架構頁面涵蓋元件、**資料模型（六張受控清單）**、狀態機、權限模型、例外處理、部署與維運、
  架構取捨，以及展示範圍 vs. 正式環境落差清單。
- 一鍵**重置展示案例**（重置後停留在費用報銷工作區），決定性地重新產生歷史付款範例與案例 A–D。
- 響應式版面（約 900px 以下側邊欄可收合）、鍵盤可操作的原生按鈕／輸入元件、清楚可見的
  Focus 樣式、語意化標題與表格。
- 入口網站登入關卡（選擇身分＋展示用密碼）與頂部整合式頭像選單（我的帳戶／切換至其他使用者／登出）。

## 5. 已知限制（僅為展示範圍，非正式環境）

- 入口網站登入為展示用的簡化機制，並非真實驗證——每個身分的「密碼」皆為可預期、已公開說明的
  展示慣例（名字小寫），完全於瀏覽器端比對，並非 Entra ID（詳見「系統架構 → 架構取捨」）。
- 上傳的收據檔案不會進行真實病毒掃描——本 Demo 僅記錄檔名，檔案內容本身不會被儲存或傳送至
  任何地方。
- 匯率換算採用展示用的固定常數，並非即時匯率（已於「系統管理」與「系統架構 → 資料模型」中
  註明）。
- 草稿可以只填一部分欄位就先儲存；「編輯」畫面讓申請人可以補齊或修改任何欄位後，選擇再次
  存為草稿或直接送出審核，但草稿的編輯過程沒有欄位層級的「異動比對」或版本歷程，僅保留最後
  一次儲存的結果。
- 所有授權判斷皆於本檔案的 `canAct()` 函式中執行；沒有獨立的伺服器端強制執行，因此「系統架構」
  頁面明確指出正式環境中此檢查應移至哪一層。
- 單據資料會持久化至 `localStorage`（見第 3 節），但登入狀態本身刻意**不**持久化——即使
  工作流程資料已還原，每次重新載入頁面仍會回到登入畫面。

## 6. AI 工具使用揭露

本原型（應用程式程式碼、介面文案與本文件）係在 **Claude**（Anthropic）協助下，依據作業規格書
直接撰寫完成。AI 協助範圍包括：搭建單一檔案應用程式架構、產生 CSS 設計系統、草擬工作流程／
狀態機邏輯，以及撰寫系統架構／架構取捨文案。所有最終的設計決策、資安假設與程式碼皆已對照
作業檢查清單審閱過後才交付，作者仍負責在面試現場說明或即席修改任何部分。

## 7. 架構摘要

*（完整內容——包含元件圖、權限模型與例外處理設計——皆位於入口網站內的「系統架構」頁面，
共八個頁籤：架構元件圖、資料模型、狀態機、權限模型、例外處理、部署與維運、架構取捨、
展示範圍 vs. 正式環境。）*

**目標 Microsoft 365 架構對應**

| 層級 | 正式環境服務 | 職責範圍 |
|---|---|---|
| Experience Layer | SharePoint Portal（或等效的角色感知、任務導向介面） | 僅負責呈現，不持有業務規則真實來源 |
| Data Layer | Microsoft Lists／Dataverse | Expense Requests、Approval History、Approval Matrix、Workflow Configuration |
| Process Layer | Power Automate | 路由、核准、通知、例外範圍、安全重試 |
| Identity Layer | Entra ID Groups | 最小權限、職責分離 |
| Operations | 服務端連線管理 | 監控、支援責任、ALM、環境分離 |

**資料模型（六張受控清單，欄位命名依規格書）**

| 清單 | 主要欄位 |
|---|---|
| 1. Expense Requests | Request_ID、Requester、Department、Expense_Date、Expense_Type、Amount、Currency、Purpose、Project_Cost_Center、Payee、Attachment_Status（Boolean）、Route_Level、Status、Payment_Reference、Is_Locked（Boolean） |
| 2. Approval History（Append-only） | History_ID、Request_ID、Actor_Name、Role、Action（Submit／Approve／Reject／Return／Resubmit／Payment）、Timestamp、Comment_Reason、Resulting_Status |
| 3. Workflow Configuration | Config_Key（例：`Level_1_Threshold`）、Config_Value（例：`5000`） |
| 4. Approval Matrix | Role_Name（Manager／GM／Finance／Cashier）、Assigned_Identity |
| 5. User Profile（個人基本資料） | User_ID、Full_Name、Photo_URL、Birthday、Country_Region、Language、Regional_Format、Email、Phone —— 對應「我的帳戶」頁面 |
| 6. Member Account（會員帳號） | Account_ID、Account、Password_Hash（隱碼儲存，絕不存明碼）、Role（與表 2 的 Role 欄位共用同一角色列舉，可於「系統管理 → 簽核權限」指派）、Is_Active（預設為 true；停用後無法登入、也會從登入清單與所有切換身分入口移除——可於「系統管理 → 簽核權限」即時管理） |

Workflow Configuration 於單據提交當下讀取一次並鎖存至該單據的 Route_Level，因此日後調整門檻值
不會回溯影響進行中的單據。

**為何隱藏按鈕不等於安全性** — 入口網站為求畫面簡潔而在 UI 上隱藏未授權的操作，但每一次狀態
轉換前，都會透過單一授權函式 (`canAct`) 重新驗證——概念上等同正式環境中 Power Automate 的
「Authorization」Scope 於伺服器端對 Entra ID 群組成員資格與 Dataverse 列／欄權限所做的檢查。
正式環境中，此檢查必須落在資料層／流程層強制執行，絕不能只依賴前端 JavaScript。

**例外處理** — 每個流程動作皆包在 Try/Catch Scope 內，每次執行皆有 Correlation ID，並寫入
專屬的 Exception Record（原因、最後有效狀態、時間戳記），通知流程負責人（而非業務審核人），
且重試以 Request ID ＋ 意圖執行的動作作為冪等鍵，確保重試後的操作不會產生重複的核准、通知
或付款。

**架構取捨（完整風險／下一步說明請見入口網站內「系統架構 → 架構取捨」，站內為雙語呈現）**

1. **切換身分機制**（Simulated Identity vs. Real Entra Authentication）— Demo 環境採用「每個身分
   各自登入」的登入畫面（選擇身分、輸入該身分的展示用密碼）模擬登入體驗。風險：該「密碼」為
   可預期、已公開說明的展示慣例，並非真實憑證，惡意使用者仍可能猜出密碼或直接呼叫 API 嘗試
   核准單據。下一步：正式環境中移除展示用密碼機制，改為讀取真實 Entra ID 登入憑證，並於資料層
   套用 Item-level Permissions。
2. **快速原型開發 vs. 集中式關聯式資料**（Rapid Prototyping vs. Centralized Relational Data）—
   採用 Microsoft Lists（NoSQL-like 平面表）以求最快建立可互動的 UI。風險：資料量龐大或需要
   複雜 Table Join（例如單一報銷單含多筆不同科目明細）時，效能與擴充性較差。下一步：延伸支援
   如 Purchase 等多筆明細表單時，應遷移至 Dataverse 或 Azure SQL Database。
3. **在地瀏覽器暫存 vs. 集中式雲端儲存**（Local Browser State vs. Persistent Cloud Data）—
   本次交付以單一 HTML 檔案，資料保存於 `localStorage`，以符合「免安裝開發環境即可開啟」的
   交付要求。風險：清除瀏覽器資料或關閉視窗可能遺失紀錄，且無法呈現真正的多人協同簽核體驗。
   下一步：正式上線前，將 Data Source 由 `localStorage` 全面替換為實際雲端 API（如 Microsoft
   Graph API 或自建 Backend RESTful API）。

## 8. 測試證據矩陣

| 案例 | 設定 | 預期證明 | 結果 |
|---|---|---|---|
| **A** | NT$3,000，收據完整 | 經理核准後直接進入財務審核；GM 完全不會收到任務；財務核准；出納完成付款；最終狀態為 Paid/Closed。 | ✅ 已驗證 — `computeRoute` 對 Level 1 回傳的 stages 為 `[Manager, Finance, Cashier]`，GM 在結構上被排除於路由之外，而非僅是畫面隱藏。 |
| **B** | NT$12,000，收據完整 | 經理、總經理、財務、出納各自以獨立具名身分操作；四位皆出現在 Approval History。 | ✅ 已驗證 — Level 2 路由包含 GM；每次轉換皆記錄操作者姓名與角色。 |
| **C** | NT$50,000，收據完整 | Level 3 ＋ High-Value 自動標示；完整路由被強制執行；出納不得修改已核准金額；付款後紀錄鎖定。 | ✅ 已驗證 — 金額超過 Level 2 門檻即自動設定 `highValue` 旗標；付款表單僅開放 Payment Reference／Date 欄位，絕不含 Amount；付款後 `locked=true`，所有後續操作皆被停用。 |
| **D** | NT$8,000，缺收據 | 經理以理由拒絕；申請人補上證明並重新送出；原始拒絕紀錄／意見保留未被覆寫；流程由正確階段繼續。 | ✅ 已驗證 — Reject 要求非空白意見；拒絕事件為新增（絕不覆寫既有紀錄）；Resubmit 會將 `currentStage` 設回 `returnedFrom`（Manager），而非回到 Draft。 |
| **新建 E2E** | 以 Alex Chen（申請人）身分，透過「＋ 新增報銷」建立任一新申請 | 送出前即顯示路由；每個階段的佇列正確地新增／移除該任務；付款完成後具備付款參考碼、Paid/Closed 狀態、鎖定金額，以及完整稽核歷程。 | ✅ 已驗證 — `buildRequest()` 於送出前即計算並顯示路由；每次 `applyAction` 呼叫皆同步更新 `status`／`currentStage` 並觸發對應的佇列通知，已透過走完五個身分的完整流程確認無誤。 |
| **匯率路由** | 新建申請，金額 200 USD（依展示匯率約當 NT$6,300） | 應路由為 Level 2（需 GM），而非 Level 1，因為門檻判斷使用的是換算後的台幣等值金額，而非原始美金數字。 | ✅ 已驗證 — `buildRequest()`／`applyAction('submit-draft')` 皆先呼叫 `toTWD()` 換算，才呼叫 `computeRoute()`；畫面上的路由預覽會隨幣別／金額即時更新。 |
| **彈性 Return** | 任一 Level 2／3 單據處於待財務審核狀態 | 財務執行 Return 時，可選擇的對象包含經理（若為 Level 3 亦包含總經理），而非只能退回申請人；選擇退回經理時，單據會直接回到「待經理審核」狀態，不需經過申請人補正流程。 | ✅ 已驗證 — Return 彈窗的對象下拉選單由 `priorStagesOf(req)` 動態產生；`applyAction('return', {target})` 在目標非申請人時，直接設定 `currentStage`／`status`。 |
| **草稿** | 新建申請時選擇「存為草稿」，之後再開啟該單據並「送出審核」 | 草稿會出現在費用報銷工作區，狀態顯示為 Draft 且尚未計算路由；點選送出審核後，才計算路由（含匯率換算）並進入待經理審核狀態。 | ✅ 已驗證 — `buildDraftRequest()` 產生 `status:'Draft'`、`routeLevel:null`；`submit-draft` 動作會鎖存路由並新增一筆 `Submit` 稽核紀錄。 |
| **草稿編輯** | 儲存一筆草稿，透過「編輯」重新開啟，修改金額／幣別／事由後再次「存為草稿」 | 表單開啟時會完整帶入草稿目前的內容；再次儲存會更新同一筆單據（單號不變），而非產生重複單據。 | ✅ 已驗證 — `edit-request` 會將該單據載入 `state.editingRequestId`；`handleSaveDraftExpense` 的編輯分支會直接修改既有單據物件，而非呼叫 `buildDraftRequest()` 建立新單據。 |
| **收據檢核不對稱性** | (a) 新建申請選擇「目前無收據」後送出。(b) 新建申請選擇「附上收據」但不選擇檔案即送出。 | (a) 應能正常送出，不出現任何檔案相關錯誤。(b) 送出應被阻擋，並顯示紅字「請選擇檔案」提示。 | ✅ 已驗證 — `resolveAttachment()` 在情境 (a) 會直接回傳 `{missing:true}`，完全不觸發檔案檢查；情境 (b) 則回傳 `null`，`handleSubmitNewExpense` 會將其視為強制阻擋條件。 |
| **簽核權限治理** | 在系統管理 → 簽核權限中，將目前使用中的身分停用，接著點選「＋ 新增角色」並填入一位新的 Manager | 被停用的身分應立即從所有「切換身分」清單消失，且目前使用中的身分會自動切回另一個啟用中的身分；新列的 Entra Group 會依所選 Role 自動帶入，且一旦填入姓名即可被選為新的展示身分。 | ✅ 已驗證 — `handleToggleUserActive()` 會在停用目前使用中的身分時，將 `state.currentUserId` 重新指向 `activeUsers()[0]`；`handleAddNewRole()` 新增的 `custom:true` 項目與其他身分一樣，皆由 `entraGroupForRole()` 與 `activeUsers()` 統一處理。 |
| **Reject 與 Return 狀態分離** | (a) 案例 D：經理執行 Reject。(b) 任一 Level 2／3 單據：財務將其 Return 給申請人 | (a) 狀態徽章與篩選皆顯示「已拒絕」，而非「已退回」。(b) 狀態徽章與篩選皆顯示「已退回」。兩者皆會出現在申請人的「我的工作」佇列中，並可執行 Fix & Resubmit。 | ✅ 已驗證 — `applyAction('reject', …)` 會設定 `status:'Rejected'`；`applyAction('return', {target:'Requester'})` 會設定 `status:'Returned'`；`canAct`／`requestsForRole`／`statusBadge` 皆同時判斷這兩種狀態。 |
| **合併後的 Stepper 節點** | 任一單據，從送出到付款完整走一輪 | 簽核路由顯示 5 個節點（Requester／Manager／GM／Finance／Cashier）；出納節點在「待付款」時顯示為進行中的脈動效果，「已付款」後才顯示勾選完成——不再有獨立的第 6 個「Paid」節點。 | ✅ 已驗證 — `renderStepper()` 的 `order` 陣列僅有 5 個項目；`isPaid` 會直接將最後的出納節點標記為完成，而非索引已移除的 `'Paid'` 項目。 |
| **角色感知的費用報銷工作區** | 以 Morgan Patel（總經理）身分登入，開啟費用報銷工作區，並存在一筆 Level 1 單據（如案例 A） | 案例 A 不會出現在總經理的費用報銷工作區列表或統計數字中，因為總經理不屬於 Level 1 的簽核路由；Level 2／3 的案例則會正常顯示。 | ✅ 已驗證 — `visibleRequestsForRole()` 對審核角色以 `r.stages.includes(u.role)` 過濾，對申請人角色則以 `r.requesterId===u.id` 過濾。 |
| **登入式身分切換** | 點擊頭像 →「切換至其他使用者」，選擇另一個身分，輸入其展示用密碼 | 應導向登入畫面，並以下拉清單（而非自由輸入的帳號欄位）預先選取目前身分；輸入錯誤密碼會顯示錯誤且不會切換；輸入正確密碼後即以新選擇的身分登入。 | ✅ 已驗證 — `handleGotoLoginAs()` 會設定 `authenticated:false` 與 `loginSelectedId`；`handleLogin()` 會透過 `demoPasswordFor()` 驗證所選身分的密碼後，才設定 `authenticated:true` 與 `currentUserId`。 |

案例 A–D 皆可由**重置展示案例**（重置後會停留在費用報銷工作區）決定性地重新產生，且可任意順序
執行——每個案例僅依賴其自身單據的狀態，不依賴先前的展示歷史。
