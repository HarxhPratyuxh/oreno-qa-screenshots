# Oreno CRM Platform — QA Audit Report

**Target URL:** https://oreno.basiq360.tech  
**Testing Date:** 2026-05-11  
**Test Account:** `oreno_admin` (Corporate Login)  
**Total Modules Tested:** 76 unique endpoints across all sidebar modules  
**Total Bugs Found:** 15  

> ⚠️ This document contains **only errors and bugs**. Modules working as expected are listed briefly in [Section 9](#9-modules-working-correctly).

---

## Table of Contents

1. [Global API Failure (CRITICAL)](#1-global-api-failure-critical)
2. [React Crashes on Add Forms (CRITICAL)](#2-react-crashes-on-add-forms-critical)
3. [WebSocket Failures — Map & Chat (HIGH)](#3-websocket-failures--map--chat-high)
4. [API Backend Errors](#4-api-backend-errors)
5. [Module-Specific JavaScript Errors](#5-module-specific-javascript-errors)
6. [Completely Empty/Non-Functional Modules](#6-completely-emptynon-functional-modules)
7. [Data Entry Blockers — Empty Dropdowns (HIGH)](#7-data-entry-blockers--empty-dropdowns-high)
8. [UI/UX Issues](#8-uiux-issues)
9. [Modules Working Correctly](#9-modules-working-correctly)
10. [Data Creation Test Results](#10-data-creation-test-results)
11. [Summary of All Issues](#11-summary-of-all-issues)

---

## 1. Global API Failure (CRITICAL)

A critical structural issue affects **every single page** on the platform.

### `Failed to fetch modules` — Platform-Wide
- **Locations Affected:** ALL routes (confirmed on 76+ unique endpoints)
- **Console Output:**
  ```
  [ERROR] Get assigned modules failed Error: Failed to fetch modules
      at Object.getAssignedModules (main.579a2585.js)
  ```
- **Root Cause:** The `getAssignedModules` API call fails silently on every page load. This cascading failure breaks dynamic dropdowns, data population, and feature rendering across the entire platform.
- **Impact:** **CRITICAL** — This is the #1 root cause behind most other bugs in this report.

![Global Module Error](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/_loyalty_offers_error.png)

---

## 2. React Crashes on Add Forms (CRITICAL)

Multiple modules crash with a full-page React ErrorBoundary (**"Oops! Something went wrong"**) when clicking "Add".

### 2a. User Management — Navigation-Triggered Crash
- **Locations:** `/user/sales` → Click "Add", `/user/backend` → Click "Add"
- **Crash Trigger:** Clicking the "Add" button from the list page triggers:
  ```
  NotFoundError: Failed to execute 'removeChild' on 'Node'
  ```
- **Workaround:** Navigating **directly** to `/user/sales/add` or `/user/backend/add` via URL bar **does load the form** without crashing.
- **However, form submission still fails:**
  - **Backend Staff:** `Role *` dropdown is completely empty (0 options). Validation error: **"Role is required"**
  - **Sales User:** `Designation *` dropdown is empty. Toast: **"Please fix all validation errors"**
- **Impact:** Users cannot be created from the list page. Direct URL requires seeding dropdown data first.

![Backend Staff - Role Required](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/user_backend_role_required.png)

![Sales User - Validation Errors](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/user_sales_save_attempt.png)

### 2b. Announcement — Add Form Crash
- **Location:** `/announcement` → Click "Add"
- **Issue:** List page is empty (0 rows). Clicking "Add" crashes with React ErrorBoundary.
- **Impact:** Cannot create or view any announcements.

![Announcement Add Crash](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/announcement_crash.png)

### 2c. Orders — Primary & Secondary Add Form Crash
- **Location:** `/sfa/orders/primary` → Click "Add" → crashes at `/sfa/orders/primary/create`
- Also: `/sfa/orders/secondary` → Click "Add" → same crash
- **Additional:** Orders list page has a **database query failure** (see [Section 4a](#4a-primary-orders--database-query-failure))
- **Impact:** Cannot create any new orders.

![Orders Add Crash](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/orders_primary_crash.png)

---

## 3. WebSocket Failures — Map & Chat (HIGH)

### 3a. Map Module — Location Tracking Broken
- **Location:** `/sfa/map`
- **Issue:** 16 console errors on load. WebSocket handshake fails.
- **Console Output:**
  ```
  WebSocket connection to 'wss://oreno.basiq360.tech/socket.io/?EIO=4&transport=websocket' failed:
  Error during WebSocket handshake: Unexpected response code: 200
  [LocationTracking] Connection error: websocket error
  ```
- **Root Cause:** Server responds HTTP 200 instead of 101 Switching Protocols.
- **Impact:** Real-time field staff location tracking is completely non-functional.

![Map Module State](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/map_module_state.png)

### 3b. Chat Module — Connection Failure
- **Location:** `/chat`
- **Issue:** Chat UI renders but enters a **"Reconnecting... (Attempt N)"** loop, then shows **"Connection error. Please check configuration."**
- **Observed State:**
  - Status: `Reconnecting... (Attempt 4)` → `Connection error`
  - Online users: `0 online` — `No users online`
  - All input controls: **disabled** (message box, emoji, attach, voice, send)
- **Root Cause:** Same broken `wss://oreno.basiq360.tech/socket.io/` endpoint as Map.
- **Impact:** Internal team communication is completely non-functional.

![Chat Reconnecting](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/chat_module_broken.png)

![Chat Connection Error](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/chat_connection_error.png)

---

## 4. API Backend Errors

### 4a. Primary Orders — Database Query Failure
- **Location:** `/sfa/orders/primary`
- **Console Output:**
  ```
  [ERROR] API Error: GET web/primary-orders/statistics → 400: Database query failed
  [ERROR] Error fetching status counts: Error: Database query failed
  ```
- **Root Cause:** `GET /api/web/primary-orders/statistics?fromDate=&toDate=` sends empty date parameters; backend SQL query crashes instead of handling nulls.
- **Impact:** Order statistics, status counts, and dashboard metrics are all broken.

### 4b. API Authorization Errors — 403 Forbidden
Several Loyalty admin routes return `403 Forbidden` for the `oreno_admin` role.

- **Locations Affected:**
  - `/loyalty/kyc-queue` — missing `view` on `LOYALTY_FRAUD_REVIEW`
  - `/loyalty/point-categories`
  - `/loyalty/referrals`
  - `/loyalty/tier-config`
  - `/loyalty/work-showcase`
  - `/loyalty/bonus-schemes` — missing `view` on `LOYALTY_BONUS_SCHEME`
  - `/loyalty/targeting` — missing `read` on `LOYALTY_INFLUENCER`
- **Impact:** Admin cannot access 7 loyalty administration features.

![403 Forbidden - KYC Queue](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/_loyalty_kyc_queue_error.png)

---

## 5. Module-Specific JavaScript Errors

### 5a. Expense Module Scope Crash
- **Location:** `/sfa/expense`
- **Console:** `TypeError: Cannot read properties of undefined (reading 'maxScope')`
- **Impact:** MEDIUM — Expense reporting UI broken.

![Expense Scope Error](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/_sfa_expense_error.png)

### 5b. Dashboard SVG Rendering Error
- **Location:** `/accounts` (Dashboard)
- **Console:** `Error: <path> attribute d: Expected arc flag ('0' or '1')`
- **Impact:** LOW — Minor visual glitch from malformed SVG.

![Dashboard Rendering Error](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/dashboard.png)

---

## 6. Completely Empty/Non-Functional Modules

These modules load their page shell but render **zero functional content**.

### 6a. Analytics Suite (3 sub-modules)
| Route | Status |
|-------|--------|
| `/analytics/sales-force-automation` | Empty — 0 charts, 0 tables |
| `/analytics/lead-management-service` | Empty — 0 charts, 0 tables |
| `/analytics/influencer-reward-program` | Empty — 0 charts, 0 tables |

![Analytics Empty State](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/analytics_sfa.png)

### 6b. Loyalty Suite (6 modules)
| Route | Status |
|-------|--------|
| `/loyalty/offers` | Blank — 0 interactive elements |
| `/loyalty/rewards-gift` | Blank |
| `/loyalty/purchase-based` | Blank |
| `/loyalty/redemption` | Blank |
| `/loyalty/point-history` | Blank |
| `/loyalty/dashboard` | Blank |

### 6c. Sites Module — Total Failure
- `/sales-force-automation/site` — **0 inputs, 0 tables, 0 buttons**. Renders nothing.

![Sites Module Empty](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/sites_module.png)

### 6d. Orders Module (list views)
| Route | Status |
|-------|--------|
| `/sfa/orders/primary` | 0 tables rendered |
| `/sfa/orders/secondary` | 0 tables rendered |
| `/sfa/orders/tertiary` | 0 tables rendered |

![Orders Module State](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/orders_module.png)

### 6e. Content Master Suite (5 sub-modules)
| Route | Status |
|-------|--------|
| `/app-config/banners` | Content doesn't populate |
| `/app-config/faq` | Content doesn't populate |
| `/app-config/videos` | Content doesn't populate |
| `/app-config/catalogues` | Content doesn't populate |
| `/app-config/contact` | Content doesn't populate |

![Content Master State](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/content_banners.png)

### 6f. Other Empty Modules
| Route | Status |
|-------|--------|
| `/announcement` | 0 rows |
| `/sfa/reports` | 0 tables |
| `/tutorial-videos` | No content |
| `/basiq-gpt` | No content |
| `/loyalty/analytics` | 0 charts |
| `/authorization/roles` | 0 rows (now has "Test Admin") |
| `/authorization/permissions` | 0 tables |

![BasiqGPT State](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/basiqgpt_module.png)

---

## 7. Data Entry Blockers — Empty Dropdowns (HIGH)

Many Add forms open successfully, but required dropdown fields are empty, making data entry impossible.

### 7a. Modules Blocked by Empty Dropdowns

| Module | Add Form URL | Empty Dropdowns | Can Save? |
|--------|-------------|-----------------|-----------|
| **Expense** | `/sfa/expense/add` | "Select User" | ❌ Save disabled |
| **Support** | `/support/add` | "Select User", "Select Type", "Select Priority", "Select Staff to Assign" | ❌ Validation fails |
| **Beat Plan** | `/sfa/beat-plan/add` | "Select Sales User", "Select Customers" | ❌ |
| **Leave** | `/sfa/leave/add` | "Select user...", "Select Leave Type" | ❌ No Save button |
| **Target** | `/sfa/target/add` | "Select User" | ❌ |
| **Task** | `/sfa/task/add` | Two "Select..." dropdowns | ❌ |
| **Customer Primary** | `/customer/primary-network/add` | "Select Sub Type", "Select sales users..." | ❌ |
| **Customer Secondary** | `/customer/secondary-network/add` | "Select Sub Type", "Select Primary Networks...", "Select sales users..." | ❌ |
| **Customer Influencer** | `/customer/influencer-network/add` | "Select Sub Type", "Select sales users..." | ❌ |
| **Customer Direct** | `/customer/direct-network/add` | "Select Sub Type", "Select sales users..." | ❌ |
| **Product** | `/product/add` | "Select Category" | ❌ |
| **Attendance** | `/sfa/attendance` | No "Add" button exists | N/A |
| **User (Backend/Sales)** | `/user/backend/add` | "State" = "No options" | ⚠️ Partial |
| **Territory Master** | `/territory-master` | Add button does nothing | N/A |

### 7b. Modules Where Add Forms Work

| Module | URL | Status |
|--------|-----|--------|
| Leads | `/leads/add` | ✅ All fields populate, Save works |
| Holiday | `/holiday/add` | ✅ No dropdown issues |
| Postal | `/postal/add` | ✅ No dropdown issues |
| Survey | `/survey/add` | ✅ No dropdown issues |
| Follow Up | `/sfa/followup/add` | ✅ No dropdown issues |
| Event Plan | `/sfa/event-plan/add` | ✅ No dropdown issues |
| Quotation | `/sfa/quotation/add` | ✅ No dropdown issues |
| Roles | `/authorization/roles/add` | ✅ Works |
| Loyalty Coupons | `/loyalty/coupons/add` | ✅ Works |
| Loyalty Gifts | `/loyalty/gifts/add` | ✅ Works |
| Loyalty Campaigns | `/loyalty/campaigns/add` | ✅ Works |
| Loyalty Bonus Schemes | `/loyalty/bonus-schemes/add` | ✅ Works |

### Root Cause
The `Failed to fetch modules` error prevents loading user lists and config data for dropdowns. Any form requiring a user, customer type, or category selection is blocked.

---

## 8. UI/UX Issues

### 8a. React Joyride Tour Overlay Blocks Interactions
- **Locations:** `/sfa/expense`, `/user/sales`, `/user/backend`, and other modules with "Page Tour"
- **Issue:** Tutorial overlay intercepts all pointer events, making buttons unclickable. "Skip" button sometimes fails.
- **Impact:** MEDIUM — Users cannot interact with forms until overlay is manually removed.

---

## 9. Modules Working Correctly

| Module | URL | Status |
|--------|-----|--------|
| Leads | `/leads` | ✅ Full CRUD works |
| Customer Network (Primary) | `/customer/primary-network` | ✅ Table loads with data |
| Customer Network (Secondary) | `/customer/secondary-network` | ✅ Table loads |
| Customer Network (Influencer) | `/customer/influencer-network` | ✅ Table loads |
| Customer Network (Direct) | `/customer/direct-network` | ✅ Table loads |
| Product Catalog | `/product/list` | ✅ Table loads |
| Product Price Manage | `/product/price-manage` | ✅ Table loads |
| Product Discount Manage | `/product/discount-manage` | ✅ Table loads |
| Dropdown Management | `/dropdown-management` | ✅ **Fully functional** (CRUD works) |
| Activity (Check-in) | `/sfa/checkin` | ✅ Table loads |
| Beat Plan | `/sfa/beat-plan` | ✅ Table loads |
| Event Plan | `/sfa/event-plan` | ✅ Table loads |
| Follow Up | `/sfa/followup` | ✅ Table loads |
| Holiday | `/holiday` | ✅ Table loads |
| Leave | `/sfa/leave` | ✅ Table loads |
| Pop-Gift | `/sfa/pop-gift` | ✅ Table loads |
| Quotation | `/sfa/quotation` | ✅ Table loads |
| Survey | `/survey` | ✅ Table loads |
| Target | `/sfa/target` | ✅ Table loads |
| Task | `/sfa/task` | ✅ Table loads |
| Postal | `/postal` | ✅ Table loads |
| Session Management | `/session-management` | ✅ Table loads |
| Loyalty Coupons | `/loyalty/coupons` | ✅ Table loads |
| Loyalty Gifts | `/loyalty/gifts` | ✅ Table loads |
| Loyalty Campaigns | `/loyalty/campaigns` | ✅ Table loads |
| Loyalty Fraud Queue | `/loyalty/fraud-queue` | ✅ Table loads |
| Loyalty OCR Receipts | `/loyalty/receipts` | ✅ Table loads |
| Loyalty Bonus Schemes | `/loyalty/bonus-schemes` | ✅ Table loads |
| Keyboard Shortcut | `/keyboard-shortcut` | ✅ Displays shortcuts |

> **Note:** "Table loads" means the data table renders. All modules still show the global `Failed to fetch modules` console error and may have empty dropdowns in Add/Edit forms.

---

## 10. Data Creation Test Results

### ✅ Successfully Created
| Record | Module | Details |
|--------|--------|---------|
| Lead "Test User 347" | `/leads/add` | Phone: `9319180958`, Email: `test@gmail.com`. Duplicate detection works (409 Conflict). |
| Backend Staff "John Doe" | `/user/backend/add` | Email: `john.doe@example.com`, Phone: `9876543210`. Required seeding Role first. |

![Backend User Created](https://raw.githubusercontent.com/HarxhPratyuxh/oreno-qa-screenshots/main/backend_user_created_success.png)

### ❌ Failed to Create
| Record | Module | Reason |
|--------|--------|--------|
| Sales User | `/user/sales/add` | Crashes from list; direct URL blocked by empty Designation dropdown |
| Expense | `/sfa/expense/add` | "Select User" empty, Save disabled |
| Support Ticket | `/support/add` | 4 dropdowns all empty |

### ⚠️ New Bug — State Dropdown "No Options"
In the User Add form, the **State** dropdown (Address Information) shows **"No options"**. The state/district master data is not loaded. Users cannot enter address details.

### 🔧 Workaround — Seeding Dropdown Data
The following data was added to unblock user creation:

| Data | Where Created | Value |
|------|--------------|-------|
| Role | `/authorization/roles/add` | "Test Admin" (`TEST_ADMIN`) |
| Designation | Dropdown Management → User | "Sales Executive" (`SALES_EXECUTIVE`) |
| Department | Dropdown Management → User | "Testing" (`TESTING_DEPT`) |
| Leave Type | Dropdown Management → Leave | "Casual Leave" (`CASUAL_LEAVE`) |

> **Dropdown Management** (`/dropdown-management`) is the only fully working CRUD tool and the key workaround for seeding data. However, some dropdowns (State, Sales User lists, Reporting Manager) pull from backend master data that cannot be seeded via the UI.

---

## 11. Summary of All Issues

| # | Priority | Issue | Impact |
|---|----------|-------|--------|
| 1 | 🔴 CRITICAL | `getAssignedModules` API fails globally | Breaks data population on ALL pages |
| 2 | 🔴 CRITICAL | User Add crash when clicked from list page | React `removeChild` DOM error |
| 3 | 🔴 CRITICAL | Announcement Add crashes (React ErrorBoundary) | Cannot create announcements |
| 4 | 🔴 CRITICAL | Orders Primary/Secondary Add crashes | Cannot create any orders |
| 5 | 🟠 HIGH | Chat WebSocket connection failure | Internal messaging completely dead |
| 6 | 🟠 HIGH | Map WebSocket handshake failure | Location tracking non-functional |
| 7 | 🟠 HIGH | Primary Orders DB query failure (400) | Order stats/metrics broken |
| 8 | 🟠 HIGH | 403 Forbidden on 7 Loyalty admin routes | Admin cannot manage loyalty features |
| 9 | 🟠 HIGH | 12+ modules have empty dropdowns in Add forms | Blocks data creation across platform |
| 10 | 🟠 HIGH | State dropdown "No options" in User forms | Cannot enter address data |
| 11 | 🟠 HIGH | 20+ modules render completely empty | Major feature gaps |
| 12 | 🟡 MEDIUM | React Joyride overlay blocks interactions | UX degradation on multiple forms |
| 13 | 🟡 MEDIUM | Expense `maxScope` TypeError | Expense UI broken |
| 14 | 🟡 MEDIUM | Territory Master Add button does nothing | Cannot add territories |
| 15 | 🟢 LOW | SVG path rendering warning | Minor visual glitch |

### Recommended Fix Priority

```
1. Fix getAssignedModules API endpoint  →  Resolves bugs #1, #9, #11
2. Fix React removeChild crash           →  Resolves bug #2
3. Configure WebSocket server            →  Resolves bugs #5, #6
4. Fix Orders statistics API             →  Resolves bugs #4, #7
5. Audit RBAC permissions                →  Resolves bug #8
6. Load State/District master data       →  Resolves bug #10
```

---

*Generated via automated Playwright testing on 2026-05-11. All screenshots captured live during testing.*
