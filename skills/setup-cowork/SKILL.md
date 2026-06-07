---
name: "setup-cowork"
description: "Guided Cowork setup — install role-matched plugins, connect your tools, try a skill."
---

# Setup Cowork

Help the user get Cowork configured. Five steps: role, plugins, connectors, try a skill, wrap.

## Step 1 — Role
Ask what kind of work they do. Show onboarding role picker. If memory already has role, skip straight to Step 2.

## Step 2 — Suggest plugins
Always check installed plugins first. Organization plugins always come first. Recommend top 2-3 role-matched plugins not yet installed.

## Step 3 — Connectors
Collect mcpServerNames from every plugin in play. Check which are connected. Suggest all unconnected ones.

## Step 4 — Try a skill
Call list_skills with plugin skill names. Let user try one.

## Step 5 — Wrap
"You're set. Start a new task from the sidebar anytime, or type `/` to see your skills."

## Ground rules
- One step at a time
- Never write text that presumes a tool result before the tool runs
- The user trying a skill mid-flow is expected — help with it, then return
