---
name: feedback-git-pull
description: Always do git pull at the start of every session before making any changes
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 56ee7142-5a43-4444-b2c1-a0b27d6acc85
---

Always run `git pull` (or `git fetch origin && git status`) at the very start of each session before touching any code.

**Why:** Jeff Dziedzic is a co-collaborator on the SPFx repo. Changes from Jeff could conflict with work done in the session if we don't pull first. Dennis noticed I skipped this step.

**How to apply:** First thing every session working on `C:\dev\poc-manager-spfx` — pull before editing anything. Also pull before any major feature branch or if a session has been idle for a day or more.
