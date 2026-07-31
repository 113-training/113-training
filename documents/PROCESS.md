# PROCESS.md — 我的練習心得

> 一個原則：**寫「具體發生的事」，不寫感想文。**
> 貼上當時真實的 prompt、真實的數字、真實的錯誤訊息——三個月後的你（和你的同事）才用得上。

#### 使用的 agent 與模型：

---

## 通用四問

### 1. 我的任務拆解

（開工前你把任務拆成哪幾步？實際做的時候順序有變嗎？為什麼變？）

- 根據問題開工單
- 排查問題原因
- 修復問題
- 開PR去master

### 2. AI 幫上大忙的地方

（哪件事 agent 做得又快又好？**貼上當時的提問原文**，說明為什麼這樣問有效。）

Investigate a bug in the OrderHub order listing flow.

## Problem

A newly created order is successfully saved in the database, but it does not appear anywhere in the Orders UI.

I checked all available page indexes in the UI, but the newly created order ID is not included in any page.

The database record exists, so investigate whether the issue is caused by the order listing logic, filtering, sorting, pagination, mapping, EF Core query, repository logic, or SQL.

## Starting point

Begin with:

`training-repo/src/OrderHub.Web/Controllers/OrdersController.cs`

Trace every method involved in loading and displaying the order list, including:
* Controller actions
* Services and service interfaces
* Repository methods and repository interfaces
* EF Core queries
* ViewModels and DTO mappings
* Razor views or frontend request parameters
* Pagination calculations
* Filtering and search conditions
* Sorting logic
* Any raw SQL, stored procedures, or database views used by the listing flow

Also inspect the order creation flow where necessary to compare the saved data against the conditions used by the listing query.

Verify the following carefully:
1. Whether the newly created order has field values that cause it to be excluded by the listing query.
2. Whether there are default filters for status, customer, date, deletion flag, active flag, or other fields.
3. Whether nullable fields or default values are handled incorrectly.
4. Whether the query uses an inner join that excludes the order because related data is missing.
5. Whether the order creation transaction is committed successfully.
6. Whether the listing query reads from the same database, table, schema, or environment as the creation flow.
7. Whether sorting and pagination are applied in the correct order.
8. Whether `Skip` and `Take` calculations are correct for the supplied `PageIndex`.
9. Whether the total count query and data query use different filtering conditions.
10. Whether the UI uses a zero-based page index while the backend expects a one-based page index, or vice versa.
11. Whether an unstable or non-unique sort order causes records to be skipped between pages.
12. Whether a mapping issue removes or overwrites the order ID.
13. Whether cached data, `AsNoTracking`, query filters, or EF Core global query filters affect the result.
14. Whether the listing query contains hardcoded conditions that do not match newly created orders.
15. Whether raw SQL or a stored procedure has incorrect conditions, joins, ordering, or pagination logic.

- 排查問題比人為排查快速，可以快速確認邏輯是不是有寫錯的。

### 3. AI 誤導我的地方，與我如何發現

（agent 說錯／改錯／過度自信的時刻。你靠什麼抓到——對照程式碼？頁面實測？跑測試？）

- 我會先查看排查結果和修復方案，問題原因準確且修復方案可行才執行，完成後查看修改的代碼確保沒有額外的調整，之後在頁面測試確保問題解決

### 4. 我會帶回日常工作的一招

（一個具體、可複製的做法，不要寫「要多驗證」這種口號——寫出**操作步驟**。）

- 列出具體排查方向，從哪個controller開始排查和列出排查那些比較可能的錯誤。需要可以縮小範圍，也更節省Token

## 自我驗證（做到哪個階段答哪題）

### 第一階段 — Agentic Coding

練習 1

1. 我能不看筆記說出三個專案（Web/Core/Infrastructure）各自的職責
2. 我核對過 agent 描述的建單流程，且**至少找出一處不精確或過度簡化的說法**
3. 我知道商業邏輯應該放在哪一層、新增頁面要動哪些地方

練習 2

1. 三個 bug 我都先在頁面上重現過，才開始找程式
2. 我給 agent 的資訊包含具體觀察（頁碼／金額數字／庫存數字），而不是只貼客訴原文
3. 每個修復都回到頁面驗證過症狀消失
4. 每個 bug 都補了一個回歸測試，`dotnet test` 全綠
5. 三個獨立 commit，message 說明症狀與根因
6. （思考題）為什麼原本的測試沒抓到這三個 bug？

練習 3

1. `/Products/LowStock` 不帶參數 → 門檻 10 的結果；帶 `?threshold=3` → 結果隨之改變
2. `?threshold=0`、`?threshold=-1` → 頁面顯示驗證錯誤，不是 500
3. 售出數量欄位排除了 Cancelled 訂單（可用一筆已取消的訂單驗證）
4. 停售（已停售 badge）商品不出現在列表
5. 程式分層與命名跟既有的 Products 功能一致（請 agent 自我 review 一次，並自己確認）
6. 至少 3 個新測試，`dotnet test` 全綠

練習 4

1. 重構後 `dotnet test` 全綠
2. 我能說出這次重構「改善了什麼、沒有改變什麼」
3. 我有在 code review 的角度看過 diff（不是 agent 說好就好）

### 第二階段 — MCP Server

練習 0
1. 更快更方便的如果用agent 可以自己做，agent可以直接展示有問題的部分，人工的話需要自己一個一個對照

練習 1
1. 新增了GetOrder，LowStock，CustomerOrders
2. GetOrder 用Id查詢訂單
3. LowStock 查詢低於庫存門檻的商品
4. CustomerOrders 用客戶Id查詢，該客戶的全部訂單

練習 2
1. 可以更清楚的知道是什麼錯誤，因為有清楚的錯誤信息

練習 3
1. 有MCP Agent可以更直接的用Tools查找，而不是跟著代碼邏輯一個一個去進行
2. 可以更快完成指令

練習 4
1. 可以直接幫忙取消訂單，也能在執行取消已取消訂單時，給到為什麼不能取消的原因

練習 5
1. 
---

## 附錄：值得留下的對話片段

（貼 1–2 段最有代表性的 prompt 與回應**摘要**——不用貼全文，重點是「我怎麼問」和「它怎麼答」。）
- Investigate a bug in the OrderHub order listing flow.
## Problem
A newly created order is successfully saved in the database, but it does not appear anywhere in the Orders UI.
I checked all available page indexes in the UI, but the newly created order ID is not included in any page.
The database record exists, so investigate whether the issue is caused by the order listing logic, filtering, sorting, pagination, mapping, EF Core query, repository logic, or SQL.