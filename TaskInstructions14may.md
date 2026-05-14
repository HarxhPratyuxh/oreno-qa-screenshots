Before you start: log out of the admin portal and log back in. The menu is cached in your browser session — without a fresh login you'll see the old menu.

  ---
  A) Verify these previously-reported issues are now fixed

  Tick each one in your test report.

  - Holidays module is no longer in the left nav (was UAT 5)
  - Chat module is no longer in the left nav (was UAT app testing)
  - Accounts module (Invoices + Payments) is no longer in the left nav (UAT 1)
  - Purchase Based module is no longer in the left nav (UAT 8)
  - "Reward & Gift" is now spelled "Rewards & Gifts" under Loyalty Program (UAT 9.a)
  - "Point History" is now labeled "Scan History" in the menu (UAT 7.b — note: the page columns themselves are still the old layout; redesign comes in Phase 5)
  - No duplicate Loyalty Dashboard — only one Loyalty Program entry in nav (UAT 13.a)
  - Sales-related modules are gone: Orders, Beat Plan, Attendance, Map, Leads, Expense, Target, Task, Leave, Quotation, Event Plan, Pop-Gift, Sites, Followup, Activity, Reports — none should appear in the left nav
  - Customer hierarchy levels gone: Primary / Secondary / Direct customer entries should NOT appear (only Influencers remains, under Loyalty)
  - Tutorials and Survey modules still appear (kept on purpose — these are common modules, not SFA)

  ---
  B) NEW feature — Channel Partners

  This is brand new in this pass. Please exercise it thoroughly.

  Where: look for "Channel Partners" in the left nav (under the loyalty section).

  Tests:

  1. List page — should show 2 entries:
    - 001234 — Default Channel Partner (Unassigned) — flagged as default, cannot be deactivated, shows 1 assigned electrician (the Test Electrician from previous testing)
    - 100001 — Mumbai Test CP — created during developer smoke-test, can be edited/deactivated
  2. Search — try searching by code (001234), by name (Mumbai), and by mobile. All should filter the list.
  3. Filter — test the Status filter (active/inactive).
  4. Create new CP: click "Add", fill the form. CP Code must be exactly 6 digits (e.g., 100002, 200001). Try:
    - Valid: 100003 / "Delhi Test CP" / mobile + city + state
    - Invalid: 1234 (4 digits) → should show validation error
    - Invalid: ABC123 (alphanumeric) → should show validation error
    - Duplicate: try 001234 again → should show "code already exists" error
  5. Edit CP — open Mumbai Test CP → edit form. The CP Code field should be read-only (you cannot change a CP's code after creation). Other fields should be editable. Save and confirm changes persist.
  6. Detail page — click into default CP (001234):
    - Overview tab: shows all CP fields
    - Assigned Electricians tab: should show the 1 test electrician (mobile 9999999999) assigned to this CP
  7. Deactivate — try deactivating Mumbai Test CP (should succeed). Try deactivating default 001234 (should be rejected with a clear error message).

  ---
  C) Mobile app flow (sanity check, no changes since last test)

  Just confirm these still work after our backend changes — should be identical to last pass:

  - Mobile login with 9999999999 + OTP 000000 succeeds
  - Test Electrician's profile shows them assigned to default CP 001234
  - Wallet balance shows correctly
  - (If you have the test QR codes from earlier — TMT-TEST01-80440 etc.) scanning them credits 50 points each

  ---
  D) Things you'll see that we already know about — don't report these

  These are scheduled for upcoming phases. Skip them in your report:

  ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬─────────┐
  │                                                       What                                                        │                                                                   Why                                                                    │  Phase  │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ "Add Influencer" form still has SFA-flavored tabs (orders, check-ins, credit summary) and doesn't ask for CP code │ Phase 2 builds the loyalty-native replacement — kept old version functional in the meantime                                              │ Phase 2 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Approve influencer from list still toggles customer.status instead of KYC                                         │ Same form will be replaced in Phase 2                                                                                                    │ Phase 2 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ No bulk upload button for influencers                                                                             │ Phase 4                                                                                                                                  │ Phase 4 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Mobile app "Purchase Based" still crashes                                                                         │ Will be removed in Phase 7                                                                                                               │ Phase 7 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Mobile app "Loyalty Dashboard" / "IRP Analytics" not clickable                                                    │ Same — Phase 7 wires the navigation                                                                                                      │ Phase 7 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Date pickers on Announcements/Survey allow back-dates                                                             │ Phase 5 polish                                                                                                                           │ Phase 5 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Wrong nav highlight when on Add Banner page                                                                       │ Phase 5                                                                                                                                  │ Phase 5 │
  ├───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼─────────┤
  │ Real SMS not sent (OTP only logged + universal 000000 bypass works)                                               │ Wired but uses dev bypass; real SMS toggled via patch already deployed but needs DLT template setup before going to real Indian carriers │ Pending │
  └───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴─────────┘