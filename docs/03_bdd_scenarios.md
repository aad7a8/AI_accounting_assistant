# CoffeeBook AI — 行為驅動情境 (BDD Scenarios)

---

**文件版本 (Document Version):** `v1.0`
**最後更新 (Last Updated):** `2026-02-11`
**主要作者 (Lead Author):** `[技術負責人, 產品經理]`
**對應 PRD:** `02_prd.md v1.5`
**狀態 (Status):** `活躍 (Active)`

**相關文件：**
- 使用者故事與驗收標準 → [PRD (02_prd.md)](02_prd.md)
- 模組層級測試案例 (DbC + AAA) → [模組規格與測試 (07_module_specification_and_tests.md)](07_module_specification_and_tests.md)
- API 端點對應 → [API 設計規範 (06_api_design_specification.md)](06_api_design_specification.md)

---

## 目錄 (Table of Contents)

- [Epic 1: 智能記帳輸入 (Smart Input)](#epic-1-智能記帳輸入-smart-input)
  - [Feature 1-1: 語音輸入記帳 (US-101)](#feature-1-1-語音輸入記帳-us-101)
  - [Feature 1-2: 快速按鈕記錄銷售 (US-102)](#feature-1-2-快速按鈕記錄銷售-us-102)
  - [Feature 1-3: AI 自動分類支出 (US-103)](#feature-1-3-ai-自動分類支出-us-103)
  - [Feature 1-4: 照片附件 (US-104)](#feature-1-4-照片附件-us-104)
  - [Feature 1-5: 自訂快速按鈕 (US-105)](#feature-1-5-自訂快速按鈕-us-105)
- [Epic 2: 每日概覽儀表板 (Daily Overview)](#epic-2-每日概覽儀表板-daily-overview)
  - [Feature 2-1: 今日營收概覽 (US-201)](#feature-2-1-今日營收概覽-us-201)
  - [Feature 2-2: 營收趨勢圖 (US-202)](#feature-2-2-營收趨勢圖-us-202)
  - [Feature 2-3: 今日品項銷售排行 (US-203)](#feature-2-3-今日品項銷售排行-us-203)
- [Epic 3: 月報表自動生成 (Monthly Report)](#epic-3-月報表自動生成-monthly-report)
  - [Feature 3-1: 月報表自動產生 (US-301)](#feature-3-1-月報表自動產生-us-301)
  - [Feature 3-2: 成本結構圓餅圖 (US-302)](#feature-3-2-成本結構圓餅圖-us-302)
  - [Feature 3-3: 熱銷品項排行 (US-303)](#feature-3-3-熱銷品項排行-us-303)
  - [Feature 3-4: 歷史月報表查詢 (US-304)](#feature-3-4-歷史月報表查詢-us-304)
- [Epic 4: 使用者管理與基本權限 (User Management)](#epic-4-使用者管理與基本權限-user-management)
  - [Feature 4-1: 新增員工帳號 (US-401)](#feature-4-1-新增員工帳號-us-401)
  - [Feature 4-2: 員工權限設定 (US-402)](#feature-4-2-員工權限設定-us-402)
  - [Feature 4-3: 記錄者追蹤 (US-403)](#feature-4-3-記錄者追蹤-us-403)

---

## Epic 1: 智能記帳輸入 (Smart Input)

### Feature 1-1: 語音輸入記帳 (US-101)

**檔案名稱**: `epic_smart_input.feature`

```gherkin
# Feature: 語音輸入記帳
# 目的: 讓員工不用打字也能透過語音快速記錄支出。
# 對應 PRD: US-101 (P0)

Feature: Voice Input for Expense Recording

  Background:
    Given I am a logged-in employee
    And I am on the main recording page

  # --- Happy Path ---

  @happy-path @smoke-test @US-101
  Scenario: 成功使用語音記錄一筆支出
    When I tap the "Voice Input" button
    Then the system should start recording audio
    When I say "買咖啡豆 500 元"
    And the recording ends
    Then the system should display the transcribed text "買咖啡豆 500 元" within 3 seconds
    And the system should auto-fill the amount as 500
    And the system should auto-classify the category as "進貨成本"
    And the system should auto-fill the item name as "咖啡豆"
    When I tap the "Confirm" button
    Then the transaction should be saved successfully
    And I should see a success message "記錄成功"

  @happy-path @US-101
  Scenario: 語音記錄一筆固定支出
    When I tap the "Voice Input" button
    And I say "繳電費 800 元"
    And the recording ends
    Then the system should display the transcribed text "繳電費 800 元" within 3 seconds
    And the system should auto-fill the amount as 800
    And the system should auto-classify the category as "固定支出"
    When I tap the "Confirm" button
    Then the transaction should be saved successfully

  # --- Sad Path ---

  @sad-path @US-101
  Scenario: 語音辨識無法識別內容
    When I tap the "Voice Input" button
    And the background noise is too loud to recognize speech
    And the recording ends
    Then the system should display a message "無法辨識語音，請再試一次或手動輸入"
    And the system should provide a "Manual Input" fallback option

  @sad-path @US-101
  Scenario: 語音辨識服務不可用 (離線狀態)
    Given the device has no network connection
    When I tap the "Voice Input" button
    Then the system should display a message "語音功能需要網路連線，請使用手動輸入"
    And the system should redirect me to the manual input form

  # --- Edge Case ---

  @edge-case @US-101
  Scenario: 語音辨識結果金額錯誤，使用者手動修正
    When I tap the "Voice Input" button
    And I say "買咖啡豆 500 元"
    And the recording ends
    And the system auto-fills the amount as 5000
    When I manually correct the amount to 500
    And I tap the "Confirm" button
    Then the transaction should be saved with amount 500

  @edge-case @US-101
  Scenario: 語音輸入未包含金額
    When I tap the "Voice Input" button
    And I say "買咖啡豆"
    And the recording ends
    Then the system should display the transcribed text "買咖啡豆"
    And the amount field should be empty
    And the system should prompt "請輸入金額"
```

---

### Feature 1-2: 快速按鈕記錄銷售 (US-102)

```gherkin
# Feature: 快速按鈕記錄銷售
# 目的: 讓員工在忙碌時段也能一鍵記錄銷售。
# 對應 PRD: US-102 (P0)

Feature: Quick Button Sales Recording

  Background:
    Given I am a logged-in employee
    And I am on the main recording page
    And the quick button "美式咖啡" exists with default price 120

  # --- Happy Path ---

  @happy-path @smoke-test @US-102
  Scenario: 點擊快速按鈕記錄一杯銷售
    When I tap the quick button "美式咖啡"
    Then the system should auto-record a sale with item "美式咖啡", quantity 1, and amount 120
    And I should see a success message "已記錄：美式咖啡 x1 $120"

  @happy-path @US-102
  Scenario: 修改杯數後記錄銷售
    When I tap the quick button "美式咖啡"
    And I change the quantity to 3
    And I tap the "Confirm" button
    Then the system should save a sale with item "美式咖啡", quantity 3, and amount 360

  @happy-path @US-102
  Scenario: 修改本次售價後記錄銷售 (促銷折扣)
    When I tap the quick button "美式咖啡"
    And I change the unit price to 100
    And I tap the "Confirm" button
    Then the system should save a sale with item "美式咖啡", quantity 1, and amount 100
    And the default price of "美式咖啡" should remain 120

  @happy-path @US-102
  Scenario: 修改杯數與售價後記錄銷售 (假日加價)
    When I tap the quick button "美式咖啡"
    And I change the quantity to 2
    And I change the unit price to 150
    And I tap the "Confirm" button
    Then the system should save a sale with item "美式咖啡", quantity 2, and amount 300

  # --- Edge Case ---

  @edge-case @US-102
  Scenario: 快速按鈕清單可滑動瀏覽
    Given there are 20 quick buttons configured
    When I am on the main recording page
    Then I should be able to scroll to view all 20 quick buttons

  @edge-case @US-102
  Scenario: 杯數修改為 0 時阻擋
    When I tap the quick button "美式咖啡"
    And I change the quantity to 0
    And I tap the "Confirm" button
    Then I should see a validation error "杯數必須大於 0"
```

---

### Feature 1-3: AI 自動分類支出 (US-103)

```gherkin
# Feature: AI 自動分類支出
# 目的: 系統自動將支出分門別類，店主不需每次手動選擇。
# 對應 PRD: US-103 (P0)
# 外部依賴: Google Gemini API (備援: 本地關鍵字規則引擎)

Feature: AI Auto-Classification of Expenses

  Background:
    Given I am a logged-in shop owner
    And I am on the expense recording page

  # --- Happy Path ---

  @happy-path @smoke-test @US-103
  Scenario Outline: 系統正確自動分類不同類型的支出
    When I input the expense description "<description>"
    Then the system should auto-classify the category as "<category>"
    And I can optionally fill in the note field with "<note>"
    When I tap the "Confirm" button
    Then the transaction should be saved with category "<category>" and note "<note>"

    Examples:
      | description       | category | note   |
      | 買咖啡豆 500 元   | 進貨成本 | 咖啡豆 |
      | 繳電費 800 元      | 固定支出 | 電費   |
      | 買牛奶 200 元      | 進貨成本 | 牛奶   |
      | 繳房租 15000 元    | 固定支出 | 房租   |
      | 修理咖啡機 3000 元 | 設備維修 |        |

  # --- Sad Path ---

  @sad-path @US-103
  Scenario: AI 分類錯誤，使用者手動修正
    When I input the expense description "買杯子 300 元"
    And the system auto-classifies the category as "進貨成本"
    But the correct category is "設備維修"
    When I manually change the category to "設備維修"
    And I tap the "Confirm" button
    Then the transaction should be saved with category "設備維修"

  @sad-path @US-103
  Scenario: AI 分類服務不可用時使用本地規則引擎
    Given the Gemini API is unavailable
    When I input the expense description "買咖啡豆 500 元"
    Then the system should fallback to the local keyword-based classifier
    And the system should auto-classify the category as "進貨成本"

  # --- Edge Case ---

  @edge-case @US-103
  Scenario: 輸入內容無法判別分類
    When I input the expense description "雜支 100 元"
    And the system cannot determine a category
    Then the system should set the category to "未分類"
    And prompt the user to select a category manually
```

---

### Feature 1-4: 照片附件 (US-104)

```gherkin
# Feature: 照片附件
# 目的: 讓使用者在記帳時附加發票或收據照片，方便日後追溯。
# 對應 PRD: US-104 (P1)

Feature: Photo Attachment for Transactions

  Background:
    Given I am a logged-in employee
    And I am creating a new transaction record

  # --- Happy Path ---

  @happy-path @US-104
  Scenario: 記帳時拍照附加照片
    When I tap the "Attach Photo" button
    And I choose "Take Photo"
    And I take a photo of the receipt
    Then the photo should be attached to the current record
    When I tap the "Confirm" button
    Then the transaction should be saved with the photo attached

  @happy-path @US-104
  Scenario: 記帳時從相簿選擇照片
    When I tap the "Attach Photo" button
    And I choose "Choose from Album"
    And I select a photo from the album
    Then the photo should be attached to the current record
    When I tap the "Confirm" button
    Then the transaction should be saved with the photo attached

  @happy-path @US-104
  Scenario: 在記錄詳情頁查看附件照片
    Given a transaction exists with an attached photo
    When I open the transaction detail page
    Then I should see the attached photo displayed

  # --- Edge Case ---

  @edge-case @US-104
  Scenario: 不附加照片也能成功記帳
    When I fill in the transaction details
    And I do not attach any photo
    And I tap the "Confirm" button
    Then the transaction should be saved successfully without a photo
```

---

### Feature 1-5: 自訂快速按鈕 (US-105)

```gherkin
# Feature: 自訂快速按鈕
# 目的: 讓店主根據實際菜單自訂快速按鈕的品項與價格。
# 對應 PRD: US-105 (P0)

Feature: Custom Quick Button Management

  Background:
    Given I am a logged-in shop owner
    And I navigate to "Settings" > "Quick Button Management"

  # --- Happy Path ---

  @happy-path @smoke-test @US-105
  Scenario: 新增一個快速按鈕品項
    When I tap the "Add Item" button
    And I fill in item name "拿鐵咖啡"
    And I fill in default price 150
    And I tap the "Save" button
    Then a new quick button "拿鐵咖啡" should appear with price 150
    And the button should be visible on the main recording page

  @happy-path @US-105
  Scenario: 新增快速按鈕並上傳圖片
    When I tap the "Add Item" button
    And I fill in item name "卡布奇諾"
    And I fill in default price 140
    And I upload an icon image
    And I tap the "Save" button
    Then a new quick button "卡布奇諾" should appear with the uploaded icon

  @happy-path @US-105
  Scenario: 編輯現有快速按鈕
    Given a quick button "美式咖啡" exists with price 120
    When I tap the edit icon on "美式咖啡"
    And I change the price to 130
    And I tap the "Save" button
    Then the quick button "美式咖啡" should now show price 130

  @happy-path @US-105
  Scenario: 刪除一個快速按鈕
    Given a quick button "季節限定" exists
    When I tap the delete icon on "季節限定"
    And I confirm the deletion
    Then the quick button "季節限定" should no longer appear

  @happy-path @US-105
  Scenario: 拖曳排序快速按鈕顯示順序
    Given the following quick buttons exist in order: "美式咖啡", "拿鐵咖啡", "卡布奇諾"
    When I drag "卡布奇諾" to the first position
    Then the quick buttons should display in order: "卡布奇諾", "美式咖啡", "拿鐵咖啡"

  # --- Edge Case ---

  @edge-case @US-105
  Scenario: 新增按鈕時未填名稱
    When I tap the "Add Item" button
    And I leave the item name empty
    And I fill in default price 100
    And I tap the "Save" button
    Then I should see a validation error "品項名稱為必填"

  @edge-case @US-105
  Scenario: 支援大量按鈕滑動查看
    Given there are 30 quick buttons configured
    When I am on the main recording page
    Then I should be able to scroll to view all 30 quick buttons
```

---

## Epic 2: 每日概覽儀表板 (Daily Overview)

### Feature 2-1: 今日營收概覽 (US-201)

**檔案名稱**: `epic_daily_overview.feature`

```gherkin
# Feature: 今日營收概覽
# 目的: 店主在營業結束後 3 分鐘內掌握今日營運狀況。
# 對應 PRD: US-201 (P0)

Feature: Daily Revenue Overview Dashboard

  Background:
    Given I am a logged-in shop owner
    And today is "Wednesday"

  # --- Happy Path ---

  @happy-path @smoke-test @US-201
  Scenario: 查看今日營收概覽
    Given the following transactions exist for today:
      | item       | type   | quantity | amount |
      | 美式咖啡   | income | 10       | 1200   |
      | 拿鐵咖啡   | income | 8        | 1200   |
      | 買咖啡豆   | expense| 1        | 500    |
    When I open the app home page
    Then I should see today's total revenue as 2400
    And I should see today's total cups sold as 18
    And I should see today's net profit as 1900

  @happy-path @US-201
  Scenario: 查看與上週同天的比較
    Given today's revenue is 2400
    And last Wednesday's revenue was 2000
    When I open the app home page
    Then I should see the week-over-week comparison "比上週三多賺 400 元 ↑20%"

  @happy-path @US-201
  Scenario: 點擊數字展開詳細明細
    When I open the app home page
    And I tap on today's total revenue number
    Then I should see the detailed transaction list for today

  # --- Edge Case ---

  @edge-case @US-201
  Scenario: 今日無任何交易記錄
    Given there are no transactions for today
    When I open the app home page
    Then I should see today's total revenue as 0
    And I should see today's total cups sold as 0
    And I should see a message "今日尚無記錄，開始記帳吧！"

  @edge-case @US-201
  Scenario: 無上週同天資料可比較 (系統剛啟用)
    Given today is the first Wednesday since the system was deployed
    When I open the app home page
    Then the week-over-week comparison should display "無歷史資料可比較"
```

---

### Feature 2-2: 營收趨勢圖 (US-202)

```gherkin
# Feature: 營收趨勢圖
# 目的: 透過折線圖識別營收模式 (例如週末比平日高)。
# 對應 PRD: US-202 (P1)

Feature: Revenue Trend Line Chart

  Background:
    Given I am a logged-in shop owner
    And I am on the daily overview page

  # --- Happy Path ---

  @happy-path @US-202
  Scenario: 查看 7 日營收趨勢
    Given revenue data exists for the past 7 days
    When I view the "Revenue Trend" chart
    Then the chart should default to "7 days" view
    And the X-axis should show dates for the past 7 days
    And the Y-axis should show daily revenue amounts

  @happy-path @US-202
  Scenario: 切換至 30 日營收趨勢
    When I view the "Revenue Trend" chart
    And I switch the view to "30 days"
    Then the chart should display data for the past 30 days

  @happy-path @US-202
  Scenario: 點擊圖表上的資料點查看詳情
    Given the 7-day trend chart is displayed
    When I tap on the data point for "2026-02-10"
    Then I should see the detailed revenue info for "2026-02-10"

  # --- Edge Case ---

  @edge-case @US-202
  Scenario: 資料不足 7 天
    Given the system has only been used for 3 days
    When I view the "Revenue Trend" chart in "7 days" mode
    Then the chart should display only 3 data points
    And the remaining days should be empty
```

---

### Feature 2-3: 今日品項銷售排行 (US-203)

```gherkin
# Feature: 今日品項銷售排行
# 目的: 讓店主知道哪些產品最受歡迎。
# 對應 PRD: US-203 (P1)

Feature: Daily Item Sales Ranking

  Background:
    Given I am a logged-in shop owner
    And I am on the daily overview page

  # --- Happy Path ---

  @happy-path @US-203
  Scenario: 查看今日熱銷 Top 3
    Given the following sales exist for today:
      | item       | quantity | revenue |
      | 美式咖啡   | 15       | 1800    |
      | 拿鐵咖啡   | 12       | 1800    |
      | 卡布奇諾   | 8        | 1120    |
      | 手沖咖啡   | 3        | 600     |
    When I view the "Today's Top 3" section
    Then I should see:
      | rank | item     | cups | percentage |
      | 1    | 美式咖啡 | 15   | 39%        |
      | 2    | 拿鐵咖啡 | 12   | 32%        |
      | 3    | 卡布奇諾 | 8    | 21%        |

  @happy-path @US-203
  Scenario: 點擊查看全部排行
    When I tap "View All" on the sales ranking section
    Then I should see the complete sales ranking for today

  # --- Edge Case ---

  @edge-case @US-203
  Scenario: 今日銷售品項不足 3 種
    Given today only has sales for "美式咖啡" (5 cups)
    When I view the "Today's Top 3" section
    Then I should see only 1 item in the ranking
```

---

## Epic 3: 月報表自動生成 (Monthly Report)

### Feature 3-1: 月報表自動產生 (US-301)

**檔案名稱**: `epic_monthly_report.feature`

```gherkin
# Feature: 月報表自動產生
# 目的: 每月 1 日自動產生上月財務報表，店主免手動整理。
# 對應 PRD: US-301 (P0)

Feature: Automatic Monthly Report Generation

  Background:
    Given I am a logged-in shop owner

  # --- Happy Path ---

  @happy-path @smoke-test @US-301
  Scenario: 每月 1 日自動產生上月報表
    Given today is "2026-03-01" at "08:00"
    And February 2026 has complete transaction data
    Then the system should automatically generate the February 2026 report
    And the report should contain:
      | section                  |
      | 本月總淨利               |
      | 本月 vs 上月淨利比較     |
      | 成本結構圓餅圖           |
      | 熱銷品項排行             |

  @happy-path @US-301
  Scenario: 首頁顯示新月報提示
    Given the February 2026 monthly report has been generated
    When I open the app home page
    Then I should see a notification "新月報已就緒：2026 年 2 月"
    When I tap the notification
    Then I should be navigated to the February 2026 report page

  @happy-path @US-301
  Scenario: 月報表顯示本月 vs 上月淨利比較
    Given February 2026 net profit is 45000
    And January 2026 net profit is 40000
    When I view the February 2026 monthly report
    Then I should see the net profit comparison "比上月增加 5,000 元 ↑12.5%"

  # --- Edge Case ---

  @edge-case @US-301
  Scenario: 系統首月無上月資料可比較
    Given January 2026 is the first month of system usage
    And there is no December 2025 data
    When I view the January 2026 monthly report
    Then the month-over-month comparison should display "首月報表，無歷史資料可比較"

  @edge-case @US-301
  Scenario: 報表生成時間應在 5 秒內
    Given today is "2026-03-01" at "08:00"
    When the system generates the February 2026 report
    Then the report generation should complete within 5 seconds
```

---

### Feature 3-2: 成本結構圓餅圖 (US-302)

```gherkin
# Feature: 成本結構圓餅圖
# 目的: 讓店主直觀了解錢花在哪裡。
# 對應 PRD: US-302 (P0)

Feature: Cost Structure Pie Chart in Monthly Report

  Background:
    Given I am a logged-in shop owner
    And I am viewing the monthly report for "February 2026"

  # --- Happy Path ---

  @happy-path @smoke-test @US-302
  Scenario: 查看成本結構圓餅圖
    Given February 2026 has the following expenses:
      | category | amount |
      | 進貨成本 | 20000  |
      | 固定支出 | 18000  |
      | 設備維修 | 3000   |
    When I view the cost structure pie chart
    Then I should see:
      | category | percentage |
      | 進貨成本 | 49%        |
      | 固定支出 | 44%        |
      | 設備維修 | 7%         |

  @happy-path @US-302
  Scenario: 點擊圓餅圖區塊展開明細
    When I tap on the "進貨成本" slice of the pie chart
    Then I should see the detailed expense list under "進貨成本"

  @happy-path @US-302
  Scenario: 查看成本與上月比較
    Given February "進貨成本" is 20000
    And January "進貨成本" was 18000
    When I view the cost structure pie chart
    Then I should see "進貨成本" with a label "比上月 +11.1%"

  # --- Edge Case ---

  @edge-case @US-302
  Scenario: 當月無支出記錄
    Given February 2026 has no expense records
    When I view the cost structure pie chart
    Then I should see a message "本月無支出資料"
```

---

### Feature 3-3: 熱銷品項排行 (US-303)

```gherkin
# Feature: 熱銷品項排行 (整月統計)
# 目的: 透過整月銷售數據協助採購決策。
# 對應 PRD: US-303 (P0)

Feature: Monthly Best-Selling Item Ranking

  Background:
    Given I am a logged-in shop owner
    And I am viewing the monthly report for "February 2026"

  # --- Happy Path ---

  @happy-path @smoke-test @US-303
  Scenario: 查看整月銷量排行 Top 10
    Given February 2026 has sales data for more than 10 items
    When I view the monthly best-selling ranking
    Then I should see the top 10 items ranked by quantity sold
    And each item should display: item name, cups sold, total revenue, and percentage

  @happy-path @US-303
  Scenario: 切換為營收排行
    When I view the monthly best-selling ranking in "Sales Volume" mode
    And I switch to "Revenue" mode
    Then the ranking should re-sort items by total revenue instead of quantity

  # --- Edge Case ---

  @edge-case @US-303
  Scenario: 當月銷售品項不足 10 種
    Given February 2026 has sales data for only 5 items
    When I view the monthly best-selling ranking
    Then I should see all 5 items ranked
```

---

### Feature 3-4: 歷史月報表查詢 (US-304)

```gherkin
# Feature: 歷史月報表查詢
# 目的: 讓店主回顧過去任意月份的營運數據。
# 對應 PRD: US-304 (P1)

Feature: Historical Monthly Report Lookup

  Background:
    Given I am a logged-in shop owner
    And I am on the "Reports" page

  # --- Happy Path ---

  @happy-path @US-304
  Scenario: 選擇過去月份查看報表
    Given monthly reports exist for January and February 2026
    When I select "January 2026" from the month picker
    Then I should see the complete January 2026 report
    And the report should contain: net profit, cost pie chart, and sales ranking

  @happy-path @US-304
  Scenario: 在網頁版查看歷史報表
    Given I am logged in on the web version
    When I navigate to "Reports" and select "February 2026"
    Then I should see the interactive February 2026 report

  # --- Edge Case ---

  @edge-case @US-304
  Scenario: 選擇尚未產生報表的月份
    Given it is currently mid-February 2026
    When I try to select "February 2026" from the month picker
    Then the system should display "本月報表尚未產生，將於 3 月 1 日自動生成"

  @edge-case @US-304
  Scenario: 選擇系統啟用前的月份
    Given the system was deployed in January 2026
    When I try to select "December 2025"
    Then the month picker should not allow selecting months before January 2026
```

---

## Epic 4: 使用者管理與基本權限 (User Management)

### Feature 4-1: 新增員工帳號 (US-401)

**檔案名稱**: `epic_user_management.feature`

```gherkin
# Feature: 新增員工帳號
# 目的: 讓店主新增員工帳號以協助日常記帳。
# 對應 PRD: US-401 (P0)

Feature: Employee Account Creation

  Background:
    Given I am a logged-in shop owner
    And I navigate to "Settings" > "Team Members"

  # --- Happy Path ---

  @happy-path @smoke-test @US-401
  Scenario: 成功新增一個員工帳號
    When I tap the "Add Member" button
    And I fill in the employee name "王小明"
    And I fill in the email "xiaoming@example.com"
    And I set the initial password "Temp1234!"
    And I tap the "Save" button
    Then the employee "王小明" should be created successfully
    And "王小明" should be able to log in with "xiaoming@example.com" and "Temp1234!"

  @happy-path @US-401
  Scenario: 員工使用帳密登入後可開始記帳
    Given the employee "王小明" has been created
    When "王小明" logs in with their credentials
    Then "王小明" should see the main recording page
    And "王小明" should be able to create transactions

  # --- Sad Path ---

  @sad-path @US-401
  Scenario: 使用已存在的 Email 新增員工
    Given an employee with email "xiaoming@example.com" already exists
    When I tap the "Add Member" button
    And I fill in the email "xiaoming@example.com"
    And I tap the "Save" button
    Then I should see an error message "此 Email 已被使用"

  @sad-path @US-401
  Scenario Outline: 新增員工時的輸入驗證
    When I tap the "Add Member" button
    And I fill in name "<name>" and email "<email>" and password "<password>"
    And I tap the "Save" button
    Then I should see a validation error "<message>"

    Examples:
      | name | email                | password   | message               |
      |      | test@example.com     | Temp1234!  | 姓名為必填            |
      | 王小明 |                    | Temp1234!  | Email 為必填          |
      | 王小明 | invalid-email      | Temp1234!  | Email 格式不正確      |
      | 王小明 | test@example.com   |            | 密碼為必填            |

  # --- Edge Case ---

  @edge-case @US-401
  Scenario: 帳號數量達到上限 (10 個)
    Given there are already 10 accounts (1 owner + 9 employees)
    When I tap the "Add Member" button
    Then I should see a message "已達帳號上限 (10 個)，請刪除現有帳號後再新增"
```

---

### Feature 4-2: 員工權限設定 (US-402)

```gherkin
# Feature: 員工權限設定
# 目的: 讓店主控制員工可存取的功能，保護敏感財務資訊。
# 對應 PRD: US-402 (P1)

Feature: Employee Permission Settings

  Background:
    Given I am a logged-in shop owner
    And the employee "王小明" exists

  # --- Happy Path ---

  @happy-path @smoke-test @US-402
  Scenario: 設定員工為「員工」角色 (僅可記帳)
    When I navigate to "王小明" permission settings
    And I set the role to "Employee"
    And I tap the "Save" button
    Then "王小明" should only see the recording features after login
    And "王小明" should NOT see the reports section

  @happy-path @US-402
  Scenario: 設定員工為「管理者」角色 (可記帳 + 查看報表)
    When I navigate to "王小明" permission settings
    And I set the role to "Manager"
    And I tap the "Save" button
    Then "王小明" should see both recording features and reports after login

  # --- Sad Path ---

  @sad-path @US-402
  Scenario: 員工角色嘗試存取報表頁面
    Given "王小明" has the "Employee" role
    When "王小明" tries to navigate to the "Reports" page
    Then "王小明" should see a message "您沒有權限查看此頁面"

  # --- Edge Case ---

  @edge-case @US-402
  Scenario: 店主角色不可被降級
    Given I am the shop owner
    When I navigate to my own permission settings
    Then the role should be locked as "Owner" and cannot be changed
```

---

### Feature 4-3: 記錄者追蹤 (US-403)

```gherkin
# Feature: 記錄者追蹤
# 目的: 每筆記錄標註由誰新增，方便追溯負責人。
# 對應 PRD: US-403 (P2)

Feature: Transaction Recorder Tracking

  Background:
    Given I am a logged-in shop owner

  # --- Happy Path ---

  @happy-path @US-403
  Scenario: 每筆記錄顯示記錄者
    Given "王小明" has created a transaction "美式咖啡 x2 $240"
    When I view the transaction detail
    Then I should see the "Recorded by" field showing "王小明"

  @happy-path @US-403
  Scenario: 篩選特定人員的記錄
    Given both "王小明" and "李小華" have created transactions today
    When I filter transactions by recorder "王小明"
    Then I should only see transactions created by "王小明"

  @happy-path @US-403
  Scenario: 查看每個人的本月記帳統計
    Given "王小明" has recorded 50 transactions this month
    And "李小華" has recorded 30 transactions this month
    When I view the "Team Recording Stats" section
    Then I should see:
      | member | records_count |
      | 王小明 | 50            |
      | 李小華 | 30            |

  # --- Edge Case ---

  @edge-case @US-403
  Scenario: 員工帳號被刪除後，其歷史記錄仍保留記錄者資訊
    Given "王小明" has created transactions
    And "王小明" account has been deleted
    When I view "王小明" previously created transactions
    Then the "Recorded by" field should still show "王小明 (已刪除)"
```

---

## 附錄：標籤 (Tag) 分類說明

| Tag | 說明 |
| :--- | :--- |
| `@happy-path` | 正常流程，預期成功 |
| `@sad-path` | 異常流程，預期失敗或錯誤處理 |
| `@edge-case` | 邊界條件、極端情境 |
| `@smoke-test` | 冒煙測試，最核心的快速驗證場景 |
| `@US-XXX` | 對應 PRD 的使用者故事 ID |

---

## 文件變更歷史 (Change Log)

| 版本 | 日期 | 變更描述 | 作者 |
| :--- | :--- | :--- | :--- |
| v1.0 | 2026-02-11 | 初版完成，涵蓋 4 個 Epic、15 個 User Story 的完整 BDD 情境 | [技術負責人] |
