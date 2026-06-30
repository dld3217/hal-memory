---
name: project-timbits
description: "Tim·bit History — HPE Networking weekly sales enablement archive (standalone HTML file)"
metadata:
  node_type: memory
  type: project
  originSessionId: e6fb5091-90ae-4e5b-bf2c-4418386f47cb
---

# Tim·bit History — HPE Networking

A standalone single-file HTML tool that serves as a searchable archive of weekly "Tim·bit" sales enablement emails for the HPE Networking sales team.

## File location
`C:\Users\doddd\Downloads\timbit-history (9).html`
(rename/move as needed — Dennis updates this manually each week)

## Source content
Weekly Tim·bit Word doc lives at HPE SharePoint (requires HPE login):
https://hpe-my.sharepoint.com/:w:/p/tim_vanevenhoven/EY1uH3hV1hBBvGCmi-4DSmMBDNHLLv3AEN3a9k_FStNqjQ?e=9dBYpf

## How to add a new entry (weekly)

Open the HTML file, find the `<script>` tag and the `const entries = [` line.
Insert a new block **before** the first existing entry (so newest appears at top):

```js
{
  date: "YYYY/MM/DD",
  title: "Tim·bit title here",
  body: "Summary paragraph here.",
  link: "https://...",
  cats: ["cat1","cat2"],
  format: "format-value",
  keywords: "keyword1 keyword2 keyword3"
},
```

## Field reference

### `cats` — pick one or more:
`iot` `security` `retail` `healthcare` `hospitality` `education` `manufacturing` `sase` `wifi` `location` `campus` `datacenter` `partner` `ai` `5g`

### `format` — pick one:
`video` `solution` `blog` `case-study` `infographic` `ebook` `webinar` `audio`

### `keywords`
Space-separated search terms — include partner names, product names, acronyms, and topic words that users might search for.

## Entry builder tool
`C:\Users\doddd\Downloads\timbit-new-entry.html` — standalone HTML form that generates a ready-to-paste JS entry block. Open in browser, fill in fields, click Generate, click Copy, paste at top of entries array in timbit-history.html.

## SPFx version — IN PROGRESS (started 2026-06-26)
- Location: `C:\dev\timbit-spfx\`
- Stack: SPFx 1.20 + React + TypeScript + Fluent UI v8 + PnPjs v4 (same as POC Manager)
- Target site: https://hpe.sharepoint.com/teams/EnterpriseSalesTeamHome
- Solution ID: e5e87860-e2cb-43da-8f1c-6f6bdfa00b2e
- GitHub: https://github.com/dld3217/timbit-spfx (private, under Dennis's account)
- Current version: 1.0.0.0 — initial commit pushed to GitHub
- Build: clean (`gulp bundle --ship` passes)

### Components built
- `TimBitWebPart.ts` — entry point, reads adminEmails + distributionList from property pane
- `TimBitApp.tsx` — root, shows admin nav bar for authorized users only
- `TimbitList.tsx` — public archive (search, year, format, category filters)
- `TimbitAdmin.tsx` — add/edit/delete/publish entries (admin only)
- `TimbitEmailPreview.tsx` — generates HTML email from latest week's entries, Copy button
- `TimbitService.ts` — PnPjs CRUD against `TimBits` SP list
- `ITimbit.ts` — data model

### Admin access
- Stored in web part property pane as comma-separated emails (`adminEmails`)
- Tim Vanevenhoven (tim.vanevenhoven@hpe.com) is primary admin
- Distribution list email stored in property pane (`distributionList`) — pre-populates email To field
- Tim's PDL address unknown — he BCCs it; need to ask Tim

### SP List needed (not yet created)
List name: `TimBits` on EnterpriseSalesTeamHome
Columns: Title, TBDate, WeekOf, Body, Link, Format, Categories, Keywords, Published

### App Catalog — COMPLETED (IT provisioned 2026-06-30)
- Creating a Site Collection App Catalog on EnterpriseSalesTeamHome requires SharePoint tenant admin rights
- Dan (americasbaumdani@hpe.com) is site owner/admin for EnterpriseSalesTeamHome but does NOT have tenant admin rights
- Jeff Dziedzic also does not have those rights — the original POC Manager App Catalog was set up by an IT admin
- Dennis submitted a helpdesk ticket on 2026-06-29 requesting App Catalog creation on EnterpriseSalesTeamHome
- Same request that was made for POC Manager — reference that ticket in the new one
- PnP PowerShell auth against HPE tenant blocked by policy (ClientId not recognized, -UseWebLogin removed in v3)

### Next steps (once IT creates App Catalog)
1. Create `TimBits` SP list on EnterpriseSalesTeamHome with correct columns
2. First deploy: `.\deploy.ps1 "first deploy"` then upload `timbit-spfx.sppkg` to site App Catalog
3. Add web part to a modern page on EnterpriseSalesTeamHome
4. Test with Tim — get his PDL address to plug into `distributionList` property pane setting

### Tim Vanevenhoven context
- Tim generates 2-3 Tim·bits per week for WW HPE Networking sales org
- Source Word doc: https://hpe-my.sharepoint.com/:w:/p/tim_vanevenhoven/... (requires HPE login)
- He sends via email with himself in To: field and distribution list in BCC
- Email generation: Tim publishes entries, clicks "Generate Email", copies HTML into Outlook

## Notes (standalone HTML file)
- No build system — file opens directly in browser
- All filtering is client-side JavaScript (no backend)
- Entries are sorted newest-first by position in the array (not auto-sorted by date)
- When Dennis says "Tim-Bits" he means this project
