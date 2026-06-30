---
name: feedback-session-end
description: Always update memory files at the end of every work session
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 35b4820e-c1a9-44b0-84f5-4d75d61da277
---

At the end of every work session, update memory files AND push to GitHub.

**Why:** Dennis wants memory backed up to GitHub so nothing is lost if the local machine is wiped. Memory files live at `C:\Users\doddd\.claude\projects\C--Users-doddd\memory\` and are backed up to `github.com/dld3217/hal-memory` (private repo).

**How to apply:** When Dennis says "wrap up and push" (or signals session end), do both steps:
1. Review the conversation and write any new facts, preferences, or project changes to the appropriate memory files
2. Run `git add . && git commit -m "Session wrap-up — [date]" && git push` in the memory folder

Also save mid-session when key milestones happen (feature completed, PA flow built, version deployed) — don't wait until end of session.
