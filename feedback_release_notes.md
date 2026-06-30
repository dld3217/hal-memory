---
name: feedback-release-notes
description: Format and closing line for POC Manager release notes emails sent to the test team
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cf56e46-c466-4627-845a-34fd5c5353ba
---

Release notes emails go to the test team after every deploy. Keep them short and professional.

**Standard format:**
- Subject: `POC Manager Update — vX.X.X.X`
- Opening: tell users to clear cache and confirm the version number in the upper-right corner
- Body: bullet or short paragraph per change, plain English (no code jargon)
- Closing: direct users to submit issues or feature requests through the Feedback button in the app (exact wording can vary naturally)
- Sign off: Dennis Dodd | HPE Networking, TOLA SEM

**Why:** Dennis sends these after every build to keep the test team informed. The Feedback button in the app is the preferred channel for issue reporting — don't ask them to reply to the email.
