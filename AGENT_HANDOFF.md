# Agent Handoff: Oreno CRM Platform QA

Welcome! If you are an agentic IDE (like Claude Code, Cursor, or another instance of Antigravity) picking up this repository, this document provides the exact context, environment, and task instructions you need to continue the work seamlessly.

## 📂 Repository Structure

The workspace has been structured to separate raw assets from core documentation:

- `AGENT_HANDOFF.md` - (This file) Your starting point and task instructions.
- `TaskInstructions14may.md` - The raw new task requirements provided by the user.
- `test_report.md` - The master QA Audit Report from our previous testing pass. **You will be updating this file.**
- `screenshots/` - A directory containing all 58 screenshots taken during previous bug discoveries. (These are already pushed to a GitHub repo for public image URLs in the markdown report, so please do not modify the GitHub URLs in `test_report.md`).

---

## 🌐 Environment & Credentials

- **Target URL:** `https://oreno.basiq360.tech`
- **Testing Approach:** You should use your browser subagent/tools (e.g., Playwright MCP) to interact with the website. 
- **Login Credentials:**
  - **Username:** `oreno_admin`
  - **Password:** `Admin@123`

> **CRITICAL FIRST STEP:** You MUST explicitly log out of the admin portal and log back in before starting. The side navigation menu is cached in the browser session. Without a fresh login, you will see the old menu structure and fail your tests.

---

## 🎯 Your Mission (Next Steps)

Your current objective is to execute the instructions defined in `TaskInstructions14may.md`, focusing **strictly on the website portion**.

Please execute the following steps in order:

### 1. Fresh Authentication
Launch the browser, navigate to the portal, and if a session is active, **log out**. Log back in using the credentials above to ensure the newest sidebar navigation is loaded.

### 2. Verify Resolved Issues (Part A)
Check the left navigation menu to verify that the following UI changes have successfully deployed:
- [ ] *Holidays* module is removed.
- [ ] *Chat* module is removed.
- [ ] *Accounts* module is removed.
- [ ] *Purchase Based* module is removed.
- [ ] "Reward & Gift" is renamed to "Rewards & Gifts".
- [ ] "Point History" is renamed to "Scan History".
- [ ] Only ONE "Loyalty Program" entry exists.
- [ ] All SFA-related modules (Orders, Beat Plan, Attendance, Map, Leads, Expense, Target, Task, Leave, Quotation, Event Plan, Pop-Gift, Sites, Followup, Activity, Reports) are gone.
- [ ] Customer hierarchy levels (Primary / Secondary / Direct) are gone (only Influencers should remain).
- [ ] *Tutorials* and *Survey* modules should still be present.

**Action:** Once verified, update the existing `test_report.md` to reflect that these issues have been fixed (e.g., mark them as resolved or cross them out).

### 3. Test the New "Channel Partners" Module (Part B)
Navigate to **Loyalty > Channel Partners** and perform a deep-dive QA test against the provided acceptance criteria:
- **List Page:** Verify the default unassigned partner (`001234`) and the test partner (`100001`).
- **Search & Filter:** Ensure searching by code, name, and mobile works, along with the active/inactive filter.
- **Create Partner:** Test the validation (Code must be exactly 6 digits). Try valid, invalid, and duplicate codes.
- **Edit Partner:** Verify the CP Code is read-only upon editing.
- **Details Page:** Check the Overview and Assigned Electricians tabs.
- **Deactivation:** Verify that the test CP can be deactivated, but the default CP (`001234`) throws an error preventing deactivation.

**Action:** Append a new section to `test_report.md` documenting your findings for the Channel Partners module. If you find bugs, take screenshots and log them clearly. 

> **What to Ignore (Part D):**
> Do not log bugs regarding the "Add Influencer" form layout, the Approve influencer status toggle, missing bulk upload buttons, or date pickers allowing back-dates. These are known issues slated for upcoming phases.

### 4. Finalize
Once you have updated the `test_report.md` with the newly verified fixes and the Channel Partners module test results, review the changes with the user. You do not need to test the Mobile App flow (Part C).
