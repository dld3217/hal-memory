---
name: feedback-deploy-cadence
description: "Daily deploy rhythm for POC Manager — commit throughout the day, single evening deploy + team email"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cf56e46-c466-4627-845a-34fd5c5353ba
---

POC Manager follows a daily release cadence now that there are active users:

**Throughout the day:** commit code changes to GitHub as each feature is finished (no version bump, no .sppkg upload). This preserves work without disrupting users mid-day.

**In the evening:** 
1. Run `.\deploy.ps1 "summary of day's changes"` — bumps version, bundles, packages, commits, pushes
2. Upload fresh `sharepoint/solution/poc-manager-spfx.sppkg` to App Catalog
3. Send daily release notes email to the team listing all changes for the day

**Why:** Active user base means mid-day deploys are disruptive. Batching to evening gives users a stable build all day and one clean email to read each morning.

**How to apply:** Don't prompt Dennis to deploy after every individual change. Accumulate changes, keep a running list of what's been built that day, and remind him to deploy in the evening.
