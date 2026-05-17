# ConnectWise Ticket Triage

_Pull the user's open ConnectWise tickets, summarize by board, status, and priority, and propose triage actions. Use when Travis or another RX Tech user says "what's on my plate", "triage my tickets", "open tickets", "my queue", "what's assigned to me", or asks about specific board status. Also triggers on direct ticket-number references like "what's the status of 12345" or "look up ticket 98765"._

Version: 0.1.1

## Install

**Claude Code CLI users:**
```
/plugin marketplace add NBTX77/rx-tech-skills
/plugin install cw-ticket-triage@rx-tech-skills
```

**Cowork users (personal plan):** download the `.plugin` file from your RxTechApp admin (https://rxtech.app/admin/skills) and drag-drop into Cowork chat.

## Source

This plugin is auto-generated from the canonical skill row in RxTechApp's `mcp_skills` table. Authoring happens at https://rxtech.app/admin/skills.
