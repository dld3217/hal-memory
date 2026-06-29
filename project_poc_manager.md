---
name: project-poc-manager
description: "HPE Networking POC Manager — two versions: standalone HTML tool and SPFx SharePoint app"
metadata: 
  node_type: memory
  type: project
  originSessionId: 35b4820e-c1a9-44b0-84f5-4d75d61da277
---

There are two versions of the POC Manager project:

## 1. Standalone HTML version
- Location: `C:\Users\doddd\OneDrive - Hewlett Packard Enterprise\HPE Networking Sales & Engineering - Customer Test Plans\Master POC Files\`
- Key files: `Customer_Info_NEW.html` (master template), `POC_Report_updated.html`, `Customer_Info.html`
- No build system — opens directly in browser

## 2. SPFx SharePoint version ("Workbench")
- Location: `C:\dev\poc-manager-spfx\`
- Stack: SPFx 1.20 + React + TypeScript + Fluent UI v8 + PnPjs v4
- Node 18.20.8 — run `nvm use 18` if gulp/npm commands fail
- Production site: https://hpe.sharepoint.com/teams/hpen-poc-manager
- GitHub: https://github.com/jeffdziedzic/poc-manager-spfx (private) — Jeff Dziedzic is co-collaborator
- App Catalog: `https://hpe.sharepoint.com/teams/hpen-poc-manager/AppCatalog/Forms/AllItems.aspx`
- Deploy script: `.\deploy.ps1 "commit message"` — auto-bumps version, bundles, commits, pushes
- After deploy: always run `.\node_modules\.bin\gulp.cmd package-solution --ship` then upload .sppkg
- After upload: Ctrl+Shift+Delete → clear cache → Ctrl+Shift+R hard refresh
- Current package version: **1.0.95.0** on branch **feature/sfdc-sync**
- **main** branch is the stable deployed baseline (last good: v1.0.3.0)

## Workflow understanding (2026-06-15)
1. AM/TM creates SFDC Task ("Proof of Concept" Presales Sub Type) with dollar value, due date, assigned SE
2. SE creates POC in POC Manager, links to SFDC Task (lookup pending admin approval)
3. SE selects solutions — each auto-loads default test plan
4. Pre-Install checklist completed before testing
5. SE runs tests: Status (Not Started/In Progress/Completed/Waiting on...) + Outcome (TBD/Pass/Fail+reason/Exception+notes)
6. POC status writes back to SFDC Task Status field (pending admin approval)
7. SE exports Customer Report (print-to-PDF signoff doc) for customer approval
8. Management sees pipeline dashboard without touching POC Manager

## SFDC Task fields (for future write-back integration)
- Subject (a) = POC title
- Presales Sub Type (b) = "Proof of Concept"
- Due Date (c) = pocEndDate
- Status (d) = POC status → write-back target
- Priority (e) = importance
- Date Initiated (f) = pocStartDate
- SFDC Status values: Not Started | In Progress | Completed | Waiting on someone else | Deferred | Not Applicable

## POC Status values (CustomerInfoTab)
- Not Started | In Progress | Waiting on... (sub: Customer/Partner/TAC/Other) | Deferred | Not Applicable | Completed
- "Stalled" kept in type for backward compat with old data, displays as "Waiting on..."
- SFDC mapping: all "Waiting on..." variants → "Waiting on someone else"

## Test Status values (SolutionTestTab)
- Not Started | In Progress | Completed | Waiting on Customer | Waiting on Partner | Waiting on TAC | Waiting on Other
- UI shows "Waiting on..." with sub-selector for who
- Outcome field: TBD | Pass | Fail (+ reason) | Exception (+ notes) — stored in outcomeReason field

## Summary tab
- Columns: Solution | Tests | Not Started | In Progress | Completed | Waiting | Pass | Fail/Exc | Completion %
- Completion % = status=Completed / total selected
- Pass/Fail counts from outcome field (not status)

## Pre-Install Checklist (PreInstallTab)
- Items have: status (Not Started/In Progress/Complete/Blocked), owner (SE/Customer/Partner/TAC), notes (expandable), category (Global or solution code)
- Grouped by category with per-section progress bars
- Defaults loaded per solution + global items with pre-assigned owners
- Add item row: pick category, label, owner

## Customer Report export (CustomerReportExport.ts)
- Button on Summary tab: "Customer Report"
- Opens print-ready HTML in new tab with "Print / Save as PDF" banner
- Sections: HPE header → POC metadata grid → outcome banner → exec summary stats → pre-install checklist → test results per solution (page break each) → notes → signature block (customer + SE)
- Failure reasons and exception notes shown prominently in test results

## Pending admin approvals
- **Microsoft Graph / Mail.Send** — email report button
- **Microsoft Flow Service / Flows.Read.All** — SFDC lookup
- To re-enable SFDC lookup after approval: change `onSfdcLookup={undefined}` → `onSfdcLookup={sfLookupFlowUrl ? handleSfdcLookup : undefined}` in PocForm.tsx

## Active branch: feature/sfdc-sync
- All work is on this branch, not yet merged to main
- Current version: **1.0.47.0**, all committed and pushed

## Deploy workaround (known issue)
`.\deploy.ps1 "message"` runs bundle + git commit/push but package-solution silently skips.
After every deploy, manually run: `.\node_modules\.bin\gulp.cmd package-solution --ship`
Then upload the fresh `sharepoint/solution/poc-manager-spfx.sppkg` to the App Catalog.

## JSON migration — COMPLETED 2026-06-18 (v1.0.47.0)
- SP POCs list: 29 custom columns (pipeline/filter/person fields only)
- JSON per-POC: `meta` section stores 29 non-pipeline fields (contacts, SFDC, partner contact, shipping, loan, MEDDPICC, PreInstallDate, POCDate, OpportunityName, TestPlanRevision, Notes)
- Load flow: `getPocById()` reads SP item + JSON meta → merges into IPoc
- Save flow: `updatePoc()` writes SP fields only; `savePocJsonData()` writes `{ ...pocJson, meta: extractMeta(dataToSave) }`
- Pipeline views (Reports.tsx, Dashboard.tsx) load purely from SP — no JSON loading needed
- IPoc model unchanged — form code works without changes beyond the save hook
- `extractMeta()` exported from PocService.ts; `IPocMeta` defined in IPocJson.ts

## SP fields in POCs list (29 custom + Title = 30 total)
CustomerName, SEPrimary, SESecondary, CSEPrimary, CSESecondary, SEMPrimary,
MistSpecialist, DCSpecialist, TMEPrimary, TMEAdditional, PLMPrimary, PLMAdditional, TMAM,
PartnerName, NewWhitespace, OpportunityAmount, POCStartDate, POCEndDate,
HPENBusinessUnit, BURegion, Solutions, Status, OwnerEmail,
CompletionPct, TestsTotal, TestsPass, TestsFail, TestsInProgress, JSONFileName

## JSON meta fields (IPocMeta — 29 fields)
customerContact, customerPhone, customerEmail, pocDate, sfdcOpportunity, opportunityName,
presentedToCustomer, presentedToPartner, partnerContact, partnerPhone, partnerEmail,
additionalResources, loanSource, loanSourceOther, evalLoanNumber,
shipToType, shipToName, attentionName, attentionPhone, shipStreet, shipRoomFloor,
shipCity, shipState, shipZIP, singleSalesObjective, meddpiccElements,
preInstallDate, testPlanRevision, notes

## Current state (2026-06-29)
- Current version: **1.0.95.0** on branch **feature/sfdc-sync** — deployed and live
- Team rollout underway — Dennis sent release notes to team; cache refresh reminder issued (build number in upper-right)
- SharePoint cache delay: typically 15–30 min before refreshed version loads for users
- Dennis's Claude assistant is named **Hal**

## Release history — 2026-06-29
### v1.0.92.0
- **Stale POC Alerts config** (Admin → Scheduled Reports): Enable/Disable toggle + threshold (days); AppConfig keys StaleAlertEnabled, StaleAlertDays; applies to Not Started / In Progress / Waiting on... only — requested by Jason Taylor
- **Pending Test inline editing** (Admin → Test Library → Pending Reviews): pencil icon → edit criteria/success/methodology/importance inline → Save Changes (stays Pending) → then Approve or Reject — requested by Carlos Rodriguez

### v1.0.93.0
- **Test Plan Templates**: SEs can save entire solution test plans as templates (Save as Template button on Solution Test tab); loads clean (no statuses/outcomes/dates); locked to one solution
- Super users submit directly; others submit for review → admin approves/rejects
- Load Template panel shows approved templates for that solution; replaces current test list
- Admin → Test Library → Test Plan Templates tab: browse all, pending review card view with inline edit/approve/reject

### v1.0.94.0
- Clean and test run only (no user-facing changes)

### v1.0.95.0 (combined release)
- Edit button now labeled (visible in Pending Reviews)
- Template edit fields no longer lose focus on each keystroke
- Template name is clickable to enter edit mode
- Admin Pending Tests tab removed
- Admin Import Legacy POCs tab removed
- Admin Restore from POC-Data tab removed

## Pending / future work
- **Stale POC Alerts PA flow** — PA flow not yet built; reads StaleAlertEnabled + StaleAlertDays from AppConfig; emails region manager (To) + SE (CC) — Dennis targeting tonight (2026-06-29)
- **POC Name blank in email** — SP Title field empty for test POCs; data issue, not code; will resolve as SEs enter real data
- **PDL setup** — Dennis creating HPE PDL for admin notifications; add to Feedback Settings in Admin; share both PA flows with PDL
- End-to-end test with real POC data before team rollout
- Merge `feature/sfdc-sync` → `main` and production deploy
- Customer Report review
- Users guide (deferred)
- Mail.Send / email report (IT rejected permissions — future PA flow alternative)
- SFDC write-back (future — PA middleman)
- ServiceNow integration (future — same pattern)
- Test Plan Templates PA flow (approval notifications — future)

## Feedback notification PA flow — COMPLETED 2026-06-19
- Flow lives in PA (flow.microsoft.com) under Dennis's account — NOT part of SPFx package
- Trigger: SP Feedback list "When an item is created"
- Gets recipient emails from AppConfig list item Title='FeedbackNotifyEmails'
- Compose → Apply to each → Compose 1 (trim) → Send an email V2
- Admin emails managed in POC Manager → Admin → Feedback Settings (saves to AppConfig)
- PDL planned: Dennis setting up an HPE PDL to use instead of individual emails in AppConfig
- Share the flow in PA with the PDL so any admin can manage it (not just Dennis/Jeff)
- Gotchas: new PA designer To field is a people picker — must use classic designer or Code view workaround; Feedback list had duplicate columns from repeated provisioning — fixed by deleting list and reprovisioning

## Manager auto-filtered reports — COMPLETED 2026-06-19 (v1.0.52.0)
- BURegionMap extended: IBUConfig (sedEmail, vpGmEmail, regions: Record<string, IRegionConfig{managerEmail}>)
- Old string[] format auto-migrates in getBURegions() via migrateBURegions()
- Admin → BU/Regions tab: SED email + VP/GM email per BU; Manager email per region (inline TextField)
- Admin → Scheduled Reports tab: on/off Toggle → writes AppConfig key 'ScheduledReportsEnabled'
- Reports.tsx Mgmt View: auto-filters on login if user email matches a region manager or BU SED/VP
- Auto-filter banner: "Auto-filtered for your Region: X" with "Show all" link
- All leaders entered by Dennis on 2026-06-19
- Save BU/Regions also writes flattened AppConfig key 'ReportRecipients' (JSON array: [{bu, region, recipients}]) for PA flow consumption

## Weekly region reports PA flow — COMPLETED 2026-06-20
- Flow name: "HPEN POC Manager — Weekly Region Reports" — shared with Jeff (shows under "Shared with me" in PA)
- Trigger: daily Recurrence → Condition 1: ScheduledReportsEnabled='true' → Condition 2: day-of-week matches ReportDayOfWeek config
- Apply to each region (Parse_Recipients body) → Set variable htmlRows='' → Get POCs → Apply to each POC → Append HTML row → Condition 3 (htmlRows not empty) → Send email
- Email: HPE navy header, table with POC Name/Customer/SE/Status/End Date/Completion; overdue rows red
- Recipients stored comma-separated in ReportRecipients JSON; To field needs replace(',',';') for Outlook V2 connector
- AppConfig keys used: ScheduledReportsEnabled, ReportDayOfWeek, ReportRecipients, ReportSuppressedEmails
- ReportDayOfWeek now configurable via Admin → Scheduled Reports tab (v1.0.62.0)
- **Suppression filtering** — COMPLETED 2026-06-21
  - Filter array action inside If yes branch: From = `split(replace(coalesce(recipients,''),' ',''),',')`
  - Advanced mode: `@not(contains(coalesce(outputs('SuppressedList'),createArray('')),toLower(trim(item()))))`
  - To Address Compose: `join(body('Filter_Recipients'),';')`
  - SuppressedList is a Compose that runs split() on AppConfig ReportSuppressedEmails value
- **PA expression gotchas learned:**
  - Status is a Choice field → must use `?['Status']?['Value']` not `?['Status']`
  - POCEndDate null check: `and(not(empty(?['POCEndDate'])), less(ticks(?['POCEndDate']), ticks(utcNow())))`
  - CompletionPct is Float → use `coalesce(..., 0)` not `empty()` for null check
  - Email To field: replace commas with semicolons `replace(recipients,',',';')`
  - **Compose action output**: use `outputs('ActionName')` NOT `body('ActionName')` — `body()` returns null for Compose actions
  - **Filter array output**: use `body('ActionName')` — correct for Filter array (Data Operations)
  - Initialize variable must be at TOP LEVEL only (not inside conditions/loops)
  - Classic PA designer required for email To field (new designer uses people picker)

## Exit guard & custom test prompt — COMPLETED 2026-06-20 (v1.0.59.0)
- "All POCs" back arrow now checks isDirty — if dirty shows "Unsaved Changes" dialog: Save & Exit / Exit Without Saving / Cancel
- "Save & Exit" routes through handleExitFlow() which checks for isCustom && !submittedToLibrary tests
- If unsubmitted custom tests exist, shows prompt listing them (all pre-checked) before exit: Submit Selected & Exit / Skip & Exit / Cancel
- After sharing via the blue Share icon in-row, test is marked submittedToLibrary=true so it won't re-prompt
- IPocTestEntry now has submittedToLibrary?: boolean field
- Version badge (top-right header) now auto-syncs to package version on every deploy — useful for support: ask user what version they see to confirm cache is clear


## SFDC Opportunity Lookup — PA Flow (feature/sfdc-sync)

### How it works
- SPFx writes a row to `SfdcLookupRequests` SP list (`https://hpe.sharepoint.com/teams/hpen-poc-manager/Lists/SfdcLookupRequests`) with `LookupStatus: Pending` and the opportunity ID as Title
- App polls that row every 3 seconds for up to 4.5 minutes (90 iterations) waiting for `LookupStatus` to flip to `Complete`
- PA flow watches the list, calls Salesforce, writes back AccountName / OppAmount / OwnerName / OpportunityName, sets `LookupStatus = Complete`
- Fields written back into POC form: customerName, opportunityAmount, tmam, opportunityName

### Bug diagnosed 2026-06-29 — BLOCKED on Salesforce admin
- **Root cause:** PA flow Salesforce connector runs under Dennis's credentials — only sees opportunities Dennis has access to in Salesforce
- **Symptom:** Requests from Josh (bugbee), Corbin (murrow), Jenise (anderson) all stay `Pending` — Salesforce returns nothing for those opportunity IDs
- **Confirmed via SfdcLookupRequests list:** Multiple Pending rows for same opportunity IDs across different users; only Dennis's own opportunities and Jeff's complete successfully
- **Fix needed:** Salesforce admin must provide either:
  1. A service/integration account with org-wide read access to Opportunities, or
  2. Grant Dennis's account (dennis.dodd@hpe.com) org-wide read visibility to Opportunity records
- **Status:** Dennis drafting email to Salesforce admin (2026-06-29) — admin identity TBD (ask TM/AM or Jeff)
- **Once fixed:** Re-authenticate the Salesforce connector in the PA flow under the broader account (~5 min fix)
- Flow connector update: Edit flow → find Salesforce action → click connection → Add new connection → sign in with service account

**Why:** HPE Networking SEs use this to track POC test plans tied to SFDC opportunities. Dennis Dodd is primary user/designer, Jeff Dziedzic is co-developer.
**How to apply:** When Dennis says "workbench" or "POC Manager", he means the SPFx version at `C:\dev\poc-manager-spfx`.
