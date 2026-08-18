# rx-tech-skills

Rx Tech internal skills library — Cowork-plugin marketplace.

Auto-generated from https://rxtech.app/admin/skills. Manual edits to this repo are overwritten on the next publish.

## Available plugins

- **project-kickoff-readiness** v0.1.1 — Run a ConnectWise project kickoff readiness review — assemble what a PM needs before kickoff and, more importantly, name what is missing. Use when asked for a "kickoff review", "kickoff readiness", "project readiness", "readiness report", "review the new projects", "what's on the project board", "are these projects ready", "10 New cohort", or when handed a project number or RXS number and asked whether it is ready to start. Also use before a Monday project meeting, or when someone asks what to chase on a project. Produces a flags-first report, optionally as a PDF.
- **cw-ticket-triage** v0.1.5 — Pull the user's open ConnectWise tickets, summarize by board, status, and priority, and propose triage actions. Use when Travis or another Rx Tech user says "what's on my plate", "triage my tickets", "open tickets", "my queue", "what's assigned to me", or asks about specific board status. Also triggers on direct ticket-number references like "what's the status of 12345" or "look up ticket 98765".

## Adding the marketplace (Claude Code CLI)

```
/plugin marketplace add NBTX77/rx-tech-skills
```

Browse + install individual plugins via the `/plugin` UI after adding the marketplace.
